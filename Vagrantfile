# -*- mode: ruby -*-
# vim: set ft=ruby  tabstop=2 softtabstop=0 expandtab shiftwidth=2 smarttab :

# Date: 2016/02/28
# Version 0.5
# License: AGPLv3
# Author: mrjk
# Source: https://github.com/mrjk/easy-vagrant
# Contact: bXJqay43OCBbYXRdIGdtYWlsIFtkb3RdIGNvbQ==



########################################
# Simplified vagrant architecture spwaner
########################################

# Please edit the "conf/vagrant.yml" file to build your own architecture.
# Do not edit what's bellow unless you know what you do (and Ruby)


Vagrant.require_version ">= 1.7.0"
VAGRANTFILE_API_VERSION = "2"
require 'yaml'


# Internal variables
show_banner = true
dry_run = false
show_config_merged = false
show_config_parsed = false
show_config_instances = true

# Determine project root folder
vagrant_root = File.dirname(__FILE__) + '/'

########################################
# Display banner
########################################

if show_banner
  puts "INFO: ===================================================="
  puts "INFO: ==             Welcome on Easy-vagrant            =="
  puts "INFO: ==                    by mrjk                     =="
  puts "INFO: ===================================================="
end


########################################
# Default configuration
########################################

# This is the default structure. All of this can
# be overriden into userland. Do not edit this structure
# unless you know what you do.

conf_default = {
  'settings' => {
    'defaults' => {
      'flavor' => 'tiny',
      'box' => 'centos7',
      'provider' => 'libvirt',
      'prefix' => 'prefix-',
      'sufix' => '-sufix',
      'provisionners' => {},
    },
    'flavors' => {
      'nano' => {
        'cpu' => '1',
        'memory' => '128',
        'disk' => '5'
      },
      'micro' => {
        'cpu' => '2',
        'memory' => '512',
        'disk' => '10'
      },
      'small' => {
        'cpu' => '2',
        'memory' => '1024',
        'disk' => '10'
      },
      'medium' => {
        'cpu' => '2',
        'memory' => '2048',
        'disk' => '20'
      },
      'large' => {
        'cpu' => '3',
        'memory' => '4096',
        'disk' => '30'
      },
      'xlarge' => {
        'cpu' => '4',
        'memory' => '8192',
        'disk' => '40'
      }
    },
    'boxes' => {
      'debian9' => 'debian/stretch64',
      'debian8' => 'debian/jessie64',
      'debian7' => 'wholebits/debian7-64',
      'ubuntu1604' => 'wholebits/ubuntu16.10-64',
      'ubuntu1404' => 'wholebits/ubuntu14.04-64',
      'centos7' => 'centos/7',
      'centos6' => 'centos/6',
      'archlinux' => 'archlinux/archlinux',
    },
    'providers' => {
      'libvirt' => {
        'settings' => {
          'driver' => 'nil',
          'host' => 'nil',
          'connect_via_ssh' => 'nil',
          'username' => 'nil',
          'password' => 'nil',
          'id_ssh_key_file' => '~/.ssh/id_rsa',
          'socket' => 'nil',
          'uri' => 'nil'
        },
      },
      'virtualbox' => {
        'instances' => {
          'linked_clones' => 'true',
          'execution_cap' => '50',
          'page_fusion' => 'on',
          'cpuhotplug' => 'on',
          'pae' => 'on',
          'largepages' => 'on',
          'guestmemoryballoon' => '128',
        },
        'boxes' => {
          'debian8' => 'minimal/jessie64',
          'debian7' => 'minimal/wheezy64',
          'ubuntu1604' => 'minimal/xenial64',
          'ubuntu1404' => 'minimal/trusty64',
          'centos7' => 'minimal/centos7',
          'centos6' => 'minimal/centos6',
        }
      },
      'docker' => {
        'settings' => {
          'force_host_vm' => 'false',
          'pull' => 'false',
          'remains_running' => 'true',
          'stop_timeout' => '30',
        },
        'boxes' => {
          'scratch' => 'scratch',
          'alpine' => 'alpine:latest',
          'debian8' => 'debian:8',
          'debian7' => 'debian:7',
          'ubuntu1604' => 'ubuntu:xenial',
          'ubuntu1404' => 'ubuntu:trusty',
          'ubuntu1204' => 'ubuntu:precise',
          'centos7' => 'centos:7',
          'centos6' => 'centos:6',
          'centos5' => 'centos:5',
          'archlinux' => 'archlinuxjp/archlinux'
        }
      }
    }
  },
  'provisionners' => {
    'test_shell_script' => {
      'type' => 'shell',
      'description' => 'This execute a shell script',
      'params' => {
        'path' => vagrant_root + 'conf/scripts/test_script.sh',
        'args' => [
          'Provisionning',
          'shell_script',
          'worked as expected as non root.',
        ],
        'privileged' => false,
      },
    },
    'install_python_for_ansible' => {
      'type' => 'shell',
      'priority' => '90',
      'description' => 'This will install Python on the target',
      'params' => {
        'path' => vagrant_root + 'conf/scripts/install_python.sh',
        'privileged' => true,
      },
    },
    'test_shell_cli' => {
      'type' => 'shell',
      'description' => 'This execute a shell command',
      'params' => {
        'inline' => 'echo $1 $2 $3 > /tmp/vagrant_provisionning_cli',
        'args' => [
          'Provisionning',
          'shell_cli',
          'worked as expected as root.',
        ],
        'privileged' => true,
      },
    },
    'ansible_test' => {
      'type' => 'ansible',
      'description' => 'Test Ansible run correctly',
      'params' => {
        'playbook' => vagrant_root + 'conf/ansible/test.yml',
      },
    },
    'ansible_deploy_key' => {
      'type' => 'ansible',
      'description' => 'Deploy an SSH pubkey to specific user',
      'params' => {
        'playbook' => vagrant_root + 'conf/ansible/deploy_key.yml',
        'extra_vars' => {
          'target_user' => 'root',
        },
      },
    },
  },
}


