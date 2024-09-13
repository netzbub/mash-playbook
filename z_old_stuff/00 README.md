# mother-of-all-self-hosting / mash-playbook

## Table of Contents
- [mother-of-all-self-hosting / mash-playbook](#mother-of-all-self-hosting--mash-playbook)
  - [Table of Contents](#table-of-contents)
- [1.0 - Prerequisites](#10---prerequisites)
  - [1.1 - Requierements](#11---requierements)
  - [1.2 - Alternative Architectures](#12---alternative-architectures)
  - [1.3 - Implementation details](#13---implementation-details)
- [2.0 - Configuring DNS](#20---configuring-dns)
- [3.0 - Getting the playbook](#30---getting-the-playbook)
  - [3.1 - Using git to get the playbook](#31---using-git-to-get-the-playbook)
  - [3.2 - Downloading the playbook as a ZIP archive](#32---downloading-the-playbook-as-a-zip-archive)
- [4.0 - Ansible](#40---ansible)
  - [4.1 - Running this playbook](#41---running-this-playbook)
  - [4.2 - Supported Ansible versions](#42---supported-ansible-versions)
  - [4.3 - Upgrading Ansible](#43---upgrading-ansible)
  - [4.4 - Alternative ways to run Ansible](#44---alternative-ways-to-run-ansible)
  - [4.4.1 - Running Ansible in a container on the server itself](#441---running-ansible-in-a-container-on-the-server-itself)
  - [4.4.2 - Running Ansible in a container on another computer (not the server)](#442---running-ansible-in-a-container-on-another-computer-not-the-server)
  - [4.4.3 - If you don't use SSH keys for authentication](#443---if-you-dont-use-ssh-keys-for-authentication)
  - [4.4.4 - Resolve directory ownership issues](#444---resolve-directory-ownership-issues)
- [5.0 - Configuring the Ansible playbook](#50---configuring-the-ansible-playbook)
- [6.0 - Configuring interoperability with other services](#60---configuring-interoperability-with-other-services)
  - [6.1 - Disabling Traefik installation](#61---disabling-traefik-installation)
  - [6.2 - Disabling  installation](#62---disabling--installation)
  - [6.3 - Disabling timesyncing (systemd-timesyncd / ntp) installation](#63---disabling-timesyncing-systemd-timesyncd--ntp-installation)
- [7.0 - Multiple instances](#70---multiple-instances)
  - [7.1 - Running multiple instances of the same service on the same host](#71---running-multiple-instances-of-the-same-service-on-the-same-host)
  - [7.2 - Re-do your inventory to add supplementary hosts](#72---re-do-your-inventory-to-add-supplementary-hosts)
  - [7.3 - Adjust the configuration of the supplementary hosts to use a new "namespace"](#73---adjust-the-configuration-of-the-supplementary-hosts-to-use-a-new-namespace)
  - [7.4 - Adjust the configuration of the base host](#74---adjust-the-configuration-of-the-base-host)
  - [7.5 - Questions & Answers](#75---questions--answers)
- [8.0 - Self-building](#80---self-building)
- [9.0 - Installing](#90---installing)
  - [9.1 - Playbook tags introduction](#91---playbook-tags-introduction)
  - [9.2 - Installing services](#92---installing-services)
  - [9.2.1 - Installing a brand new server (without importing data)](#921---installing-a-brand-new-server-without-importing-data)
  - [9.2.2 - Installing a server into which you'll import old data](#922---installing-a-server-into-which-youll-import-old-data)
  - [9.3 - Maintaining your setup in the future](#93---maintaining-your-setup-in-the-future)
  - [9.4 - Things to do next](#94---things-to-do-next)
- [10.0 - Importing an existing Postgres database](#100---importing-an-existing-postgres-database)
  - [10.1 - Prerequisites](#101---prerequisites)
  - [10.2 - Importing a dump file](#102---importing-a-dump-file)
- [12.1 - Getting a database terminal](#121---getting-a-database-terminal)
- [12.2 - Vacuuming PostgreSQL](#122---vacuuming-postgresql)
- [12.3 - Backing up PostgreSQL](#123---backing-up-postgresql)
- [12.4 - Upgrading PostgreSQL](#124---upgrading-postgresql)
- [12.5 - Tuning PostgreSQL](#125---tuning-postgresql)
- [12.6 - Recommended other services](#126---recommended-other-services)
- [13.1 - How to see the current status of your services](#131---how-to-see-the-current-status-of-your-services)
- [13.2 - Increasing logging](#132---increasing-logging)
- [13.3 - Remove unused Docker data](#133---remove-unused-docker-data)
- [13.4 - Postgres](#134---postgres)
- [14.1 - Support a new service | Create your own role](#141---support-a-new-service--create-your-own-role)
  - [14.1.1 Check if](#1411-check-if)
  - [14.1.2 Create the Ansible role in a public git repository](#1412-create-the-ansible-role-in-a-public-git-repository)
  - [14.1.3 Update the MASH-Playbook to support your created Ansible role](#1413-update-the-mash-playbook-to-support-your-created-ansible-role)
  - [14.2 - Additional hints](#142---additional-hints)
- [15.0 - Uninstalling](#150---uninstalling)
- [16.0 - Services](#160---services)
  - [16.1 - Supported Services](#161---supported-services)
  - [16.2 - Related playbooks](#162---related-playbooks)
<!-- /TOC -->

## 1.0 - Prerequisites

### 1.1 - Requierements

To install services using this Ansible playbook, you need:

- (Recommended) An **x86-64** (`amd64`) or **arm64** server running one of these operating systems:
  - **Red Hat Enterprise Linux** or derivative distros, e.g. Rocky Linux (Major version 7 or newer)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](#42---supported-ansible-versions))
  - **Archlinux**

Generally, newer is better. We only strive to support released stable versions of distributions, not betas or pre-releases. This playbook can take over your whole server or co-exist with other services that you have there.

This playbook somewhat supports running on non-`amd64` architectures like ARM. See [Alternative Architectures](#12---alternative-architectures).

If your distro runs within an [LXC container](https://linuxcontainers.org/), you may hit [this issue](https://github.com/spantaleev/matrix-docker-ansible-deploy/issues/703). It can be worked around, if absolutely necessary, but we suggest that you avoid running from within an LXC container.

- `root` access to your server (or a user capable of elevating to `root` via `sudo`).

- [Python](https://www.python.org/) being installed on the server. Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`). On some distros, Ansible may incorrectly [detect the Python version](https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html) (2 vs 3) and you may need to explicitly specify the interpreter path in `inventory/hosts` during installation (e.g. `ansible_python_interpreter=/usr/bin/python3`)

- [sudo](https://www.sudo.ws/) being installed on the server, even when you've configured Ansible to log in as `root`. Some distributions, like a minimal Debian net install, do not include the `sudo` package by default.

- The [Ansible](http://ansible.com/) program being installed on your own computer. It's used to run this playbook and configures your server for you. Take a look at [our guide about Ansible](#40---ansible) for more information, as well as [version requirements](#42---supported-ansible-versions) and [alternative ways to run Ansible](#44---alternative-ways-to-run-ansible).

- the [passlib](https://passlib.readthedocs.io/en/stable/index.html) Python library installed on the computer you run Ansible. On most distros, you need to install some `python-passlib` or `py3-passlib` package, etc.

- [git](https://git-scm.com/) is the recommended way to download the playbook to your computer. `git` may also be required on the server if you will be [self-building](#80---self-building) components.

- [just](https://github.com/casey/just) for running `just update` and playbook installation commands, etc. (see [`justfile`](../justfile)). You can get by without `just` (by running `ansible-galaxy`, `ansible-playbook` commands manually), but maintaining your playbook setup will require more manual work. `just` (thanks to the commands defined in the `justfile`) keeps various files (`setup.yml`, `requirements.yml`, `group_vars/mash_servers`) up-to-date with the templates in [the `templates/` directory](../templates/) automatically.

- at least one domain name you can use

- Some TCP/UDP ports open. This playbook (actually [Docker itself](https://docs.docker.com/network/iptables/)) configures the server's internal firewall for you. In most cases, you don't need to do anything special. But **if your server is running behind another firewall**, you'd need to open these ports:

  - `80/tcp`: HTTP webserver
  - `443/tcp`: HTTPS webserver
  - potentially some other ports, depending on the services that you enable in the **configuring the playbook** step (later on). Consult each service's documentation page in `docs/` for that.

### 1.2 - Alternative Architectures

As stated in the [Requirements](#11---requierements), currently only `amd64` (`x86_64`) is fully supported.

The playbook automatically determines the target server's architecture (the `mash_playbook_architecture` variable) to be one of the following:

- `amd64` (`x86_64`)
- `arm32`
- `arm64`

Some tools and container images can be built on the host or other measures can be used to install on that architecture.

### 1.3 - Implementation details

For `amd64`, prebuilt container images are used for all components.

For other architecture (`arm64`, `arm32`), components which have a prebuilt image make use of it. If the component is not available for the specific architecture, [self-building](#80---self-building) will be used. Not all components support self-building though, so your mileage may vary.

## 2.0 - Configuring DNS

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

When you're done configuring DNS, proceed to [Configuring the playbook](#50---configuring-the-ansible-playbook).

## 3.0 - Getting the playbook

This Ansible playbook is meant to be executed on your own computer (not on the server).

In special cases (if your computer cannot run Ansible, etc.) you may put the playbook on the server as well.

You can retrieve the playbook's source code by:

- [Using git to get the playbook](#31---using-git-to-get-the-playbook)

- [Downloading the playbook as a ZIP archive](#32---downloading-the-playbook-as-a-zip-archive) (not recommended)

### 3.1 - Using git to get the playbook

We recommend using the [git](https://git-scm.com/) tool to get the playbook's source code, because it lets you easily keep up to date in the future when [Maintaining services](maintenance-upgrading-services.md).

Once you've installed git on your computer, you can go to any directory of your choosing and run the following command to retrieve the playbook's source code:

```bash
git clone https://github.com/mother-of-all-self-hosting/mash-playbook.git
```

This will create a new `mash-playbook` directory.
You're supposed to execute all other installation commands inside that directory.

### 3.2 - Downloading the playbook as a ZIP archive

Alternatively, you can download the playbook as a ZIP archive.
This is not recommended, as it's not easy to keep up to date with future updates. We suggest you [use git](#using-git-to-get-the-playbook) instead.

The latest version is always at the following URL: <https://github.com/mother-of-all-self-hosting/mash-playbook/archive/master.zip>

You can extract this archive anywhere. You'll get a directory called `mash-playbook-master`.
You're supposed to execute all other installation commands inside that directory.

---------------------------------------------

No matter which method you've used to download the playbook, you can proceed by [Configuring the playbook](configuring-playbook.md).

## 4.0 - Ansible

### 4.1 - Running this playbook

This playbook is meant to be run using [Ansible](https://www.ansible.com/).

Ansible typically runs on your local computer and carries out tasks on a remote server.
If your local computer cannot run Ansible, you can also run Ansible on some server somewhere (including the server you wish to install to).

### 4.2 - Supported Ansible versions

To manually check which version of Ansible you're on, run: `ansible --version`.

For the **best experience**, we recommend getting the **latest version of Ansible available**.

We're not sure what's the minimum version of Ansible that can run this playbook successfully.
The lowest version that we've confirmed (on 2022-11-26) to be working fine is: `ansible-core` (`2.11.7`) combined with `ansible` (`4.10.0`).

If your distro ships with an Ansible version older than this, you may run into issues. Consider [Upgrading Ansible](#43---upgrading-ansible) or [using Ansible via](#44---alternative-ways-to-run-ansible).

### 4.3 - Upgrading Ansible

Depending on your distribution, you may be able to upgrade Ansible in a few different ways:

- by using an additional repository (PPA, etc.), which provides newer Ansible versions. See instructions for [CentOS](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-rhel-centos-or-fedora), [Debian](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-debian), or [Ubuntu](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) on the Ansible website.

- by removing the Ansible package (`yum remove ansible` or `apt-get remove ansible`) and installing via [pip](https://pip.pypa.io/en/stable/installation/) (`pip install ansible`).

If using the `pip` method, do note that the `ansible-playbook` binary may not be on the `$PATH` (<https://linuxconfig.org/linux-path-environment-variable>), but in some more special location like `/usr/local/bin/ansible-playbook`. You may need to invoke it using the full path.

**Note**: Both of the above methods are a bad way to run system software such as Ansible.
If you find yourself needing to resort to such hacks, please consider reporting a bug to your distribution and/or switching to a sane distribution, which provides up-to-date software.

### 4.4 - Alternative ways to run Ansible

Alternatively, you can run Ansible inside a  container (powered by the [devture/ansible](https://hub..com/r/devture/ansible/)  image).

This ensures that you're using a very recent Ansible version, which is less likely to be incompatible with the playbook.

You can either [run Ansible in a container on the server itself](#441---running-ansible-in-a-container-on-the-server-itself) or [run Ansible in a container on another computer (not the server)](#442---running-ansible-in-a-container-on-another-computer-not-the-server).

### 4.4.1 - Running Ansible in a container on the server itself

To run Ansible in a (Docker) container on the server itself, you need to have a working  installation.
 is normally installed by the playbook, so this may be a bit of a chicken and egg problem. To solve it:

- you **either** need to install [Docker](services/ansible.md) manually first. Follow [the upstream instructions](https://docs.docker.com/engine/install/)  for your distribution and consider setting `mash_playbook__installation_enabled: false` in your `vars.yml` file, to prevent the playbook from installing
- **or** you need to run the playbook in another way (e.g. [Running Ansible in a container on another computer (not the server)](#442---running-ansible-in-a-container-on-another-computer-not-the-server)) at least the first time around.

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

### 4.4.2 - Running Ansible in a container on another computer (not the server)

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

First, consider running `git config --global --add safe.directory /work` to [resolve directory ownership issues](#444---resolve-directory-ownership-issues).

Finally, you execute `just` or `ansible-playbook ...` commands as per normal now.

### 4.4.3 - If you don't use SSH keys for authentication

If you don't use SSH keys for authentication, simply remove that whole line (`-v $HOME/.ssh/id_rsa:/root/.ssh/id_rsa:ro`).
To authenticate at your server using a password, you need to add a package. So, when you are in the shell of the ansible  container (the previously used `run -it ...` command), run:

```bash
apk add sshpass
```

Then, to be asked for the password whenever running an  `ansible-playbook` command add `--ask-pass` to the arguments of the command.

### 4.4.4 - Resolve directory ownership issues

Because you're `root` in the container running Ansible and this likely differs fom the owner (your regular user account) of the playbook directory outside of the container, certain playbook features which use `git` locally may report warnings such as:

> fatal: unsafe repository ('/work' is owned by someone else)
> To add an exception for this directory, call:
> git config --global --add safe.directory /work

These errors can be resolved by making `git` trust the playbook directory by running `git config --global --add safe.directory /work`

## 5.0 - Configuring the Ansible playbook

To configure the playbook, you need to have done the following things:

- have a server where services will run
- [retrieved the playbook's source code](#30---getting-the-playbook) to your computer

You can then follow these steps inside the playbook directory:

1. create a directory to hold your configuration (`mkdir -p inventory/host_vars/<your-domain>`)

2. copy the sample configuration file (`cp examples/vars.yml inventory/host_vars/<your-domain>/vars.yml`)

3. edit the configuration file (`inventory/host_vars/<your-domain>/vars.yml`) to your liking. You should [enable one or more services](#161---supported-services) in your `vars.yml` file. You may also take a look at the various `roles/**/ROLE_NAME_HERE/defaults/main.yml` files (after importing external roles with `just update` into `roles/galaxy`) and see if there's something you'd like to copy over and override in your `vars.yml` configuration file.

4. copy the sample inventory hosts file (`cp examples/hosts inventory/hosts`)

5. edit the inventory hosts file (`inventory/hosts`) to your liking

If you're installing services on the same server using another playbook (like [matrix--ansible-deploy](https://github.com/spantaleev/matrix--ansible-deploy)) or you already have [Traefik](./services/traefik.md) or [Docker](./services/docker.md) installed on the server, consult the next chapter about interoperability.

## 6.0 - Configuring interoperability with other services

This playbook tries to get you up and running with minimal effort and provided you have followed the [example `vars.yml` file](../examples/vars.yml), will install the [Traefik](services/traefik.md) reverse-proxy server by default.

Sometimes, you're using a server which already has Traefik. In such cases these are undesirable:

- the playbook trying to run its own Traefik instance and running into a conflict with your other Traefik instance over ports (`tcp/80` and `tcp/443`)

- multiple playbooks trying to install , etc.

Below, we offer some suggestions for how to make this playbook more interoperable. Feel free to cherry-pick the parts that make sense for your setup.

### 6.1 - Disabling Traefik installation

If you're installing [Traefik](services/traefik.md) on your server in another way, you can use your already installed Traefik instance and [disable the Traefik instance installed by MASH](services/traefik.md#using-another-traefik-instance-not-installing-traefik).

If you are using the [matrix--ansible-deploy](https://github.com/spantaleev/matrix--ansible-deploy) playbook, it already runs its own Traefik instance (`matrix-traefik`). We recommend that you [disable the Traefik instance installed by MASH](services/traefik.md#using-another-traefik-instance-not-installing-traefik), because the Traefik instance installed by the Matrix playbook does the same, but also contains additional configuration for handling the Matrix federation port (`8448`).

### 6.2 - Disabling  installation

If you're installing [Docker](https://www.docker.com/) on your server in another way, disable this component from the playbook:

```yaml
mash_playbook__installation_enabled: false
```

### 6.3 - Disabling timesyncing (systemd-timesyncd / ntp) installation

If you're installing `systemd-timesyncd` or `ntp` on your server in another way, disable this component from the playbook:

```yaml
devture_timesync_installation_enabled: false
```

## 7.0 - Multiple instances

### 7.1 - Running multiple instances of the same service on the same host

The way this playbook is structured, each Ansible role can only be invoked once and made to install one instance of the service it's responsible for.

If you need multiple instances (of whichever service), you'll need some workarounds as described below.

The example below focuses on hosting multiple [KeyDB](services/keydb.md) instances, but you can apply it to hosting multiple instances or whole stacks of any kind.

Let's say you're managing a host called `mash.example.com` which installs both [PeerTube](services/peertube.md) and [NetBox](services/netbox.md). Both of these services require a [KeyDB](services/keydb.md) instance. If you simply add `keydb_enabled: true` to your `mash.example.com` host's `vars.yml` file, you'd get a KeyDB instance (`mash-keydb`), but it's just one instance. As described in our [KeyDB](services/keydb.md) documentation, this is a security problem and potentially fragile as both services may try to read/write the same data and get in conflict with one another.

We propose that you **don't** add `keydb_enabled: true` to your main `mash.example.com` file, but do the following:

### 7.2 - Re-do your inventory to add supplementary hosts

Create multiple hosts in your inventory (`inventory/hosts`) which target the same server, like this:

```ini
[mash_servers]
[mash_servers:children]
mash_example_com

[mash_example_com]
mash.example.com-netbox-deps ansible_host=1.2.3.4
mash.example.com-peertube-deps ansible_host=1.2.3.4
mash.example.com ansible_host=1.2.3.4
```

This creates a new group (called `mash_example_com`) which groups all 3 hosts:

- (**new**) `mash.example.com-netbox-deps` - a new host, for your [NetBox](services/netbox.md) dependencies
- (**new**) `mash.example.com-peertube-deps` - a new host, for your [PeerTube](services/peertube.md) dependencies
- (old) `mash.example.com` - your regular inventory host

When running Ansible commands later on, you can use the `-l` flag to limit which host to run them against. Here are a few examples:

- `just install-all` - runs the [installation](#90---installing) process on all hosts (3 hosts in this case)
- `just install-all -l mash_example_com` - runs the installation process on all hosts in the `mash_example_com` group (same 3 hosts as `just install-all` in this case)
- `just install-all -l mash.example.com-netbox-deps` - runs the installation process on the `mash.example.com-netbox-deps` host


### 7.3 - Adjust the configuration of the supplementary hosts to use a new "namespace"

Multiple hosts targetting the same server as described above still causes conflicts, because services will use the same paths (e.g. `/mash/keydb`) and service/container names (`mash-keydb`) everywhere.

To avoid conflicts, adjust the `vars.yml` file for the new hosts (`mash.example.com-netbox-deps` and `mash.example.com-peertube-deps`)
and set non-default and unique values in the `mash_playbook_service_identifier_prefix` and `mash_playbook_service_base_directory_name_prefix` variables. Examples below:

`inventory/host_vars/mash.example.com-netbox-deps/vars.yml`:

```yaml
---

########################################################################
#                                                                      #
# Playbook                                                             #
#                                                                      #
########################################################################

# Put a strong secret below, generated with `pwgen -s 64 1` or in another way
# Various other secrets will be derived from this secret automatically.
mash_playbook_generic_secret_key: ''

# Override service names and directory path prefixes
mash_playbook_service_identifier_prefix: 'mash-netbox-'
mash_playbook_service_base_directory_name_prefix: 'netbox-'

########################################################################
#                                                                      #
# /Playbook                                                            #
#                                                                      #
########################################################################


########################################################################
#                                                                      #
# keydb                                                                #
#                                                                      #
########################################################################

keydb_enabled: true

########################################################################
#                                                                      #
# /keydb                                                               #
#                                                                      #
########################################################################
```

`inventory/host_vars/mash.example.com-peertube-deps/vars.yml`:

```yaml
---

########################################################################
#                                                                      #
# Playbook                                                             #
#                                                                      #
########################################################################

# Put a strong secret below, generated with `pwgen -s 64 1` or in another way
# Various other secrets will be derived from this secret automatically.
mash_playbook_generic_secret_key: ''

# Override service names and directory path prefixes
mash_playbook_service_identifier_prefix: 'mash-peertube-'
mash_playbook_service_base_directory_name_prefix: 'peertube-'

########################################################################
#                                                                      #
# /Playbook                                                            #
#                                                                      #
########################################################################


########################################################################
#                                                                      #
# keydb                                                                #
#                                                                      #
########################################################################

keydb_enabled: true

########################################################################
#                                                                      #
# /keydb                                                               #
#                                                                      #
########################################################################
```

The above configuration will create **2** KeyDB instances:

- `mash-netbox-keydb` with its base data path in `/mash/netbox-keydb`
- `mash-peertube-keydb` with its base data path in `/mash/peertube-keydb`

These instances reuse the `mash` user and group and the `/mash` data path, but are not in conflict with each other.


### 7.4 - Adjust the configuration of the base host

Now that we've created separate KeyDB instances for both PeerTube and NetBox, we need to put them to use by editing the `vars.yml` file of the main host (the one that installs PeerTbue and NetBox) to wire them to their KeyDB instances.

You'll need configuration (`inventory/host_vars/mash.example.com/vars.yml`) like this:

```yaml
########################################################################
#                                                                      #
# netbox                                                               #
#                                                                      #
########################################################################

netbox_enabled: true

# Other NetBox configuration here

# Point NetBox to its dedicated KeyDB instance
netbox_environment_variable_redis_host: mash-netbox-keydb
netbox_environment_variable_redis_cache_host: mash-netbox-keydb

# Make sure the NetBox service (mash-netbox.service) starts after its dedicated KeyDB service (mash-netbox-keydb.service)
netbox_systemd_required_services_list_custom:
  - mash-netbox-keydb.service

# Make sure the NetBox container is connected to the container network of its dedicated KeyDB service (mash-netbox-keydb)
netbox_container_additional_networks_custom:
  - mash-netbox-keydb

########################################################################
#                                                                      #
# /netbox                                                              #
#                                                                      #
########################################################################


########################################################################
#                                                                      #
# peertube                                                             #
#                                                                      #
########################################################################

# Other PeerTube configuration here

# Point PeerTube to its dedicated KeyDB instance
peertube_config_redis_hostname: mash-peertube-keydb

# Make sure the PeerTube service (mash-peertube.service) starts after its dedicated KeyDB service (mash-peertube-keydb.service)
peertube_systemd_required_services_list_custom:
  - "mash-peertube-keydb.service"

# Make sure the PeerTube container is connected to the container network of its dedicated KeyDB service (mash-peertube-keydb)
peertube_container_additional_networks_custom:
  - "mash-peertube-keydb"

########################################################################
#                                                                      #
# /peertube                                                            #
#                                                                      #
########################################################################
```

### 7.5 - Questions & Answers

**Can't I just use the same KeyDB instance for multiple services?**

> You may or you may not. See the [KeyDB](services/keydb.md) documentation for why you shouldn't do this.

**Can't I just create one host and a separate stack for each service** (e.g. Nextcloud + all dependencies on one inventory host; PeerTube + all dependencies on another inventory host; with both inventory hosts targetting the same server)?

> That's a possibility which is somewhat clean. The downside is that each "full stack" comes with its own Postgres database which needs to be maintained and upgraded separately.

## 8.0 - Self-building

**Caution: self-building does not have to be used on its own. See the [Alternative Architectures](#12---alternative-architectures) page.**

The playbook supports self-building of various components, which don't have a container image for your architecture. For `amd64`, self-building is not required.

For other architectures (e.g. `arm32`, `arm64`), ready-made container images are used when available. If there's no ready-made image for a specific component and said component supports self-building, an image will be built on the host. Building images like this takes more time and resources (some build tools need to get installed by the playbook to assist building).

To make use of self-building, you don't need to do anything. If a component has an image for the specified architecture, the playbook will use it directly. If not, it will build the image on the server itself.

Note that **not all components support self-building yet**.

Adding self-building support to other roles is welcome. Feel free to contribute!

If you'd like **to force self-building** even if an image is available for your architecture, look into the `*_self_build` variables provided by individual roles.

## 9.0 - Installing

If you've [configured the playbook](#50---configuring-the-ansible-playbook) and have prepared the required domains [(DNS records)](#20---configuring-dns) depending on the services you've enabled, you can start the installation procedure.

This playbook makes use of the [`just`](https://github.com/casey/just) utility to make it easier to run playbook-related commands defined in the [`justfile`](../justfile).
We recommend installing and using `just` - otherwise, you'll need to do more manual work.

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

There are 2 ways to start the installation process - depending on whether you're Installing a brand new server (without importing data) or Installing a server into which you'll import old data - see the following two chapters.

### 9.2.1 - Installing a brand new server (without importing data)

If this is **a brand new** server and you **won't be importing old data into it**, run all these tags:

```sh
# This is equivalent to: just run-tags install-all,start
just install-all

# Or, when not using just, you can use this instead:
# ansible-playbook -i inventory/hosts setup.yml --tags=install-all,start
```

This will do a full installation and start all services.

Proceed to [Maintaining your setup in the future](#93---maintaining-your-setup-in-the-future) and [Finalize the installation](#3-finalize-the-installation) .

<!-- ( C A N N O T  F I N D  R E F E R E N C E  F O R  O R I G I N A L  L I N K )-->

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

### 9.3 - Maintaining your setup in the future

Feel free to **re-run the setup command any time** you think something is off with the server configuration. Ansible will take your configuration and update your server to match.

Note that if you remove components from `vars.yml`, or if we switch some component from being installed by default to not being installed by default anymore, you'd need to use `setup-all` instead of `install-all`. See [Playbook tags introduction](#91---playbook-tags-introduction)

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
^
After you have started the services, you can:

- start using the configured services
- or set up additional services
- or learn how to [upgrade services when new versions are released](#110---upgrading-services)
- or come say Hi in our [Matrix](https://matrix.org) support room - [#mash-playbook:devture.com](https://matrix.to/#/#mash-playbook:devture.com). You might learn something or get to help someone else new to hosting services with this playbook.
- or help make this playbook better by contributing (code, documentation, or [coffee/beer](https://liberapay.com/mother-of-all-self-hosting/donate))

## 10.0 - Importing an existing Postgres database

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
- after importing a large database, it's a good idea to run [an `ANALYZE` operation](https://www.postgresql.org/docs/current/sql-analyze.html) to make Postgres rebuild its database statistics and optimize its query planner. You can easily do this via the playbook by running `just run-tags run-postgres-vacuum -e postgres_vacuum_preset=analyze` (see [Vacuuming PostgreSQL](#122---vacuuming-postgresql) for more details).

# 11.0 - Upgrading services

This playbook not only installs various services for you, but can also upgrade them as new versions are made available.

To upgrade services:

- update your playbook directory (`git pull`), so you'd obtain everything new we've done

- take a look at [the changelog](../CHANGELOG.md) to see if there have been any backward-incompatible changes that you need to take care of

- download the upstream Ansible roles used by the playbook by running `just roles`

- re-run the [playbook setup](installing.md) and restart all services: `just setup-all`

**Note**: major version upgrades to the internal PostgreSQL database are not done automatically. To upgrade it, refer to the [upgrading PostgreSQL guide](services/postgres.md#upgrading-postgresql).

# 12.0 - Maintenance Postgres

This section shows you how to perform various maintenance tasks related to the Postgres database server used by various components of this playbook.

Table of contents:

- [Getting a database terminal](#getting-a-database-terminal), for when you wish to execute SQL queries

- [Vacuuming PostgreSQL](#vacuuming-postgresql), for when you wish to run a Postgres [VACUUM](https://www.postgresql.org/docs/current/sql-vacuum.html) (optimizing disk space)

- [Backing up PostgreSQL](#123---backing-up-postgresql), for when you wish to make a backup

- [Upgrading PostgreSQL](#124---upgrading-postgresql), for upgrading to new major versions of PostgreSQL. Such **manual upgrades are sometimes required**.

- [Tuning PostgreSQL](#125---tuning-postgresql) to make it run faster

## 12.1 - Getting a database terminal

You can use the `/mash/postgres/bin/cli` tool to get interactive terminal access ([psql](https://www.postgresql.org/docs/15/app-psql.html)) to the PostgreSQL server.

By default, this tool puts you in the `main` database, which contains nothing.

To see the available databases, run `\list` (or just `\l`).

To change to another database (for example `miniflux`), run `\connect miniflux` (or just `\c miniflux`).

You can then proceed to write queries. Example: `SELECT COUNT(*) FROM users;`

**Be careful**. Modifying the database directly (especially as services are running) is dangerous and may lead to irreversible database corruption.
).

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

Restoring a backup made this way can be done by [importing it](#10---importing-an-existing-postgres-database).

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

See chapter 10 + 12.

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

## 15.0 - Uninstalling

**Warning**: If you have some trouble with your installation, you can just [re-run the playbook](installing.md) and it will try to set things up again. **Uninstalling and then installing anew rarely solves anything**.

To uninstall, run these commands (most are meant to be executed on the server itself):

- ensure all services are stopped: `just stop` (if you can't get Ansible working to run this command, you can run `systemctl stop 'mash-*'` manually on the server)

- delete the systemd `.service` and `.timer` files (`rm -f /etc/systemd/system/mash-*.{service,timer}`) and reload systemd (`systemctl daemon-reload`)

- delete some cached Docker images (`docker system prune -a`) or just delete them all (`docker rmi $(docker images -aq)`)

- uninstall Docker itself, if necessary

- delete the `/mash` directory (`rm -rf /mash`)


## 16.0 - Services

### 16.1 - Supported Services

|              Name              |              Description              | Documentation |
| ------------------------------ | ------------------------------------- | ------------- |
| [AUX](https://github.com/mother-of-all-self-hosting/ansible-role-aux) | Auxiliary file/directory management on your server via Ansible | [Link](services/auxiliary.md) |
| [AdGuard Home](https://adguard.com/en/adguard-home/overview.html/) | A network-wide DNS software for blocking ads & tracking | [Link](services/adguard-home.md) |
| [APISIX Dashboard](https://apisix.apache.org/docs/dashboard/USER_GUIDE/) | A web UI for [APISIX Gateway](services/apisix-gateway.md) | [Link](services/apisix-dashboard.md) |
| [APISIX Gateway](https://apisix.apache.org/docs/apisix/getting-started/README/) | An API Gateway, Ingress Controller, etc | [Link](services/apisix-gateway.md) |
| [Appsmith](https://www.appsmith.com/) | Platform for building and deploying custom internal tools and applications without writing code | [Link](services/appsmith.md) |
| [Authelia](https://www.authelia.com/) | An open-source authentication and authorization server that can work as a companion to [common reverse proxies](https://www.authelia.com/overview/prologue/supported-proxies/) (like [Traefik](traefik.md) frequently used by this playbook) | [Link](services/authelia.md) |
| [authentik](https://goauthentik.io/) | An open-source Identity Provider focused on flexibility and versatility. | [Link](services/authentik.md) |
| [borgbackup](https://www.borgbackup.org/) (via [borgmatic](https://torsion.org/borgmatic/)) | A deduplicating backup program with optional compression and encryption| [Link](services/backup-borg.md) |
| [Calibre-Web](https://github.com/janeczku/calibre-web) | Web app for browsing, reading and downloading eBooks stored in a [Calibre](https://calibre-ebook.com/) database | [Link](services/calibre-web.md) |
| [Changedetection.io](https://github.com/dgtlmoon/changedetection.io) | A simple website change detection and restock monitoring solution. | [Link](services/changedetection.md) |
| [ClickHouse](https://clickhouse.com/) | An open-source column-oriented DBMS for online analytical processing (OLAP) that allows users to generate analytical reports using SQL queries in real-time. | [Link](services/clickhouse.md) |
| [Collabora Online](https://www.collaboraoffice.com/) | Your Private Office Suite In The Cloud | [Link](services/collabora-online.md) |
| [CouchDB](https://couchdb.apache.org/) | An open-source document-oriented NoSQL database, implemented in Erlang. | [Link](services/couchdb.md) |
| [Docker](https://www.docker.com/) | Open-source software for deploying containerized applications | [Link](services/docker.md) |
| [Docker Registry](https://docs.docker.com/registry/) | A container image distribution registry | [Link](services/docker-registry.md) |
| [Docker Registry Browser](https://github.com/klausmeyer/docker-registry-browser) | Web Interface for the Docker Registry HTTP API V2 written in Ruby on Rails | [Link](services/docker-registry-browser.md) |
| [Docker Registry Proxy](https://gitlab.com/etke.cc/docker-registry-proxy/) | Pass-through docker registry (distribution) proxy with metadata caching, docker-compatible errors, prometheus metrics, etc. | [Link](services/docker-registry-proxy.md) |
| [Docker Registry Purger](https://github.com/devture/docker-registry-purger) | A small tool used for purging a private Docker Registry's old tags | [Link](services/docker-registry-purger.md) |
| [Echo IP](https://github.com/mpolden/echoip) | A simple service for looking up your IP address | [Link](services/echoip.md) |
| [Endlessh-go](https://github.com/shizunge/endlessh-go) | A golang implementation of endlessh, a ssh trapit | [Link](services/endlessh.md) |
| [etcd](https://etcd.io/) | A distributed, reliable key-value store for the most critical data of a distributed system | [Link](services/etcd.md) |
| [exim-relay](https://github.com/devture/exim-relay) | A lightweight [Exim](https://www.exim.org/) SMTP mail relay server | [Link](services/exim-relay.md) |
| [Focalboard](https://www.focalboard.com/) | An open source, self-hosted alternative to [Trello](https://trello.com/), [Notion](https://www.notion.so/), and [Asana](https://asana.com/). | [Link](services/focalboard.md) |
| [FreshRSS](https://freshrss.org/) | RSS and Atom feed aggregator. | [Link](services/freshrss.md) |
| [Firezone](https://www.firezone.dev/) | A self-hosted VPN server (based on [WireGuard](https://www.wireguard.com/)) with a Web UI | [Link](services/firezone.md) |
| [Funkwhale](https://funkwhale.audio/) | Listen and share music with a selfhosted streaming server.| [Link](services/funkwhale.md) |
| [Gitea](https://gitea.io/) | A painless self-hosted [Git](https://git-scm.com/) service. | [Link](services/gitea.md) |
| [GoToSocial](https://gotosocial.org/) | A self-hosted [ActivityPub](https://activitypub.rocks/) social network server | [Link](services/gotosocial.md) |
| [Grafana](https://grafana.com/) | An open and composable observability and data visualization platform, often used with [Prometheus](services/prometheus.md) | [Link](services/grafana.md) |
| [Grafana Loki](https://grafana.com/docs/loki/latest/) | Open-source log aggregation system that helps collect, store, and analyze logs in a scalable and efficient manner | [Link](services/grafana-loki.md) |
| [Healthchecks](https://healthchecks.io/) | A simple and Effective Cron Job Monitoring solution | [Link](services/healthchecks.md) |
| [Hubsite](https://github.com/moan0s/hubsite) | A simple, static site that shows an overview of the available services | [Link](services/hubsite.md) |
| [ILMO](https://github.com/moan0s/ILMO2) | An open source library management tool. | [Link](services/ilmo.md) |
| [Infisical](https://infisical.com/) | An open-source end-to-end encrypted platform for securely managing secrets and configs across your team, devices, and infrastructure. | [Link](services/infisical.md) |
| [InfluxDB](https://www.influxdata.com/) | A self-hosted time-series database. | [Link](services/influxdb.md) |
| [Jitsi](https://jitsi.org/) | A fully encrypted, 100% Open Source video conferencing solution | [Link](services/jitsi.md) |
| [Keycloak](https://www.keycloak.org/) | An open source identity and access management solution. | [Link](services/keycloak.md) |
| [KeyDB](https://docs.keydb.dev/) | An in-memory data store used by millions of developers as a database, cache, streaming engine, and message broker. | [Link](services/keydb.md) |
| [Lago](https://www.getlago.com/) | Open-source metering and usage-based billing | [Link](services/lago.md) |
| [languageTool](https://languagetool.org/) | An open source online grammar, style and spell checker | [Link](services/languagetool.md) |
| [linkding](https://github.com/sissbruecker/linkding/) | Bookmark manager designed to be minimal and fast. | [Link](services/linkding.md) |
| [MariaDB](https://mariadb.org/) | A powerful, open source object-relational database system | [Link](services/mariadb.md) |
| [Matrix Rooms Search API](https://gitlab.com/etke.cc/mrs/api) | A fully-featured, standalone, matrix rooms search service. | [Link](services/mrs.md) |
| [MongoDB](https://www.mongodb.com/) | A source-available cross-platform document-oriented (NoSQL) database program. | [Link](services/mongodb.md) |
| [Mosquitto](https://mosquitto.org/) | An open-source MQTT broker | [Link](services/mosquitto.md) |
| [Miniflux](https://miniflux.app/) | Minimalist and opinionated feed reader. | [Link](services/miniflux.md) |
| [Mobilizon](https://joinmobilizon.org/en/) | An ActivityPub/Fediverse server to create and share events. | [Link](services/mobilizon.md) |
| [n8n](https://n8n.io/) | Workflow automation for technical people. | [Link](services/n8n.md) |
| [Navidrome](https://www.navidrome.org/) | [Subsonic-API](http://www.subsonic.org/pages/api.jsp) compatible music server | [Link](services/navidrome.md)
| [n.eko](https://neko.m1k1o.net/) | A self-hosted virtual browser or even desktop environment | [Link](services/neko.md) |
| [NetBox](https://docs.netbox.dev/en/stable/) | Web application that provides [IP address management (IPAM)](https://en.wikipedia.org/wiki/IP_address_management) and [data center infrastructure management (DCIM)](https://en.wikipedia.org/wiki/Data_center_management#Data_center_infrastructure_management) functionality | [Link](services/netbox.md) |
| [Nextcloud](https://nextcloud.com/) | The most popular self-hosted collaboration solution for tens of millions of users at thousands of organizations across the globe. | [Link](services/nextcloud.md) |
| [Outline](https://www.getoutline.com/) | An open-source knowledge base for growing teams. | [Link](services/outline.md) |
| [OAuth2-Proxy](https://oauth2-proxy.github.io/oauth2-proxy/) | A reverse proxy and static file server that provides authentication using OpenID Connect Providers (Google, GitHub, [Keycloak](services/keycloak.md), and others) to SSO-protect services which do not support SSO natively. | [Link](services/oauth2-proxy.md) |
| [Owncast](https://owncast.online/) | Owncast is a free and open source live video and web chat server for use with existing popular broadcasting software. | [Link](services/owncast.md) |
| [OxiTraffic](https://codeberg.org/mo8it/oxitraffic) | [OxiTraffic](https://codeberg.org/mo8it/oxitraffic) is a self-hosted, simple and privacy respecting website traffic tracker. | [Link](services/oxitraffic.md) |
| [Paperless-ngx](https://paperless-ngx.com) | [Paperless-ngx](https://paperless-ngx.com) is a community-supported open-source document management system that transforms your physical documents into a searchable online archive so you can keep, well, less paper. | [Link](services/paperless-ngx.md) |
| [PeerTube](https://joinpeertube.org/) | A tool for sharing online videos | [Link](services/peertube.md) |
| [Plausible Analytics](https://plausible.io/) | Intuitive, lightweight and open source web analytics | [Link](services/plausible.md) |
| [Postgis](https://postgis.net/) | A spatial database extender for PostgreSQL object-relational database | [Link](services/postgis.md) |
| [Postgres](https://www.postgresql.org) | A powerful, open source object-relational database system | [Link](services/postgres.md) |
| [Postgres Backup](https://github.com/prodrigestivill/docker-postgres-backup-local) | A solution for backing up PostgresSQL to local filesystem with periodic backups. | [Link](services/postgres-backup.md) |
| [Prometheus](https://prometheus.io/) | A metrics collection and alerting monitoring solution | [Link](services/prometheus.md) |
| [Prometheus Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) | Blackbox probing of HTTP/HTTPS/DNS/TCP/ICMP and gRPC endpoints | [Link](services/prometheus-blackbox-exporter.md) |
| [Prometheus SSH Exporter](https://github.com/treydock/ssh_exporter) | SSH probes | [Link](services/prometheus-ssh-exporter.md) |
| [Prometheus Node Exporter](https://github.com/prometheus/node_exporter) | Exporter for machine metrics | [Link](services/prometheus-node-exporter.md) |
| [Prometheus Postgres Exporter](https://github.com/prometheus-community/postgres_exporter) | A PostgreSQL metric exporter for Prometheus | [Link](services/prometheus-postgres-exporter.md) |
| [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/) | An agent which ships the contents of local logs to a private [Grafana Loki](services/grafana-loki.md) instance | [Link](services/promtail.md) |
| [Radicale](https://radicale.org/) | A Free and Open-Source CalDAV and CardDAV Server (solution for hosting contacts and calendars) | [Link](services/radicale.md) |
| [Redmine](https://redmine.org/) | A flexible project management web application. | [Link](services/redmine.md) |
| [Redis](https://redis.io/) | An in-memory data store used by millions of developers as a database, cache, streaming engine, and message broker. | [Link](services/redis.md) |
| [Roundcube](https://roundcube.net/) | A browser-based multilingual IMAP client with an application-like user interface | [Link](services/roundcube.md) |
| [rumqttd](https://github.com/bytebeamio/rumqtt) | A high performance, embeddable [MQTT](https://en.wikipedia.org/wiki/MQTT) broker | [Link](services/rumqttd.md) |
| [Ansible Semaphore](https://www.ansible-semaphore.com/) | A responsive web UI for running Ansible playbooks  | [Link](services/semaphore.md) |
| [Soft Serve](https://github.com/charmbracelet/soft-serve) | A tasty, self-hostable [Git](https://git-scm.com/) server for the command line | [Link](services/soft-serve.md) |
| [Syncthing](https://syncthing.net/) | A continuous file synchronization program which synchronizes files between two or more computers in real time | [Link](services/syncthing.md) |
| [Tandoor](https://docs.tandoor.dev/) | The recipe manager that allows you to manage your ever growing collection of digital recipes.| [Link](services/tandoor.md)
| [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) | An open source server agent to help you collect metrics from your stacks, sensors, and systems. | [Link](services/telegraf.md) |
| [Traefik](https://doc.traefik.io/traefik/) | A container-aware reverse-proxy server | [Link](services/traefik.md) |
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | A lightweight unofficial and compatible implementation of the [Bitwarden](https://bitwarden.com/) password manager | [Link](services/vaultwarden.md) |
| [Uptime-kuma](https://uptime.kuma.pet/) | A fancy self-hosted monitoring tool | [Link](services/uptime-kuma.md) |
| [Wetty](https://github.com/butlerx/wetty) | An SSH terminal over HTTP/HTTPS | [Link](services/wetty.md) |
| [WireGuard Easy](https://github.com/wg-easy/wg-easy) | The easiest way to run [WireGuard](https://www.wireguard.com/) VPN + Web-based Admin UI. | [Link](services/wg-easy.md) |
| [Forgejo](https://forgejo.org/) | An alternative fork of Gitea. Easy and painless self-hosted git server. | [Link](services/forgejo.md) |
| [Woodpecker CI](https://woodpecker-ci.org/) | A simple Continuous Integration (CI) engine with great extensibility. | [Link](services/woodpecker-ci.md) |
| [WordPress](https://wordpress.org/) | A widley used open source web content management system | [Link](services/wordpress.md) |
| [Writefreely](https://writefreely.org) | A clean, minimalist publishing platform made for writers with optional federation via ActivityPub. | [Link](services/writefreely.md) |
| System-related | A collection of various system-related components | [Link](services/system.md) |


### 16.2 - Related playbooks

- [matrix-docker-ansible-deploy](https://github.com/spantaleev/matrix-docker-ansible-deploy) - for deploying a fully-featured [Matrix](https://matrix.org) homeserver. This playbook will remain independent, because the Matrix ecosystem is incredibly large - lots of bots, bridges and other pieces of software. It deserves its own dedicated playbook.
