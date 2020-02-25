<p><img src="https://code.benco.io/icon-collection/logos/ansible.svg" alt="ansible logo" title="ansible" align="left" height="60" /></p>
<p><img src="https://previews.123rf.com/images/viktorijareut/viktorijareut1710/viktorijareut171000267/90109811-zcash-crypto-currency-block-chain-flat-logo.jpg" alt="zcash logo" title="zcash" align="right" height="80" /></p>

Ansible Role :lock_with_ink_pen: :link: Zcashd
=========
[![Galaxy Role](https://img.shields.io/ansible/role/46798.svg)](https://galaxy.ansible.com/0x0I/zcashd)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/0x0I/ansible-role-zcashd?color=yellow)
[![Downloads](https://img.shields.io/ansible/role/d/46798.svg?color=lightgrey)](https://galaxy.ansible.com/0x0I/zcashd)
[![Build Status](https://travis-ci.org/0x0I/ansible-role-zcashd.svg?branch=master)](https://travis-ci.org/0x0I/ansible-role-zcashd)
[![License: MIT](https://img.shields.io/badge/License-MIT-blueviolet.svg)](https://opensource.org/licenses/MIT)

**Table of Contents**
  - [Supported Platforms](#supported-platforms)
  - [Requirements](#requirements)
  - [Role Variables](#role-variables)
      - [Install](#install)
      - [Config](#config)
      - [Launch](#launch)
      - [Uninstall](#uninstall)
  - [Dependencies](#dependencies)
  - [Example Playbook](#example-playbook)
  - [License](#license)
  - [Author Information](#author-information)

Ansible role that installs and configures Zcashd: a client for the Zcash zero-knowledge privacy blockchain/protocol.

##### Supported Platforms:
```
* Redhat(CentOS/Fedora)
* Ubuntu
* Debian
```

Requirements
------------
Requires the `unzip/gtar` utility to be installed on the target host. See ansible `unarchive` module [notes](https://docs.ansible.com/ansible/latest/modules/unarchive_module.html#notes) for details.

Role Variables
--------------
Variables are available and organized according to the following software & machine provisioning stages:
* _install_
* _config_
* _launch_
* _uninstall_

#### Install

`zcashd`can be installed using compressed archives (`.tar`, `.zip`) for debian-based installations, downloaded and extracted from various sources, or built from source.

_The following variables can be customized to control various aspects of this installation process, ranging from software version and source location of binaries to the installation directory where they are stored:_

`zcashd_user: <service-user-name>` (**default**: *zcashd*)
- dedicated service user and group used by `zecminer` for privilege separation (see [here](https://www.beyondtrust.com/blog/entry/how-separation-privilege-improves-security) for details)

`install_type: <archive>` (**default**: archive)
- **archive**: currently compatible with **tar** formats, installation of **Zcashd** via compressed archives results in the direct download of its component binaries, consisting of the `zcashd` blockchain service and cli software.

  **note:** archived installation binaries can be obtained from the official [releases](https://github.com/zcash/zcash/releases) site or those generated from development/custom sources.

`install_dir: </path/to/installation/dir>` (**default**: `/opt/zcashd`)
- path on target host where the `zcashd` binaries should be extracted to.

`archive_url: <path-or-url-to-archive>` (**default**: see `defaults/main.yml`)
- address of a compressed **tar or zip** archive containing `zcashd` binaries. This method technically supports installation of any available version of `zcashd`. Links to official versions can be found [here](https://github.com/zcash/zcash/releases).

`archive_options: <untar-or-unzip-options>` (**default**: `[]`)
- list of additional unarchival arguments to pass to either the `tar` or `unzip` binary at runtime for customizing how the archive is extracted to the designated installation directory. See [man tar](https://linux.die.net/man/1/tar) and [man unzip](https://linux.die.net/man/1/unzip) for available options to specify, respectively.

#### Config

**Zecminer** supports specification of various options controlling aspects of the Zcash miner's behavior and operational profile. Each configuration can be expressed via either the tool's command-line interface or within in an INI-sytle configuration file. The following details the facilities provided by this role to manage the contents of the aforementioned configuration file.

In typical `INI` or `TOML` fashion, individual sets of configurations consist of definitions representing config sections and associated setting key-value pairs. These sections and settings are made up of common miner options and custom Zcash server pool properties to access.

Reference [here](https://zec.nanopool.org/) for a list of server pool connection settings and [here](https://gist.github.com/0x0I/8a02072f7a2729bd8a0ab626d89c16d2) for an example config.

`config_dir: </path/to/configuration/dir>` (**default**: `{{ install_dir }}`)
- path on target host where the `zecminer` configuration file should be rendered

Each of these configurations can be expressed using the `zecminer_configs` hash, which contains a list of various `zecminer` configuration options (hash) objects organized according to the following:
* common - miner configuration options common to all server connections
* server - custom server connection and operational configuration options to the specified server

Each `zecminer_configs` entry is a hash representing the equivalent of a configuration section as expected to be expressed by the `zecminer` server. As previously described, these hashes are structured by config section and associated key-value setting pairs.

`[zecminer_configs: <entry>:] name: <common|server>` (**default**: *required*)
- name of the configuration section to render

`[zecminer_configs: <entry>:] settings: <YAML>` (**default**: )
- specifies parameters that manage various aspects of the Zcash miner's operations

###### Example

 ```yaml
  zecminer_configs:
    - name: common
      settings:
        log: 0
        logfile: /var/log/miner.log
        api: 127.0.0.1:42000
    - name: server
      settings:
        server: zec-us-east1.nanopool.org
        port: 6666
        user: user1-key
        pass: user1-password
  ```

#### Launch

This role supports launching `zcashd` utilizing the [systemd](https://www.freedesktop.org/wiki/Software/systemd/) service management tool, which manages the service as a background process or daemon subject to the configuration and execution potential provided by its underlying management framework.

_The following variables can be customized to manage the service's **systemd** [Service] unit definition and execution profile/policy:_

`extra_run_args: <zecminer-cli-options>` (**default**: `[]`)
- list of `zcashd` commandline arguments to pass to the binary at runtime for customizing launch

Supporting full expression of `zcashd`'s [cli](https://gist.github.com/0x0I/8a57be009fcdb3a006262309aadd741c) and, consequently the full set of configuration options as referenced and described above, this variable enables the launch to be customized according to the user's exact specification.

`custom_unit_properties: <hash-of-systemd-service-settings>` (**default**: `[]`)
- hash of settings used to customize the `[Service]` unit configuration and execution environment of the `zcashd` **systemd** service.

#### Uninstall

Support for uninstalling and removing artifacts necessary for provisioning allows for users/operators to return a target host to its configured state prior to application of this role. This can be useful for recycling nodes and perhaps providing more graceful/managed transitions between tooling upgrades.

_The following variable(s) can be customized to manage this uninstall process:_

`perform_uninstall: <true | false>` (**default**: `false`)
- whether to uninstall and remove all artifacts and remnants of this `zcashd` installation on a target host (**see**: `handlers/main.yml` for details)

Dependencies
------------

- 0x0i.systemd

Example Playbook
----------------
default example:
```
- hosts: all
  roles:
  - role: 0x0I.zcashd
```

License
-------

MIT

Author Information
------------------

This role was created in 2020 by O1.IO.