########################################
# Define functions
########################################

# This function checks if a subelement of a
# hash exists and it is not set to the 'null'
# string. Return a boolean.

def attribute_is_defined(object, key)

  if object.class ==  Hash and object.key?(key)
    value = object[key].to_s

    #puts "DEBUG: object" , object.class
    #puts "DEBUG: key" , value
    #puts 

    if value.casecmp("nil") != 0
      return true
    end
  end
  return false
end


# This function helps to retrieve the merge strategy key. 
# Returns a Hash with merge settings.
def get_merge_params(hash, key)

  merge_string = '_'

  if hash.has_key? key + merge_string

    if hash[key + merge_string].class == String
      out = {}
      out['action'] = hash[key + merge_string]
      out['options'] = nil
      out
    elsif hash[key + merge_string].class == Hash
      out = hash[key + merge_string]
      out
    else
      print 'ERROR: Action must be a String of a Hash and not ', hash[key + merge_string].class, "\n"
    end

  else
    out = {}
    out['action'] = 'union'
    out['options'] = {}
    out
  end

end


# This function is used to merge recursively two hashes. The
# right array always win except when a '*_' key is found 
# which can override the default behavior. See the documentation
# for usage and complete examples. Return the merged array.

def merge_recursively(o_src, o_over, p=nil)
  # o_src for source object
  # o_over for overriding object
  # p for params (string or hash)

  result = 'UNSET'

  # Retrieve parameters via yaml object or\
  # Retrieve parameters via parameters
  if p.class == String
    action = p
    options = nil
  elsif p.class == Hash
    action = p.fetch('action','union').to_s
    options = p.fetch('options', nil)
  else
    action = 'union'
    options = nil
  end

  # Debug
  #print "===> (", action, ") Working with two objects: src: ",o_src.class, ", dst: ", o_over.class, "\n"
  #print "====> Work on key: ", key, " (", action ,") \n== OLD (", val_old.class ,") : ", YAML::dump(val_old), "\n== NEW (" , val_new.class ,") : " , YAML::dump(val_new) , "\n\n"
  #print "====> Work on key: ", key, " (", action ,") \n== OLD (", val_old.class ,") : ", val_old, "\n== NEW (" , val_new.class ,") : " , val_new , "\n\n"

  # Do the action
  case action

  when 'replace'
    # Human name: replace
    # Return inconditionnaly the overriding object, even if undefined/empty.
    result = o_over

  when 'unset'
    # Human name: unset
    # Unset the value
    result = nil

  when 'union'
    # Human name: merge
    # Merge all objects, existing object are overrided

    # Is overrinding object equal nil or empty?
    if o_over.nil? or o_over.to_s.empty?
      result = o_src

    # Are both object Hash ?
    elsif o_src.class == Hash and o_over.class == Hash
      result = o_src.merge(o_over)

      # Work on sub-objects
      result = result.each {|k, v|
        result[k] = merge_recursively(o_src[k], o_over[k], get_merge_params(o_over, k) )
      }

    # Then the new object type takes precedence
    else
      result =  o_over
    end

  when 'inherit'
    # Human name: common
    # Merge only new objects, discard parent objects

    # Is overrinding object equal nil ?
    if o_over == nil or o_over.to_s.empty?
      result = o_src

    # Are both object Hash ?
    elsif o_src.class == Hash and o_over.class == Hash

      result = merge_recursively(o_src, o_over, 'union')

      # Delete key
      result = result.delete_if {|k, v|
        not o_over.has_key?(k)
      }

    else
      result = nil
    end

  when 'intersect'
    # Human name: common only
    # Merge only common objects, existing object are overrided

    # Is overrinding object equal nil ?
    if o_over == nil or o_over.to_s.empty?
      result = o_src

    # Are both object Hash ?
    elsif o_src.class == Hash and o_over.class == Hash

      result = merge_recursively(o_src, o_over, 'union')

      result = result.delete_if {|k, v|
        not ( o_src.has_key?(k) and  o_over.has_key?(k) )
      }

    else
      result = nil
    end

  when 'difference'
    # Human name: exclude
    # Remove all listed objects, existing object are overrided

    # Is overrinding object equal nil ?
    if o_over == nil or o_over.to_s.empty?
      result = o_src

    # Are both object Hash ?
    elsif o_src.class == Hash and o_over.class == Hash

      # Remove source objects if defined in options
      o_src.each {|k, v|
        if options.class == Hash or options.class == Array
          if options.class == Hash and options.has_key?(k)
            o_src.delete(k)
          elsif options.class == Array and options.include?(k)
            o_src.delete(k)
          end
        else
          print "WARN: Options for difference must be a Hash or Array for key: ", k, "\n"
        end
      }

      # Merge and override all objects
      result = merge_recursively(o_src, o_over, 'union')

    else
      result = nil
    end

  when 'complement'
    # Human name: invert
    # Remove all listed objects, existing object are overrided, new objects are discarded if not previously defined

    # Is overrinding object equal nil ?
    if o_over == nil or o_over.to_s.empty?
      result = o_src

    # Are both object Hash ?
    elsif o_src.class == Hash and o_over.class == Hash

      result = {}
      o_src.each {|k, v|

        if ( options.class == Hash and not options.has_key?(k) ) or (options.class == Array and not options.include?(k))

          # Check if the key is defined and not nil/empty in o_over
          if o_over.has_key?(k) and (o_over[k] != nil and not o_over[k].to_s.empty? )
            result[k] = merge_recursively(v, o_over[k], 'union')
          else
            result[k] = v
          end

        elsif options.class != Hash and options.class != Array
          print "WARN: Options for difference must be a Hash or Array for key: ", k, "\n"
        end
      }
      return result

    else

      # Return object only if previously defined
      if o_src.class == Hash and o_over.class == Hash
        result = o_over
      else
        result = nil
      end

    end

  else
    print "WARN: Action is not recognized: ", action, "\n"
    result nil
  end

  # Debug
  #print "Returns: ", YAML::dump(result)

  # Return the result
  if result == "UNSET"
    puts "ERROR: You found a bug, please contact the maintener"
    return nil
  elsif result.class == Hash
    return result.delete_if {|k, v| k.end_with?('_')}
  else
    return result
  end

