# [Mini-dinstall](https://manpages.debian.org/stretch/mini-dinstall/mini-dinstall.1.en.html)

[![Build Status](https://travis-ci.com/maxlareo/ansible-mini-dinstall.svg?branch=master)](https://travis-ci.com/maxlareo/ansible-mini-dinstall)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-mini--dinstall-blue.svg)](https://galaxy.ansible.com/maxlareo/mini_dinstall)

Install and configure mini-dinstall APT repository tool.

## Requirements

+ Debian based host

## Role Variables

Variable                    | Description                                         | Type   | Default
--------                    | -----------                                         | ----   | -------
`minidinstall_user`         | System user                                         | String | `"mini-dinstall"`
`minidinstall_user_home`    | User home directory path                            | String | `"/data/mini-dinstall"`
`minidinstall_basedir`      | Apt repository base directory                       | String | `"{{ minidinstall_user_home }}/debpkg"`
`minidinstall_conf`         | Configuration file path                             | String | `/etc/mini-dinstall/mini-dinstall.conf`
`minidinstall_sshkeys`      | SSH authorized keys list to allow repository access | List   | `[]`
`minidinstall_config`       | mini-dinstall "[DEFAULT]" configuration section     | Dict   | see [config](#config)
`minidinstall_repositories` | mini-dinstall repositories configuration section    | Dict   | see [repositories](#repositories)

### Role Mechanism

#### config

Set `[DEFAULT]` configuration options through a Dict variable.
See `man mini-dinstall` for option list and explanation.

The YAML `key: value` will correspond to `key = vale` into the
mini-dinstall.conf file.

Default:
```
minidinstall_config:
  archivedir: "{{ minidinstall_basedir }}"
  archive_style: flat
  generate_release: 1
  incoming_permissions: "0750"
```

#### repositories

Set different repositories configuration with this Dict variable.

The first key will be the repository name, then all the sub key/value pair will
correspond to the specific options for that repository.

Default:
```
minidinstall_repositories:
  unstable:
    release_codename: sid
```

## Dependencies

None.

## Example Playbook

Playbook to use this role and allow charlie to push package:

```yaml
- hosts: aptserver
  roles:
    - mini-dinstall
  vars:
    minidinstall_sshkeys:
      - https://github.com/charlie.keys
      - "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
```

## Usage

### Signing the Release file

The 2 main options are:

1. to use a  GPG key created for the occasion, store that key (including the
 private part) on the apt server and use the mini-dinstall contrib script
 sign-release.sh by adding something like the following to your
 mini-dinstall.conf:
 _release_signscript = ~/bin/sign-release.sh_

2. to use some script to sign the Release file remotely and send the resulting
 deatched signature back to the apt server just after the mini-dinstall pulse;
 that way you can sign with your own private key.
 example: [zack sign-remote script](https://upsilon.cc/~zack/blog/posts/2009/04/howto:_uploading_to_people.d.o_using_dput/sign-remote)

### Importing the pgp key of the apt repo

To fetch the signed apt repositories Release, the public key have to be added
to the apt-key store using `apt-key add`.

### Add the repo urls in your apt sources.list

The mini-dinstall syntax to use the repo, for flat architecture are
`deb http://fqdn.repo/ repository/`.

example: `deb http://fqdn.repo/ unstable/`

### Dput

Example of ~/.dput.cf file:

```
[unstable]
fqdn = fqdn.repo
login = mini-dinstall
incoming = /data/mini-dinstall/debpkg/mini-dinstall/incoming
method = scp
run_dinstall = 0
post_upload_command = ssh -l mini-dinstall fqdn.repo \
  "mini-dinstall -c /etc/mini-dinstall/mini-dinstall.conf -b" \
  && sign-remote mini-dinstall@fqdn.repo:~/debpkg/unstable/Release
```

## Todo

+ Add HTTP config management
+ Add GPG host key management for use case [1](#signing-the-release-file).
