# -*- mode: ruby -*-
# vi: set ft=ruby :

# NOTE: The following variables must be set according your aws configuration:
# export AWS_ACCESS_KEY='????????????????????'                              
# export AWS_SECRET_KEY='????????????????????????????????????????'
# export AWS_PRIVATE_KEY_PATH='~/.ssh/vagrant.pem'
# export AWS_DEFAULT_REGION='us-east-1'
# export AWS_EC2_KEY_NAME='vagrant'
# export AWS_SECURITY_GROUP_NAME='vagrant'

# AMI Data
HOSTS = {
  'webserver' => {
    :amazon_image  => 'ami-9a562df2',
    :shh_username  => 'ubuntu',
    :instance_type => 't2.micro'
    }
  }

HOSTS.each do | name, info |
  Vagrant.configure(2) do |config|

    config.vm.box = 'aws'
    config.vm.synced_folder '.', '/vagrant', disabled: true

    config.vm.define name do |vm_name|
      vm_name.vm.provider :aws do |aws, override|
        aws.access_key_id        = ENV['AWS_ACCESS_KEY']
        aws.secret_access_key    = ENV['AWS_SECRET_KEY']
        aws.region               = ENV['AWS_DEFAULT_REGION']
        aws.availability_zone    = ENV['AWS_DEFAULT_REGION'] + 'a'
        aws.keypair_name         = ENV['AWS_EC2_KEY_NAME']
        aws.security_groups      = ENV['AWS_SECURITY_GROUP_NAME']
        aws.ami                  = info[:amazon_image]
        aws.instance_type        = info[:instance_type]
        aws.block_device_mapping = [{ 'DeviceName' => '/dev/sda1', 'Ebs.VolumeSize' => 8 }]
        aws.tags = {
          'Name'        => name,
          'Environment' => 'vagrant-sandbox'
          }  
        # Credentials to login to EC2 Instance
        override.ssh.username         = info[:shh_username]
        override.ssh.private_key_path = ENV['AWS_PRIVATE_KEY_PATH']
      end

      # Configuration for Ansible as Provisioner
      vm_name.vm.provision :ansible do |ansible|
        ansible.playbook          = 'playbook.yml'
        ansible.verbose           = 'v'
        ansible.host_key_checking = false
        ansible.limit             = 'all'
      end

    end
  end
end