end




########################################
# Load and merge config settings
########################################


# Load default configuration
conf_merged = conf_default

# Load mainteneur configuration
if File.file?(vagrant_root + 'conf/vagrant.yml')
  conf_merged = merge_recursively(conf_merged, YAML.load_file(vagrant_root + 'conf/vagrant.yml') )
end

# Load user configuration
if File.file?(vagrant_root + 'local.yml')
  conf_merged = merge_recursively(conf_merged, YAML.load_file(vagrant_root + 'local.yml') )
end

# Display configuration if requested
if show_config_merged 
  puts "NOTICE: This is the merged raw configuration:"
  print YAML::dump(conf_merged)
end


########################################
# Resolve configuration links
########################################

conf_final = {}


# Settings: Boxes
# =====================

conf_final['providers'] = conf_merged['settings']['providers']

# Boxes
conf_merged['settings']['providers'].each do |key, value|

  # Get common boxes
  global_boxes = conf_merged['settings']['boxes']

  # Get local provider box override
  if attribute_is_defined(value, 'boxes')
    provider_boxes = value['boxes']
  else
    provider_boxes = {}
  end

  # Update config
  conf_final['providers'][key]['boxes'] = merge_recursively(global_boxes, provider_boxes)

end



# Instances: flavors, boxes, number
# =====================
conf_final['instances'] = {}

