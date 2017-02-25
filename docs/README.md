


## Table of content
[TOC]

## Installation


To run easy-vagrant, you must have:

- vagrant
  - vagrant > 1.8
  - plugins:
    - ansible
    - vagrant-libvirt
- Providers:
  - libvirt: libvirtd
  - virtualbox: virtualbox
- Provisionners:
  - ansible: ansible (any versions)


## Usage

### Boxes
At this stage, easy-vagrant comes with a preset


#### Provisionners

Structure:
```
provsionneurs:
  $provisionner_id:
    [priority: <integer O to 100, default to 0>]
    type: Must be one of Vagrant supported provsionner, such as 'shell' or 'ansible' or your_provsionner
    params: A list of
      $param1:
      $param1:
      $paramN:

```

## Internal

The vagrant definition is should be exclusively done with yaml. You should not have to modify the ```Vagrantfile``` unless you want to debug or add new features. The configuration is split in 3 main sources:

1. Default config: This is the hardcoded configuration. It comes with some handy defaults, such as provisionners and preset boxes for vagrant providers. You should never have to modify this configuration. Source: ```Vagrantfile```
2. Developper config: This is the configuration made by the developper of this Vagrant setup. It should mainly define the instance settings, and some provisionners. Source file: ```conf/vagrant.yml```
0. User config: This config is optional and allow the final user to ovverrides some settings according to its local computer. This configuration is outside of the git repository. Source: ```local.yml```

The default configuration inherits configuration to developper configuraiton, and user configuration inherits all configurations.

### Internal vs exposed configuration

In way to make a friendly editable yaml configuration, there were the need of exposing an easy yaml structure for the end user, and a more computer readable configuration for vagrant. The process is the following:

0. The user defines a yaml confikguration with 3 primary keys:
  1. ```settings```:
  2. ```instances```: This is literraly
  3. ```provisionners```:


### Loading order and inheritance

There are two special values:

- ```nil```: Mainly used to not override a inherited value. Sort of comments finally.
- ```unset```: This remove the key. You may use this when you do not want to inherits some settings from default or developper configuration. You may not need to use this, but always useful to have it in case of. Be aware: easy-vagrant expect to have keys always defined, even if they have an empty value. Dont try to unset top level keys unless you want to break easy-vagrant.


```sequence
None-->Default: provides
Default-->Developper: provides
Developper-->User: provides

Developper-->Default: nil
User-->Developper: nil
Developper->None: unset
User->None: unset

```

### Expand and override defaults settings
It is quite easy, just replace the directive with your own content. If it is a hash, it will be merged with the inherited value. Example to expand available boxes:
```
settings:
  boxes:
    mycustomboxname: mycustombox_url/mycustombox_atlas_id
```
To overrides, there is a trick, we need to declare two times the children, the firest one with the unset value, and another time with the new content:
```
settings:
  boxes: unset
  boxes:
    mycustomboxname: mycustombox_url/mycustombox_atlas_id
```
