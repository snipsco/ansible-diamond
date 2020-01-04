# Ansible Role: Diamond #

Ansible role that installs the [Diamond metric collector daemon](https://github.com/python-diamond/Diamond) on RedHat/CentOS Debian/Ubuntu.

This is a fork of https://github.com/snipsco/ansible-diamond, but with saner defaults.

# Requirements #

This role is currently meant to be used against Graphite (via the carbon protocol) or any other metric trending software that supports statsd (this also includes Graphite if running statsd daemon)

# Role variables #

## Default behaviour ##

The default is to write enable the archive handler.

```
diamond_handlers:
  - name: ArchiveHandler
    path: diamond.handler.archive.ArchiveHandler
    config:
      log_file: /var/log/diamond/archive.log

```

To provide your own collector definitions, define the hash `diamond_collector_extra_defs` either in your invoking playbook, or in `defaults/main.yml` inside the role. See below for examples.

## Overridable variables ##

Setting this to true will build the latest master branch of BrightcoveOS/Diamond, see below.
If set to false, currently supports only **Debian/Ubuntu** platforms and expects a prebuilt deb package under files/diamond_<version>_all.deb. See below on specifying the version.

`diamond_git_repo_url: https://github.com/python-diamond/Diamond`

Change this if you'd like to a fork of the Diamond repository; used with `build_from_source: true`

    graphitehandler:
      enable: False
      host: none
      port: 2003
    statsdhandler:
      enable: True
      host: tbd
      port: 8125


As mentioned above, you **must** specify enable: True for one of the above, along with the correct IP / port

`diamond_git_repo_version: "e891f8e1eef6067407f2bb0ddff4973893c33c94"`

When building from source, specify the commit id or a tag.
The default tag refers to v4.0.

`diamond_version: "3.5.8"`

Specify the version mentioned in the prebuilt deb package filename; valid only when `build_from_source: false`

`diamond_conf_basepath: "/etc/diamond"`

Specifies the directory for diamond's config file. Just leave it to default if you don't have a specific reason to change it.

`collector_conf_path: "{{diamond_conf_basepath}}/collectors"`

Directory where collector definitions go

`diamond_handlers_path: "/usr/share/pyshared/diamond/handler/"`

Directory where handlers will end up; this is valid for Debian.
For RedHat typically it is `/usr/lib/python2.6/site-packages/diamond/handler/`

In the next version it will be defined automatically based on platform family.

## Generating collector definitions ##

This role is capable of automatically generating collector definitions based on default values specified in vars/collector_definitions.yml

To alter the default behaviour specify the hash `diamond_collector_extra_defs` in your playbook vars: or in defaults/main.yml.

### Example1: enabling Zookeeper collector -- disabled by default ###

First have a look at vars/collector_definitions.yml
At the bottom of the file we can see that the Zookeeper collector is disabled:

    ZookeeperCollector:
        enabled: False


To enable it, in our playbook, in the vars: section we define:

    diamond_collector_extra_defs:
      ZookeeperCollector:
        enabled: True

### Example2: also enable RabbitMQCollector -- disabled by default ###

Again have a look at vars/collector_definitions.yml to see how this is defined and append it to your existing definitions in vars: section of your playbook:

    diamond_collector_extra_defs:
      ZookeeperCollector:
        enabled: True
      RabbitMQCollector:
        enabled: True

# Dependencies #

None

# Example playbook #

See the included playbook install_debian.yml

    ---
    - hosts: all
      sudo: true
      vars:
        build_from_source: true
        statsdhandler:
          enable: True
          host: 127.0.0.1
          port: 8125
        diamond_collector_extra_defs:
          ZookeeperCollector:
            enabled: False
      roles:
        - liappis.diamond

# testing wtih molecule

```
tox -e py36-ansible28 -- molecule test -s default

tox -e py36-ansible29 -- molecule test -s default
```

# Testing with Vagrant #

There are a number of vagrant machines defined.
If you have just pulled this role from github make sure it is under a directory called roles.
If you've installed it via `ansible-galaxy install liappis.diamond` that should have been taken care for you.

After executing any of the following you should get a VM running diamond and shipping metrics to localhost port 8125 via the statsd protocol.

## For virtualbox ##

* CentOS65: `vagrant up centos65virtualbox`
* Ubunty Trusty: `vagrant up trustyvirtualbox`
* Debian Wheezy: `vagrant up debianwheezyvirtualbox`

## For libvirt ##

* CentOS65: `vagrant up centos65libvirt`
* CentOS7: `vagrant up centos7libvirt`
* Ubuntu Trusty: `vagrant up trusty64libvirt`
* Debian Wheezy: `vagrant up debian75libvirt`

# License #

Apache

# Author information #

This role was created in 2014 by [Dimitrios Liappis](mailto:liappis@pythian.com)