# Check for defined instances
if not conf_merged.has_key?('instances')
  print "ERROR: No instances are defined.\n"
  exit 1 
end

# Iterate over instances
conf_merged['instances'].each do |key, value|

  vm_config = {}

  # Do some basic checks
  # ====================


  # Manage flavors and key parameters
  # =====================
 
  # Get flavor name (default value)
  if attribute_is_defined(value, 'flavor')
    # Instance flavor is set
    vm_flavor = value['flavor']
  else
    # Fall back on default flavor
    vm_flavor = conf_merged['settings']['defaults']['flavor']
  end

  # Check flavor existance
  if not attribute_is_defined(conf_merged['settings']['flavors'],vm_flavor)
    print "ERROR: Flavor '", vm_flavor, "' is not defined.\n"
    exit 1
  end


  # Define cpu
  if attribute_is_defined(value, 'cpu') and value['cpu'].to_i > 0
    vm_config['cpu'] = value['cpu']
  elsif conf_merged['settings']['flavors'][vm_flavor]['cpu'].to_i > 0
    vm_config['cpu'] = conf_merged['settings']['flavors'][vm_flavor]['cpu']
  else
    vm_config['cpu'] = 1
    print "WARN: Cpu for instance '", key, "' is not valid. Fall back to 1 cpu.\n"
  end

  # Define memory
  if attribute_is_defined(value, 'memory') and value['memory'].to_i > 0
    vm_config['memory'] = value['memory']
  elsif conf_merged['settings']['flavors'][vm_flavor]['memory'].to_i > 0
    vm_config['memory'] = conf_merged['settings']['flavors'][vm_flavor]['memory']
  else
    vm_config['memory'] = 512
    print "WARN: Memory for instance '", key, "' is not valid. Fall back to 512 (Mb).\n"
  end

  # Define disk
  if attribute_is_defined(value, 'disk') and value['memory'].to_i > 0
    vm_config['disk'] = value['disk']
  elsif conf_merged['settings']['flavors'][vm_flavor]['disk'].to_i > 0
    vm_config['disk'] = conf_merged['settings']['flavors'][vm_flavor]['disk'] 
  else
    vm_config['disk'] = 10
    print "WARN: Disk for instance '", key, "' is not valid. Fall back to 10 (Gb).\n"
  end

  # Define box
  if attribute_is_defined(value, 'box') and attribute_is_defined(conf_merged['settings']['boxes'], value['box'])
    vm_config['box'] = value['box']
  elsif attribute_is_defined(conf_merged['settings']['boxes'],conf_merged['settings']['defaults']['box'])
    vm_config['box'] = conf_merged['settings']['defaults']['box'] 
  else
    print "ERROR: Box '", conf_merged['settings']['defaults']['box'] ,"' for '", key, "' is not valid.\n"
    exit 1
  end


  # Manage VirtualBox ports
  # =====================
  vm_config['ports'] = []
  if attribute_is_defined(value, 'ports') 
    
    if value['ports'].class == Hash

      value['ports'].each do |port|

        if port.has_key?('guest') and port.has_key?('host') \
         and port['guest'].to_i > 0 and port['host'].to_i > 0

          # TODO: Check if port is already defined/used and warn user

          p = {}
          p['guest'] = port['guest'].to_i
          p['host'] = port['host'].to_i

          if port.has_key?('protocol') \
            and ( port['protocol'] == 'tcp' or port['protocol'] == 'udp' )

            p['protocol'] = port['protocol']

          else
            p['protocol'] = 'tcp'
          end

          vm_config['ports'].push(p)

        end
      end
    else
      print "WARN: Ports list for instance '", key, "' should be a list of hashes, not '", value['ports'].class ,"'.\n"
    end
  end


  # Manage provisionners
  # =====================
  
  vm_provisionners = {}

  # Detect which provisionners should be applied on instance
  if conf_merged['settings']['defaults']['provisionners'] != {}
    vm_provisionners = merge_recursively(conf_merged['provisionners'], conf_merged['settings']['defaults']['provisionners'], 'inherit')
  end 

  # Merge instance provisionners with global provisionners
  vm_provisionners.each do |name, settings|

    # Check if provisionner exists
    if conf_merged['provisionners'][name].class == Hash
      # Check if we need to override object
      if settings.class != Hash
        settings = conf_merged['provisionners'][name]
      else
        settings = merge_recursively(conf_merged['provisionners'][name], settings)
      end
    end

    # Check if provisionners are correctly set
    if settings.class != Hash
      puts "ERROR: The provisionner '" + name + "' is not correctly set."
      abort
    end

    # Ensure priority attribute is defined, or defaulted to 0
    if attribute_is_defined(settings, 'priority')
      settings['priority'] = settings['priority'].to_i
    else
      settings['priority'] = 0
    end

    # Merge the config
    vm_provisionners[name] = settings

  end

  # Reverse sort provisionners
  vm_config['provisionners'] = vm_provisionners.sort_by { |name, settings| settings['priority'] }.reverse.to_h


  # Manage instance number
  # =====================
 
  # Detect how many instances
  if attribute_is_defined(value, 'number') and value['number'].to_i > 1

    # Generate the request number of instances
    for number in 1..value['number'] do
      # TODO: Do IP increment here
      conf_final['instances'][key + number.to_s] = vm_config.clone
    end

  elsif (
      attribute_is_defined(value, 'number') \
      and value['number'] != 0 \
    ) or not attribute_is_defined(value, 'number')

    # Write config
    conf_final['instances'][key] = vm_config
  end

