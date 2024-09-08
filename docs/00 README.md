# mother-of-all-self-hosting / mash-playbook

## Table of Contents
- [mother-of-all-self-hosting / mash-playbook](#mother-of-all-self-hosting--mash-playbook)
  - [Table of Contents](#table-of-contents)
  - [1.0 - Prerequisites](#10---prerequisites)
  - [2.0 - Ansible](#20---ansible)
    - [2.1 - Running this playbook](#21---running-this-playbook)
    - [2.2 - Supported Ansible versions](#22---supported-ansible-versions)
    - [2.3 - Upgrading Ansible](#23---upgrading-ansible)
    - [2.4 - Using Ansible via](#24---using-ansible-via)
    - [2.4.1 - Running Ansible in a container on the server itself](#241---running-ansible-in-a-container-on-the-server-itself)
    - [2.4.2 - Running Ansible in a container on another computer (not the server)](#242---running-ansible-in-a-container-on-another-computer-not-the-server)
    - [2.4.3 - If you don't use SSH keys for authentication](#243---if-you-dont-use-ssh-keys-for-authentication)
    - [2.4.4 - Resolve directory ownership issues](#244---resolve-directory-ownership-issues)
  - [3.0 - Alternative architectures](#30---alternative-architectures)
    - [3.1 - Implementation details](#31---implementation-details)
  - [4.0 - Self-building](#40---self-building)
  - [5.0 - Configuring DNS](#50---configuring-dns)
  - [6.0 - Getting the playbook](#60---getting-the-playbook)
    - [6.1 - Using git to get the playbook](#61---using-git-to-get-the-playbook)
    - [6.2 - Downloading the playbook as a ZIP archive](#62---downloading-the-playbook-as-a-zip-archive)
  - [7.0 - Configuring the Ansible playbook](#70---configuring-the-ansible-playbook)
  - [8.0 - Configuring interoperability with other services](#80---configuring-interoperability-with-other-services)
    - [8.1 - Disabling Traefik installation](#81---disabling-traefik-installation)
    - [8.2 - Disabling  installation](#82---disabling--installation)
    - [8.3 - Disabling timesyncing (systemd-timesyncd / ntp) installation](#83---disabling-timesyncing-systemd-timesyncd--ntp-installation)
  - [9.0 - Installing](#90---installing)
    - [9.1 - Playbook tags introduction](#91---playbook-tags-introduction)
    - [9.2 - Installing services](#92---installing-services)
    - [9.2.1 - Installing a brand new server (without importing data)](#921---installing-a-brand-new-server-without-importing-data)
    - [9.2.2 - Installing a server into which you'll import old data](#922---installing-a-server-into-which-youll-import-old-data)
    - [9.3 - Maintaining your setup in the future](#93---maintaining-your-setup-in-the-future)
    - [9.4 - Things to do next](#94---things-to-do-next)
  - [10 - Importing an existing Postgres database](#10---importing-an-existing-postgres-database)
    - [10.1 - Prerequisites](#101---prerequisites)
    - [10.2 - Importing a dump file](#102---importing-a-dump-file)
- [11.0 - Upgrading services](#110---upgrading-services)
- [12.0 - Maintenance / Postgres](#120---maintenance--postgres)
  - [12.1 - Getting a database terminal](#121---getting-a-database-terminal)
  - [12.2 - Vacuuming PostgreSQL](#122---vacuuming-postgresql)
  - [12.3 - Backing up PostgreSQL](#123---backing-up-postgresql)
  - [12.4 - Upgrading PostgreSQL](#124---upgrading-postgresql)
  - [12.5 - Tuning PostgreSQL](#125---tuning-postgresql)
  - [12.6 - Recommended other services](#126---recommended-other-services)
- [13.0 - Maintenance and Troubleshooting](#130---maintenance-and-troubleshooting)
  - [13.1 - How to see the current status of your services](#131---how-to-see-the-current-status-of-your-services)
  - [13.2 - Increasing logging](#132---increasing-logging)
  - [13.3 - Remove unused Docker data](#133---remove-unused-docker-data)
  - [13.4 - Postgres](#134---postgres)
- [14.0 - Developer Documentation](#140---developer-documentation)
  - [14.1 - Support a new service | Create your own role](#141---support-a-new-service--create-your-own-role)
    - [14.1.1 Check if](#1411-check-if)
    - [14.1.2 Create the Ansible role in a public git repository](#1412-create-the-ansible-role-in-a-public-git-repository)
    - [14.1.3 Update the MASH-Playbook to support your created Ansible role](#1413-update-the-mash-playbook-to-support-your-created-ansible-role)
    - [14.2 - Additional hints](#142---additional-hints)
- [15.0 - Uninstalling](#150---uninstalling)

## 1.0 - Prerequisites

To install services using this Ansible playbook, you need:

- (Recom>Peaed) An **x86-64** (`amd64`) or **arm64** server running one of these operating systems:
  - **Red Hat Enterprise Linux** or derivative distros, e.g. Rocky Linux (Major version 7 or newer)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](02-ansible.md#supported-ansible-versions)
  - **Archlinux**

Generally, newer is better. We only strive to support released stable versions of distributions, not betas or pre-releases. This playbook can take over your whole server or co-exist with other services that you have there.

This playbook somewhat supports running on non-`amd64` architectures like ARM. See [Alternative Architectures](03-alternative-architectures.md).

If your distro runs within an [LXC container](https://linuxcontainers.org/), you may hit [this issue](https://github.com/spantaleev/matrix--ansible-deploy/issues/703). It can be worked around, if absolutely necessary, but we suggest that you avoid running from within an LXC container.

- `root` access to your server (or a user capable of elevating to `root` via `sudo`).

- [Python](https://www.python.org/) being installed on the server. Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`). On some distros, Ansible may incorrectly [detect the Python version](https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html) (2 vs 3) and you may need to explicitly specify the interpreter path in `inventory/hosts` during installation (e.g. `ansible_python_interpreter=/usr/bin/python3`)

- [sudo](https://www.sudo.ws/) being installed on the server, even when you've configured Ansible to log in as `root`. Some distributions, like a minimal Debian net install, do not include the `sudo` package by default.

- The [Ansible](http://ansible.com/) program being installed on your own computer. It's used to run this playbook and configures your server for you. Take a look at [our guide about Ansible](02-ansible.md) for more information, as well as [version requirements](02-ansible.md#supported-ansible-versions) and alternative ways to run Ansible.

- the [passlib](https://passlib.readthedocs.io/en/stable/index.html) Python library installed on the computer you run Ansible. On most distros, you need to install some `python-passlib` or `py3-passlib` package, etc.

- [git](https://git-scm.com/) is the recommended way to download the playbook to your computer. `git` may also be required on the server if you will be [self-building](04-self-building.md) components.

- [just](https://github.com/casey/just) for running `just update` and playbook installation commands, etc. (see [`justfile`](../justfile)). You can get by without `just` (by running `ansible-galaxy`, `ansible-playbook` commands manually), but maintaining your playbook setup will require more manual work. `just` (thanks to the commands defined in the `justfile`) keeps various files (`setup.yml`, `requirements.yml`, `group_vars/mash_servers`) up-to-date with the templates in [the `templates/` directory](../templates/) automatically.

- at least one domain name you can use

- Some TCP/UDP ports open. This playbook (actually [ itself](https://docs..com/network/iptables/)) configures the server's internal firewall for you. In most cases, you don't need to do anything special. But **if your server is running behind another firewall**, you'd need to open these ports:

  - `80/tcp`: HTTP webserver
  - `443/tcp`: HTTPS webserver
  - potentially some other ports, depending on the services that you enable in the **configuring the playbook** step (later on). Consult each service's documentation page in `docs/` for that.

When ready to proceed, continue with [Configuring DNS](05-configuring-dns.md).

## 2.0 - Ansible

### 2.1 - Running this playbook

This playbook is meant to be run using [Ansible](https://www.ansible.com/).

Ansible typically runs on your local computer and carries out tasks on a remote server.
If your local computer cannot run Ansible, you can also run Ansible on some server somewhere (including the server you wish to install to).

### 2.2 - Supported Ansible versions

To manually check which version of Ansible you're on, run: `ansible --version`.

For the **best experience**, we recommend getting the **latest version of Ansible available**.

We're not sure what's the minimum version of Ansible that can run this playbook successfully.
The lowest version that we've confirmed (on 2022-11-26) to be working fine is: `ansible-core` (`2.11.7`) combined with `ansible` (`4.10.0`).

If your distro ships with an Ansible version older than this, you may run into issues. Consider [Upgrading Ansible](02-ansible.md#upgrading-ansible) or [using Ansible via](02-ansible.md#using-ansible-via-).

### 2.3 - Upgrading Ansible

Depending on your distribution, you may be able to upgrade Ansible in a few different ways:

- by using an additional repository (PPA, etc.), which provides newer Ansible versions. See instructions for [CentOS](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-rhel-centos-or-fedora), [Debian](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-debian), or [Ubuntu](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) on the Ansible website.

- by removing the Ansible package (`yum remove ansible` or `apt-get remove ansible`) and installing via [pip](https://pip.pypa.io/en/stable/installation/) (`pip install ansible`).

If using the `pip` method, do note that the `ansible-playbook` binary may not be on the `$PATH` (<https://linuxconfig.org/linux-path-environment-variable>), but in some more special location like `/usr/local/bin/ansible-playbook`. You may need to invoke it using the full path.

**Note**: Both of the above methods are a bad way to run system software such as Ansible.
If you find yourself needing to resort to such hacks, please consider reporting a bug to your distribution and/or switching to a sane distribution, which provides up-to-date software.

### 2.4 - Using Ansible via

Alternatively, you can run Ansible inside a  container (powered by the [devture/ansible](https://hub..com/r/devture/ansible/)  image).

This ensures that you're using a very recent Ansible version, which is less likely to be incompatible with the playbook.

You can either [run Ansible in a container on the server itself](02-ansible.md#running-ansible-in-a-container-on-the-server-itself) or [run Ansible in a container on another computer (not the server)](02-ansible.md#running-ansible-in-a-container-on-another-computer-not-the-server).

### 2.4.1 - Running Ansible in a container on the server itself

To run Ansible in a () container on the server itself, you need to have a working  installation.
 is normally installed by the playbook, so this may be a bit of a chicken and egg problem. To solve it:

- you **either** need to install Ansible manually first. Follow [the upstream instructions](https://docs..com/engine/install/) for your distribution and consider setting `mash_playbook__installation_enabled: false` in your `vars.yml` file, to prevent the playbook from installing
- **or** you need to run the playbook in another way (e.g. [Running Ansible in a container on another computer (not the server)](02-ansible.md#running-ansible-in-a-container-on-another-computer-not-the-server)) at least the first time around

Once you have a working  installation on the server, **clone the playbook** somewhere on the server and configure it as per usual (`inventory/hosts`, `inventory/host_vars/..`, etc.), as described in [configuring the playbook](05-
configuring-playbook.md).

You would then need to add `ansible_connection=community..nsenter` to the host line in `inventory/hosts`. This tells Ansible to connect to the "remote" machine by switching Linux namespaces with [nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html), instead of using SSH.
Alternatively, you can leave your `inventory/hosts` as is and specify the connection type in **each** `ansible-playbook` call you do later, like this: `just install-all --connection=community..nsenter` (or `ansible-playbook --connection=community..nsenter ...`).

Run this from the playbook's directory:

```bash
 run -it --rm \
--privileged \
--pid=host \
-w /work \
-v `pwd`:/work \
--entrypoint=/bin/sh \
.io/devture/ansible:2.17.0-r0-1
```

Once you execute the above command, you'll be dropped into a `/work` directory inside a  container.
The `/work` directory contains the playbook's code.

First, consider running `git config --global --add safe.directory /work` to [resolve directory ownership issues](02-ansible.md#resolve-directory-ownership-issues).

Finally, you can execute `just` or `ansible-playbook ...` (e.g. `ansible-playbook --connection=community..nsenter ...`) commands as per normal now.

### 2.4.2 - Running Ansible in a container on another computer (not the server)

Run this from the playbook's directory:

```bash
 run -it --rm \
-w /work \
-v `pwd`:/work \
-v $HOME/.ssh/id_rsa:/root/.ssh/id_rsa:ro \
--entrypoint=/bin/sh \
.io/devture/ansible:2.17.0-r0-1
```

The above command tries to mount an SSH key (`$HOME/.ssh/id_rsa`) into the container (at `/root/.ssh/id_rsa`).
If your SSH key is at a different path (not in `$HOME/.ssh/id_rsa`), adjust that part.

Once you execute the above command, you'll be dropped into a `/work` directory inside a  container.
The `/work` directory contains the playbook's code.

First, consider running `git config --global --add safe.directory /work` to [resolve directory ownership issues](#resolve-directory-ownership-issues).

Finally, you execute `just` or `ansible-playbook ...` commands as per normal now.

### 2.4.3 - If you don't use SSH keys for authentication

If you don't use SSH keys for authentication, simply remove that whole line (`-v $HOME/.ssh/id_rsa:/root/.ssh/id_rsa:ro`).
To authenticate at your server using a password, you need to add a package. So, when you are in the shell of the ansible  container (the previously used `run -it ...` command), run:

```bash
apk add sshpass
```

Then, to be asked for the password whenever running an  `ansible-playbook` command add `--ask-pass` to the arguments of the command.

### 2.4.4 - Resolve directory ownership issues

Because you're `root` in the container running Ansible and this likely differs fom the owner (your regular user account) of the playbook directory outside of the container, certain playbook features which use `git` locally may report warnings such as:

> fatal: unsafe repository ('/work' is owned by someone else)
> To add an exception for this directory, call:
> git config --global --add safe.directory /work

These errors can be resolved by making `git` trust the playbook directory by running `git config --global --add safe.directory /work`

## 3.0 - Alternative architectures

As stated in the [Prerequisites](01-prerequisites.md), currently only `amd64` (`x86_64`) is fully supported.

The playbook automatically determines the target server's architecture (the `mash_playbook_architecture` variable) to be one of the following:

- `amd64` (`x86_64`)
- `arm32`
- `arm64`

Some tools and container images can be built on the host or other measures can be used to install on that architecture.

### 3.1 - Implementation details

For `amd64`, prebuilt container images are used for all components.

For other architecture (`arm64`, `arm32`), components which have a prebuilt image make use of it. If the component is not available for the specific architecture, [self-building](04-self-building.md) will be used. Not all components support self-building though, so your mileage may vary.

## 4.0 - Self-building

**Caution: self-building does not have to be used on its own. See the [Alternative Architectures](03-alternative-architectures.md) page.**

The playbook supports self-building of various components, which don't have a container image for your architecture. For `amd64`, self-building is not required.

For other architectures (e.g. `arm32`, `arm64`), ready-made container images are used when available. If there's no ready-made image for a specific component and said component supports self-building, an image will be built on the host. Building images like this takes more time and resources (some build tools need to get installed by the playbook to assist building).

To make use of self-building, you don't need to do anything. If a component has an image for the specified architecture, the playbook will use it directly. If not, it will build the image on the server itself.

Note that **not all components support self-building yet**.

Adding self-building support to other roles is welcome. Feel free to contribute!

If you'd like **to force self-building** even if an image is available for your architecture, look into the `*_self_build` variables provided by individual roles.

## 5.0 - Configuring DNS

To reach your services, you'd need to do some DNS configuration.

**We recommend** that you:

- create at least one generic domain (e.g. `mash.DOMAIN`) for easily hosting various services at different subpaths (e.g. `mash.DOMAIN/miniflux`, `mash.DOMAIN/radicale`, etc.)

- create additional domains (`CNAME` DNS records pointing to the main generic domain) for large services or services that explicitly require their own dedicated domain

Some services (like [Uptime-kuma](services/uptime-kuma.md)) require being hosted at their own dedicated domain.
Others, you can put on their own domain/subdomain or at subpaths on any domain you'd like.

As an **example** setup, consider this one:

| Service                    | Type  | Host                | Target              |
|--------------------------- |-------|---------------------|---------------------|
| Miniflux, Radicale, others | A     | `mash.example.com`  | `IP of your server` |
| Nextcloud                  | CNAME | `cloud.example.com` | `mash.example.com`  |

With such a setup, you could reach:

- the feedreader [Miniflux](services/miniflux.md) at `https://mash.example.com/miniflux` (if you set
`miniflux_hostname: mash.example.com` and `miniflux_path_prefix: /miniflux` in your `vars.yml`)

- the [Radicale](services/radicale.md) CalDAV/CardDAV sever at `https://mash.example.com/radicale` (if you set
`radicale_hostname: mash.example.com` and `radicale_path_prefix: /radicale` in your `vars.yml`)

- Nextcloud at its own dedicated domain, at `https://cloud.example.com`

Hosting services at subpaths is more convenient, because it doesn't require you to create additional DNS records and no new SSL certificates need to be retrieved.

Still, if you'd like each service to have its own dedicated domain (or subdomain), feel free to configure services that way by making sure that you set `<service>_hostname` and `<service>_path_prefix`
accordingly in your `vars.yml`.

Be mindful as to how long it will take for the DNS records to propagate.

When you're done configuring DNS, proceed to [Configuring the playbook](07-configuring-playbook.md).

## 6.0 - Getting the playbook

This Ansible playbook is meant to be executed on your own computer (not on the server).

In special cases (if your computer cannot run Ansible, etc.) you may put the playbook on the server as well.

You can retrieve the playbook's source code by:

- [Using git to get the playbook](06-getting-the-playbook.md#using-git-to-get-the-playbook) (recommended)

- [Downloading the playbook as a ZIP archive](06-getting-the-playbook.md#downloading-the-playbook-as-a-zip-archive) (not recommended)

### 6.1 - Using git to get the playbook

We recommend using the [git](https://git-scm.com/) tool to get the playbook's source code, because it lets you easily keep up to date in the future when [Maintaining services](maintenance-upgrading-services.md).

Once you've installed git on your computer, you can go to any directory of your choosing and run the following command to retrieve the playbook's source code:

```bash
git clone https://github.com/mother-of-all-self-hosting/mash-playbook.git
```

This will create a new `mash-playbook` directory.
You're supposed to execute all other installation commands inside that directory.

### 6.2 - Downloading the playbook as a ZIP archive

Alternatively, you can download the playbook as a ZIP archive.
This is not recommended, as it's not easy to keep up to date with future updates. We suggest you [use git](#using-git-to-get-the-playbook) instead.

The latest version is always at the following URL: <https://github.com/mother-of-all-self-hosting/mash-playbook/archive/master.zip>

You can extract this archive anywhere. You'll get a directory called `mash-playbook-master`.
You're supposed to execute all other installation commands inside that directory.

---------------------------------------------

No matter which method you've used to download the playbook, you can proceed by [Configuring the playbook](configuring-playbook.md).

## 7.0 - Configuring the Ansible playbook

To configure the playbook, you need to have done the following things:

- have a server where services will run
- [retrieved the playbook's source code](getting-the-playbook.md) to your computer

You can then follow these steps inside the playbook directory:

1. create a directory to hold your configuration (`mkdir -p inventory/host_vars/<your-domain>`)

2. copy the sample configuration file (`cp examples/vars.yml inventory/host_vars/<your-domain>/vars.yml`)

3. edit the configuration file (`inventory/host_vars/<your-domain>/vars.yml`) to your liking. You should [enable one or more services](supported-services.md) in your `vars.yml` file. You may also take a look at the various `roles/**/ROLE_NAME_HERE/defaults/main.yml` files (after importing external roles with `just update` into `roles/galaxy`) and see if there's something you'd like to copy over and override in your `vars.yml` configuration file.

4. copy the sample inventory hosts file (`cp examples/hosts inventory/hosts`)

5. edit the inventory hosts file (`inventory/hosts`) to your liking

If you're installing services on the same server using another playbook (like [matrix--ansible-deploy](https://github.com/spantaleev/matrix--ansible-deploy)) or you already have [Traefik](./services/traefik.md) or [](./services/.md) installed on the server, consult our [Interoperability](./interoperability.md) documentation.

When you're done with all the configuration you'd like to do, continue with [Installing](installing.md).

## 8.0 - Configuring interoperability with other services

This playbook tries to get you up and running with minimal effort and provided you have followed the [example `vars.yml` file](../examples/vars.yml), will install the [Traefik](services/traefik.md) reverse-proxy server by default.

Sometimes, you're using a server which already has Traefik. In such cases these are undesirable:

- the playbook trying to run its own Traefik instance and running into a conflict with your other Traefik instance over ports (`tcp/80` and `tcp/443`)

- multiple playbooks trying to install , etc.

Below, we offer some suggestions for how to make this playbook more interoperable. Feel free to cherry-pick the parts that make sense for your setup.

### 8.1 - Disabling Traefik installation

If you're installing [Traefik](services/traefik.md) on your server in another way, you can use your already installed Traefik instance and [disable the Traefik instance installed by MASH](services/traefik.md#using-another-traefik-instance-not-installing-traefik).

If you are using the [matrix--ansible-deploy](https://github.com/spantaleev/matrix--ansible-deploy) playbook, it already runs its own Traefik instance (`matrix-traefik`). We recommend that you [disable the Traefik instance installed by MASH](services/traefik.md#using-another-traefik-instance-not-installing-traefik), because the Traefik instance installed by the Matrix playbook does the same, but also contains additional configuration for handling the Matrix federation port (`8448`).

### 8.2 - Disabling  installation

If you're installing [](https://www..com/) on your server in another way, disable this component from the playbook:

```yaml
mash_playbook__installation_enabled: false
```

### 8.3 - Disabling timesyncing (systemd-timesyncd / ntp) installation

If you're installing `systemd-timesyncd` or `ntp` on your server in another way, disable this component from the playbook:

```yaml
devture_timesync_installation_enabled: false
```

## 9.0 - Installing

If you've [configured the playbook](configuring-playbook.md) and have prepared the required domains (DNS records) depending on the services you've enabled, you can start the installation procedure.

This playbook makes use of the [`just`](https://github.com/casey/just) utility to make it easier to run playbook-related commands defined in the [`justfile`](../justfile).
We recommend installing and using using `just` - otherwise, you'll need to do more manual work.

**Before installing** and each time you update the playbook in the future, you will need to:

- (only if you're not using the [`just`](https://github.com/casey/just) utility): create `setup.yml`, `requirements.yml` and `group_vars/mash_servers` based on the up-to-date templates found in the [`templates/` directory](../templates). If you are using `just`, these files are created and maintained up-to-date automatically.

- update the Ansible roles in this playbook by either running `just update` or `git pull && just roles`. `just update` is a shortcut that calls `git pull` and `just roles` with a single command, while `just roles` is a shortcut which ultimately runs either [agru](https://github.com/etkecc/agru) or [ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html) to download Ansible roles defined in the `requirements.yml` file. If you don't have `just`, you can also manually run the `roles` commands seen in the [`justfile`](../justfile).

### 9.1 - Playbook tags introduction

The Ansible playbook's tasks are tagged, so that certain parts of the Ansible playbook can be run without running all other tasks.

The general command syntax is:

- (**recommended**) when using `just`: `just run-tags COMMA_SEPARATED_TAGS_GO_HERE`

- when not using `just`: `ansible-playbook -i inventory/hosts setup.yml --tags=COMMA_SEPARATED_TAGS_GO_HERE`

Here are some playbook tags that you should be familiar with:

- `setup-all` - runs all setup tasks (installation and uninstallation) for all components, but does not start/restart services

- `install-all` - like `setup-all`, but skips uninstallation tasks. Useful for maintaining your setup quickly when its components remain unchanged. If you adjust your `vars.yml` to remove components, you'd need to run `setup-all` though, or these components will still remain installed

- `setup-SERVICE` (e.g. `setup-miniflux`) - runs the setup tasks only for a given role ([Miniflux](services/miniflux.md) in this example), but does not start/restart services. You can discover these additional tags in each role (`roles/**/tasks/main.yml`). Running per-component setup tasks is **not recommended**, as components sometimes depend on each other and running just the setup tasks for a given component may not be enough. For example, for setting up the [Miniflux](services/miniflux.md) service, in addition to the `setup-miniflux` tag, changes to the database are also necessary (the `setup-postgres` tag).

- `install-SERVICE` (e.g. `install-miniflux`) - like `setup-SERVICE`, but skips uninstallation tasks. See `install-all` above for additional information.

- `start` - starts all systemd services and makes them start automatically in the future

- `stop` - stops all systemd services

`setup-*` tags and `install-*` tags **do not start services** automatically, because you may wish to do things before starting services, such as importing a database dump, restoring data from another server, etc.

When using `just`, there are also helpful shortcuts you can use:

- `just install-all`: runs all installation tasks and starts/restarts the services

- `just setup-all`: runs all installation tasks and also uninstallation tasks that clean up after services you have removed from your `vars.yml` file. This task also starts/restarts all services.

- `just install-service SERVICE_NAME`: runs the installation tasks only for the `SERVICE_NAME` service and starts/restarts the service. As mentioned above, this is not usually recommended, as installing a service may require running installation tasks for other (dependent) roles to prepare a database, etc.

- `just stop-all`: stops all services

- `just start-all`: starts all services

### 9.2 - Installing services

If you **don't** use SSH keys for authentication, but rather a regular password, you may need to add `--ask-pass` to the all Ansible (or `just`) commands

If you **do** use SSH keys for authentication, **and** use a non-root user to *become* root (sudo), you may need to add `-K` (`--ask-become-pass`) to all Ansible commands

There 2 ways to start the installation process - depending on whether you're [Installing a brand new server (without importing data)](#installing-a-brand-new-server-without-importing-data) or [Installing a server into which you'll import old data](#installing-a-server-into-which-youll-import-old-data).

### 9.2.1 - Installing a brand new server (without importing data)

If this is **a brand new** server and you **won't be importing old data into it**, run all these tags:

```sh
# This is equivalent to: just run-tags install-all,start
just install-all

# Or, when not using just, you can use this instead:
# ansible-playbook -i inventory/hosts setup.yml --tags=install-all,start
```

This will do a full installation and start all services.

Proceed to [Maintaining your setup in the future](#2-maintaining-your-setup-in-the-future) and [Finalize the installation](#3-finalize-the-installation)

### 9.2.2 - Installing a server into which you'll import old data

If you will be importing data into your newly created server, install it, but **do not** start its services just yet.
Starting its services or messing with its database now will affect your data import later on.

To do the installation **without** starting services, run only the `install-all` tag:

```sh
just run-tags install-all

# Or, when not using just, you can use this instead:
# ansible-playbook -i inventory/hosts setup.yml --tags=install-all
```

When this command completes, **services won't be running yet**.

You can now:

- [Importing an existing Postgres database (from another installation)](services/postgres.md#importing) (optional)

.. and then proceed to starting all services:

```sh
# This is equivalent to: just run-tags start (or: just run-tags start-all)
just start-all

# Or, when not using just, you can use this instead:
# ansible-playbook -i inventory/hosts setup.yml --tags=start
```

Proceed to [Maintaining your setup in the future](#2-maintaining-your-setup-in-the-future).

### 9.3 - Maintaining your setup in the future

Feel free to **re-run the setup command any time** you think something is off with the server configuration. Ansible will take your configuration and update your server to match.

Note that if you remove components from `vars.yml`, or if we switch some component from being installed by default to not being installed by default anymore, you'd need to use `setup-all` instead of `install-all`. See [Playbook tags introduction](#playbook-tags-introduction)

To do it with `just`:

```sh
just install-all

# Or, to run potential uninstallation tasks too:
# just setup-all
```

To do it without `just`:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=install-all,start

# Or, to run potential uninstallation tasks too:
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

### 9.4 - Things to do next

After you have started the services, you can:

- start using the configured services
- or set up additional services
- or learn how to [upgrade services when new versions are released](maintenance-upgrading-services.md)
- or come say Hi in our [Matrix](https://matrix.org) support room - [#mash-playbook:devture.com](https://matrix.to/#/#mash-playbook:devture.com). You might learn something or get to help someone else new to hosting services with this playbook.
- or help make this playbook better by contributing (code, documentation, or [coffee/beer](https://liberapay.com/mother-of-all-self-hosting/donate))

## 10 - Importing an existing Postgres database

Follow this section if you'd like to import your database from a previous installation.

### 10.1 - Prerequisites

The playbook supports importing Postgres dump files in **text** (e.g. `pg_dump > dump.sql`) or **gzipped** formats (e.g. `pg_dump | gzip -c > dump.sql.gz`).

Importing multiple databases (as dumped by `pg_dumpall`) is also supported.

Before doing the actual import, **you need to upload your Postgres dump file to the server** (any path is okay).

### 10.2 - Importing a dump file

To import, run this command (make sure to replace `SERVER_PATH_TO_POSTGRES_DUMP_FILE` with a file path on your server):

```sh
just run-tags import-postgres \
--extra-vars=server_path_postgres_dump=SERVER_PATH_TO_POSTGRES_DUMP_FILE \
--extra-vars=postgres_default_import_database=main
```

**Notes**:

- `SERVER_PATH_TO_POSTGRES_DUMP_FILE` must be a file path to a Postgres dump file on the server (not on your local machine!)
- `postgres_default_import_database` defaults to `main`, which is useful for importing multiple databases (for dumps made with `pg_dumpall`). If you're importing a single database (e.g. `miniflux`), consider changing `postgres_default_import_database` to the name of the database (e.g. `miniflux`)
- after importing a large database, it's a good idea to run [an `ANALYZE` operation](https://www.postgresql.org/docs/current/sql-analyze.html) to make Postgres rebuild its database statistics and optimize its query planner. You can easily do this via the playbook by running `just run-tags run-postgres-vacuum -e postgres_vacuum_preset=analyze` (see [Vacuuming PostgreSQL](#vacuuming-postgresql) for more details).

# 11.0 - Upgrading services

This playbook not only installs various services for you, but can also upgrade them as new versions are made available.

To upgrade services:

- update your playbook directory (`git pull`), so you'd obtain everything new we've done

- take a look at [the changelog](../CHANGELOG.md) to see if there have been any backward-incompatible changes that you need to take care of

- download the upstream Ansible roles used by the playbook by running `just roles`

- re-run the [playbook setup](installing.md) and restart all services: `just setup-all`

**Note**: major version upgrades to the internal PostgreSQL database are not done automatically. To upgrade it, refer to the [upgrading PostgreSQL guide](services/postgres.md#upgrading-postgresql).

# 12.0 - Maintenance / Postgres

This section shows you how to perform various maintenance tasks related to the Postgres database server used by various components of this playbook.

Table of contents:

- [Getting a database terminal](#getting-a-database-terminal), for when you wish to execute SQL queries

- [Vacuuming PostgreSQL](#vacuuming-postgresql), for when you wish to run a Postgres [VACUUM](https://www.postgresql.org/docs/current/sql-vacuum.html) (optimizing disk space)

- [Backing up PostgreSQL](#backing-up-postgresql), for when you wish to make a backup

- [Upgrading PostgreSQL](#upgrading-postgresql), for upgrading to new major versions of PostgreSQL. Such **manual upgrades are sometimes required**.

- [Tuning PostgreSQL](#tuning-postgresql) to make it run faster

## 12.1 - Getting a database terminal

You can use the `/mash/postgres/bin/cli` tool to get interactive terminal access ([psql](https://www.postgresql.org/docs/15/app-psql.html)) to the PostgreSQL server.

By default, this tool puts you in the `main` database, which contains nothing.

To see the available databases, run `\list` (or just `\l`).

To change to another database (for example `miniflux`), run `\connect miniflux` (or just `\c miniflux`).

You can then proceed to write queries. Example: `SELECT COUNT(*) FROM users;`

**Be careful**. Modifying the database directly (especially as services are running) is dangerous and may lead to irreversible database corruption.
When in doubt, consider [making a backup](#backing-up-postgresql).

## 12.2 - Vacuuming PostgreSQL

Deleting lots data from Postgres does not make it release disk space, until you perform a [`VACUUM` operation](https://www.postgresql.org/docs/current/sql-vacuum.html).

You can run different `VACUUM` operations via the playbook, with the default preset being `vacuum-complete`:

- (default) `vacuum-complete`: stops all services temporarily and runs `VACUUM FULL VERBOSE ANALYZE`.
- `vacuum-full`: stops all services temporarily and runs `VACUUM FULL VERBOSE`
- `vacuum`: runs `VACUUM VERBOSE` without stopping any services
- `vacuum-analyze` runs `VACUUM VERBOSE ANALYZE` without stopping any services
- `analyze` runs `ANALYZE VERBOSE` without stopping any services (this is just [ANALYZE](https://www.postgresql.org/docs/current/sql-analyze.html) without doing a vacuum, so it's faster)

**Note**: for the `vacuum-complete` and `vacuum-full` presets, you'll need plenty of available disk space in your Postgres data directory (usually `/mash/postgres/data`). These presets also stop all services while the vacuum operation is running.

Example playbook invocations:

- `just run-tags run-postgres-vacuum`: runs the default `vacuum-complete` preset and restarts all services
- `just run-tags run-postgres-vacuum -e postgres_vacuum_preset=analyze`: runs the `analyze` preset with all services remaining operational at all times

## 12.3 - Backing up PostgreSQL

To automatically make Postgres database backups on a fixed schedule, consider enabling the [Postgres Backup](postgres-backup.md) service.

To make a one-off back up of the current PostgreSQL database, make sure it's running and then execute a command like this on the server:

```bash
/usr/bin/docker exec \
--env-file=/mash/postgres/env-postgres-psql \
mash-postgres \
/usr/local/bin/pg_dumpall -h mash-postgres \
| gzip -c \
> /mash/postgres.sql.gz
```

Restoring a backup made this way can be done by [importing it](#importing).

## 12.4 - Upgrading PostgreSQL

Once this playbook installs Postgres for you, it attempts to preserve the Postgres version it starts with.
This is because newer Postgres versions cannot start with data generated by older Postgres versions.

Upgrades must be performed manually.

This playbook can upgrade your existing Postgres setup with the following command:

```sh
just run-tags upgrade-postgres
```

**The old Postgres data directory is backed up** automatically, by renaming it to `/mash/postgres/data-auto-upgrade-backup`.
To rename to a different path, pass some extra flags to the command above, like this: `--extra-vars="postgres_auto_upgrade_backup_data_path=/another/disk/mash-postgres-before-upgrade"`

The auto-upgrade-backup directory stays around forever, until you **manually decide to delete it**.

As part of the upgrade, the database is dumped to `/tmp`, an upgraded and empty Postgres server is started, and then the dump is restored into the new server.
To use a different directory for the dump, pass some extra flags to the command above, like this: `--extra-vars="postgres_dump_dir=/directory/to/dump/here"`

To save disk space in `/tmp`, the dump file is gzipped on the fly at the expense of CPU usage.
If you have plenty of space in `/tmp` and would rather avoid gzipping, you can explicitly pass a dump filename which doesn't end in `.gz`.
Example: `--extra-vars="postgres_dump_name=mash-postgres-dump.sql"`

**All databases, roles, etc. on the Postgres server are migrated**.

## 12.5 - Tuning PostgreSQL

PostgreSQL can be [tuned](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server) to make it run faster. This is done by passing extra arguments to the Postgres process.

The [Postgres Ansible role](https://github.com/devture/com.devture.ansible.role.postgres) **already does some tuning by default**, which matches the [tuning logic](https://github.com/le0pard/pgtune/blob/master/src/features/configuration/configurationSlice.js) done by websites like <https://pgtune.leopard.in.ua/>.
You can manually influence some of the tuning variables . These parameters (variables) are injected via the `devture_postgres_postgres_process_extra_arguments_auto` variable.

Most users should be fine with the automatically-done tuning. However, you may wish to:

- **adjust the automatically-deterimned tuning parameters manually**: change the values for the tuning variables defined in the Postgres role's [default configuration file](https://github.com/devture/com.devture.ansible.role.postgres/blob/main/defaults/main.yml) (see `devture_postgres_max_connections`, `devture_postgres_data_storage` etc). These variables are ultimately passed to Postgres via a `devture_postgres_postgres_process_extra_arguments_auto` variable

- **turn automatically-performed tuning off**: override it like this: `devture_postgres_postgres_process_extra_arguments_auto: []`

- **add additional tuning parameters**: define your additional Postgres configuration parameters in `devture_postgres_postgres_process_extra_arguments_custom`. See `devture_postgres_postgres_process_extra_arguments_auto` defined in the Postgres role's [default configuration file](https://github.com/devture/com.devture.ansible.role.postgres/blob/main/defaults/main.yml) for inspiration

## 12.6 - Recommended other services

You may also wish to look into:

- [Postgres Backup](postgres-backup.md) for backing up your Postgres database

- [Prometheus](prometheus.md), [prometheus-postgres-exporter](prometheus-postgres-exporter.md) and [Grafana](grafana.md) for monitoring your Postgres database

# 13.0 - Maintenance and Troubleshooting

## 13.1 - How to see the current status of your services

You can check the status of your services by using `systemctl status`. Example:

```
sudo systemctl status mash-miniflux

● mash-miniflux.service - Miniflux (mash-miniflux)
   Loaded: loaded (/etc/systemd/system/mash-miniflux.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2023-03-14 17:41:59 EET; 15h ago
```

You can see the logs by using journalctl. Example:

```
sudo journalctl -fu mash-miniflux
```

## 13.2 - Increasing logging

Various Ansible roles for various services supported by this playbook support a `*_log_level` variable or some debug mode which you can enable in your configuration and get extended logs.

[Re-run the playbook](installing.md) after making such configuration changes.

## 13.3 - Remove unused Docker data

You can free some disk space from Docker by running `docker system prune -a` on the server.

## 13.4 - Postgres

See the dedicated [PostgreSQL Maintenance](services/postgres.md#maintenance) documentation page.

# 14.0 - Developer Documentation

## 14.1 - Support a new service | Create your own role

### 14.1.1 Check if

- the role doesn't exist already in [`supported-services.md`](supported-services.md) and no one else is already [working on it](https://github.com/mother-of-all-self-hosting/mash-playbook/pulls)

- the service you wish to add can run in a Docker container

- a container image already exists

### 14.1.2 Create the Ansible role in a public git repository

You can follow the structure of existing roles, like the [`ansible-role-gitea`](https://github.com/mother-of-all-self-hosting/ansible-role-gitea) or the [`ansible-role-gotosocial`](https://github.com/mother-of-all-self-hosting/ansible-role-gotosocial).

Some advice:

- Your file structure will probably look something like this:

```
.
├── defaults/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── install.yml
│   ├── uninstall.yml
│   └── validate_config.yml
├── templates/
│   ├── env.j2
│   ├── labels.j2
│   └── NEW-SERVICE.service.j2
├── .gitignore
├── LICENSE
└── README.md
```

- You will need to decide on a licence, without it, ansible-galaxy won't work. We recommend AGPLv3, like all of MASH.

### 14.1.3 Update the MASH-Playbook to support your created Ansible role

There are a few files that you need to adapt:

```
.
├── docs/
│   ├── supported-services.md  -> Add your service
│   └── services/
│       └── YOUR-SERVICE.md  -> document how to use it
├── templates/
│   ├── group_vars_mash_servers  -> Add default config
│   └── requirements.yml  -> add your Ansible role
│   └── setup.yml  -> add your Ansible role
```

<details>

<summary> file: templates / group_vars_mash_servers </summary>
In this file you wire your role with the rest of the playbook - integrating with the service manager or potentially with other roles.

```yaml
# role-specific:systemd_service_manager
########################################################################
#                                                                      #
# systemd_service_manager                                              #
#                                                                      #
########################################################################

mash_playbook_devture_systemd_service_manager_services_list_auto_itemized:
  [...]
  # role-specific:YOUR-SERVICE
  - |-
    {{ ({'name': (YOUR-SERVICE_identifier + '.service'), 'priority': 2000, 'groups': ['mash', 'YOUR-SERVICE']} if YOUR-SERVICE_enabled else omit) }}
  # /role-specific:YOUR-SERVICE

[...]
########################################################################
#                                                                      #
# /systemd_service_manager                                             #
#                                                                      #
########################################################################
# /role-specific:systemd_service_manager

```

</details>

<details>
<summary>Support Postgres</summary>

```yaml
# role-specific:postgres
########################################################################
#                                                                      #
# postgres                                                             #
#                                                                      #
########################################################################
[...]

mash_playbook_devture_postgres_managed_databases_auto_itemized:
  [...]
  # role-specific:YOUR-SERVICE
  - |-
    {{
      ({
        'name': YOUR-SERVICE_database_name,
        'username': YOUR-SERVICE_database_username,
        'password': YOUR-SERVICE_database_password,
      } if gYOUR-SERVICE_enabled else omit)
    }}
  # /role-specific:YOUR-SERVICE
  
  [...]
########################################################################
#                                                                      #
# /postgres                                                            #
#                                                                      #
########################################################################
# /role-specific:postgres

[...]

# role-specific:YOUR-SERVICE
########################################################################
#                                                                      #
# YOUR-SERVICE                                                         #
#                                                                      #
########################################################################

[...]

# role-specific:postgres
YOUR-SERVICE_database_hostname: "{{ devture_postgres_identifier if devture_postgres_enabled else '' }}"
YOUR-SERVICE_database_port: "{{ '5432' if devture_postgres_enabled else '' }}"
YOUR-SERVICE_database_password: "{{ '%s' | format(mash_playbook_generic_secret_key) | password_hash('sha512', 'db.authentik', rounds=655555) | to_uuid }}"
YOUR-SERVICE_database_username: "{{ authentik_identifier }}"
# /role-specific:postgres

YOUR-SERVICE_container_additional_networks_auto: |
  {{
    ([devture_postgres_identifier ~ '.service'] if devture_postgres_enabled and YOUR-SERVICE_database_hostname == devture_postgres_identifier else [])
  }}
  
########################################################################
#                                                                      #
# /YOUR-SERVICE                                                        #
#                                                                      #
########################################################################
# /role-specific:YOUR-SERVICE
```

</details><details>
<summary>Support exim-relay</summary>

The [exim-relay](https://github.com/devture/exim-relay) is an easy way to configure for all services a way for outgoing mail.

```yaml
[...]

# role-specific:YOUR-SERVICE
########################################################################
#                                                                      #
# YOUR-SERVICE                                                         #
#                                                                      #
########################################################################

[...]

YOUR-SERVICE_container_additional_networks_auto: |
  {{
 [...]
 +
    ([exim_relay_container_network | default('mash-exim-relay')] if (exim_relay_enabled | default(false) and YOUR-SERVICE_config_mailer_smtp_addr == exim_relay_identifier | default('mash-exim-relay') and YOUR-SERVICE_container_network != exim_relay_container_network) else [])
  }}

# role-specific:exim_relay
YOUR-SERVICE_config_mailer_enabled: "{{ 'true' if exim_relay_enabled else '' }}"
YOUR-SERVICE_config_mailer_smtp_addr: "{{ exim_relay_identifier if exim_relay_enabled else '' }}"
YOUR-SERVICE_config_mailer_smtp_port: 8025
YOUR-SERVICE_config_mailer_from: "{{ exim_relay_sender_address if exim_relay_enabled else '' }}"
YOUR-SERVICE_config_mailer_protocol: "{{ 'smtp' if exim_relay_enabled else '' }}"
# /role-specific:exim_relay
  
########################################################################
#                                                                      #
# /YOUR-SERVICE                                                        #
#                                                                      #
########################################################################
# /role-specific:YOUR-SERVICE
```

</details><details>

<summary>Support hubsite</summary>

- Add the logo of your Service to [`ansible-role-hubsite/assets`](https://github.com/mother-of-all-self-hosting/ansible-role-hubsite/tree/main/assets) via a pull request.
- configure the `group_vars_mash_servers` file:

```yaml
[...]
# role-specific:hubsite
########################################################################
#                                                                      #
# hubsite                                                              #
#                                                                      #
########################################################################
[...]

# Services
##########
[...]

# role-specific:YOUR-SERVICE
# YOUR-SERVICE
hubsite_service_YOUR-SERVICE_enabled: "{{ YOUR-SERVICE_enabled }}"
hubsite_service_YOUR-SERVICE_name: Adguard Home
hubsite_service_YOUR-SERVICE_url: "https://{{ YOUR-SERVICE_hostname }}{{ YOUR-SERVICE_path_prefix }}"
hubsite_service_YOUR-SERVICE_logo_location: "{{ role_path }}/assets/YOUR-SERVICE.png"
hubsite_service_YOUR-SERVICE_description: "YOUR-SERVICE Description"
hubsite_service_YOUR-SERVICE_priority: 1000
# /role-specific:YOUR-SERVICE
[...]

mash_playbook_hubsite_service_list_auto_itemized:
  [...]
  # role-specific:YOUR-SERVICE
  - |-
    {{
      ({
        'name': hubsite_service_YOUR-SERVICE_name,
        'url': hubsite_service_YOUR-SERVICE_url,
        'logo_location': hubsite_service_YOUR-SERVICE_logo_location,
        'description': hubsite_service_YOUR-SERVICE_description,
        'priority': hubsite_service_YOUR-SERVICE_priority,
      } if hubsite_service_YOUR-SERVICE_enabled else omit)
    }}
  # /role-specific:YOUR-SERVICE
[...]
```

</details>

### 14.2 - Additional hints

- Add a line like `# Project source code URL: YOUR-SERVICE-GIT-REPO` to your Ansible role's `defaults/main.yml` file, so that [`bin/feeds.py`](/bin/feeds.py) can automatically find the Atom/RSS feed for new releases.

# 15.0 - Uninstalling

**Warning**: If you have some trouble with your installation, you can just [re-run the playbook](installing.md) and it will try to set things up again. **Uninstalling and then installing anew rarely solves anything**.

To uninstall, run these commands (most are meant to be executed on the server itself):

- ensure all services are stopped: `just stop` (if you can't get Ansible working to run this command, you can run `systemctl stop 'mash-*'` manually on the server)

- delete the systemd `.service` and `.timer` files (`rm -f /etc/systemd/system/mash-*.{service,timer}`) and reload systemd (`systemctl daemon-reload`)

- delete some cached Docker images (`docker system prune -a`) or just delete them all (`docker rmi $(docker images -aq)`)

- uninstall Docker itself, if necessary

- delete the `/mash` directory (`rm -rf /mash`)