end


# Final configuration is ready
# =====================

# Show debugging
if show_config_instances
  puts "NOTICE: This is the instances configuration:"
  print YAML::dump(conf_final['instances'])
end

# Show debugging
if show_config_parsed
  puts "NOTICE: This is the final configuration:"
  print YAML::dump(conf_final)
end

# Check dry run
if dry_run
  puts "NOTICE: Dry run mode enabled, not going to Vagrant."
  exit 0
end


########################################
# Magic business
########################################

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  conf_final['instances'].each do |key, value|

    # Display instance config
    # =====================
    # puts "Define instance: " + key
    # print YAML::dump(value)
    # puts ""


    # Define instance
    # =====================
    config.vm.define key do |instance|


      # Generic Instance Definition
      # =====================
      instance.vm.hostname = key

      # Disable synced folders
      instance.vm.synced_folder ".", "/vagrant", :disabled => true


      # Provisionners
      # =====================
      value['provisionners'].each do |name, settings|

        instance.vm.provision settings['type'] do |s|

          # TODO: Make this called one time at the end for Ansible

          settings['params'].each do |k, v|
            s.send("#{k}=",v)

          end

          # This is what the above block dynamically do :-D
          #s.inline = settings['params']['inline']
          #s.args   = settings['params']['args']

        end
      end


      # Provider: Libvirt
      # =====================
      instance.vm.provider "libvirt" do |l, o|

        # Instance provider
        l.cpus = value["cpu"].to_i
        l.memory = value["memory"].to_i
        l.nested = true
        l.volume_cache = 'none'
        l.keymap = 'en-us'

        # Override
        o.vm.box = conf_final['providers']['libvirt']['boxes'][value["box"]]

        # Connection settings
        l.host = conf_final['providers']['libvirt']['settings']['host']
        l.connect_via_ssh = conf_final['providers']['libvirt']['settings']['connect_via_ssh']
        l.username = conf_final['providers']['libvirt']['settings']['username']

      end
    

      # Provider: VirtualBox
      # =====================
      instance.vm.provider "virtualbox" do |v, o|

        # Instance provider
        v.customize ["modifyvm", :id, "--cpus", value["cpu"]]
        v.customize ["modifyvm", :id, "--memory", value["memory"]]
        v.customize ["modifyvm", :id, "--usb", "off"]
        v.customize ["modifyvm", :id, "--usbehci", "off"]

        # Override
        o.vm.box = conf_final['providers']['virtualbox']['boxes'][value["box"]]

        # Open network ports
        value['ports'].each do |port|
          o.vm.network 'forwarded_port', guest: port['guest'], host: port['host'], protocol: port['protocol'], auto_correct: true
        end

      end
    

      # Provider: Docker
      # =====================
      # Note: Todo

    end

  end


end

