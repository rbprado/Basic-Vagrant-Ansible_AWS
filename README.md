Provision and configure a Gerrit master in AWS EC2 instance with Vagrant and Ansible
=====================================================================================


This is repository contains all you need to provision a Gerrit master EC2 VM at a AWS VPC.
You just need to edit the configuration at the roles/gerrit/defaults/main.yml and roles/gerrit/vars/main.yml according your needs.

The configuration at the file roles/gerrit/defaults/main.yml set the ldap server for the Gerrit, to setup a freeipa as ldap server see the following repository:
http://github.com/rbprado/provision-freeipa-aws


## Required packages:
For first, install the following packages at your unix system:
* vagrant 1.6.5 or higher
* ansible 2.0.0.2 or higher


## Install the vagrant-aws plugin:
The vagrant-aws plugin can be installed executing the following:
```
$ vagrant plugin install vagrant-aws
```


## AWS account set up:
Please set up your AWS account and export the following variables:
```
$ export AWS_ACCESS_KEY='????????????????????'
$ export AWS_SECRET_KEY='????????????????????????????????????????'
$ export AWS_PRIVATE_KEY_PATH='~/.ssh/vagrant.pem'
$ export AWS_DEFAULT_REGION='us-east-1'
$ export AWS_EC2_KEY_NAME='vagrant'
$ export AWS_SECURITY_GROUP_NAME='vagrant'
```

NOTE:
You can set it into your ~/.profile or ~/.bashrc.
To reach the data to set the variables follow the below instructions,
starting from your AWS console:
* AWS_ACCESS_KEY and AWS_SECRET_KEY:
  1. Go to Security Credentials
  2. Create a new Access Key 
  3. Obtain the data at the fields: Access Key ID and Secret Access Key

* AWS_DEFAULT_REGION
  1. Go to Services
  2. EC2
  3. Select the region

* AWS_PRIVATE_KEY_PATH and AWS_EC2_KEY_NAME
  1. Go to Services
  2. EC2
  3. Key Pairs
  4. Create Key Pairs
  5. Copy the file to the path

* AWS_SECURITY_GROUP_NAME
  1. Go to Services
  2. EC2
  3. Security Groups 
  4. Create Security Group
  5. Add a inbound rule for HTTP and SSH from anywhere.


## Edit data at the Vagrantfile
Please configure the following keys at the Vagrantfile file according to your AWS setup:
```
HOSTS = {
  'gerrit' => {
    :amazon_image   => 'ami-fce3c696',
    :shh_username   => 'ubuntu',
    :instance_type  => 't2.micro',
    :disk_size_gb   => 10,
    :security_group => 'sg-4c72a434',
    :elastic_ip     => '50.16.175.221',
    :private_ip     => '172.31.0.20',
    :subnet_id      => 'subnet-4168ec37'    
  }
}
```
NOTE 1: You need to create an elastic IP for your VM, without it after each reboot you will get a new IP.

NOTE 2: Do not forget to open the needed ports at your security group.

NOTE 3: While using VPC, the security group must be set through ID and not by it's name.


## Edit the defaults for the VM
At the file gerrit/defaults/main.yml is possible configure some gerrit intallation data.
Please set these data according your gerrit configuration:
```
#Gerrit data for installation
gerrit_account_name: gerrit
gerrit_account_comment: Gerrit User
gerrit_account_home: "{{ gerrit_install_dir }}"
gerrit_account_group: gerrit
gerrit_global_pass: $6$ORmN2r3j8$YIWsVOdiJzwYmbI0.d/n5ggM0K6R6voWeQaI97zeDqMPS2934D3FWp/YWHFxBsvcg0mdocMnL7Q/xsimxZjTA. #gerrit123 sha256 hashed

gerrit_db_type: mysql
gerrit_db_name: reviewdb
gerrit_db_user: gerrit
gerrit_db_host: localhost

gerrit_dl_url: https://www.gerritcodereview.com/download
gerrit_dl_sha256sum: 8f2e2e61b939664a20a025ef4f99095c08be14342ea5457bf5613bccedabcd27

gerrit_http_listen_port: 8080
gerrit_init: "java -jar {{ gerrit_war_path }} init --batch --no-auto-start -d {{ gerrit_install_dir }} --install-plugin reviewnotes --install-plugin download-commands --install-plugin singleusergroup"
gerrit_war_dir: /opt/
gerrit_war_path: "{{ gerrit_war_dir }}/{{ gerrit_war_name }}"
gerrit_war_name: "gerrit-{{ gerrit_version }}.war"
gerrit_mysql_connector_file: mysql-connector-java-5.1.21.jar
gerrit_mysql_connector_url: http://repo2.maven.org/maven2/mysql/mysql-connector-java/5.1.21
gerrit_install_dir: /var/gerrit
gerrit_smtp_server: localhost
gerrit_sshd_listen_port: 29418
gerrit_version: 2.12

#MariaDB MYSQL data for installation
mariadb_debian_repo: 'deb http://ftp.osuosl.org/pub/mariadb/repo/{{ mariadb_version }}/{{ ansible_distribution|lower }} {{ ansible_distribution_release|lower }} main'
mariadb_debian_repo_key: '0xcbcb082a1bb943db'
mariadb_debian_repo_keyserver: 'keyserver.ubuntu.com'
mariadb_redhat_repo: 'http://yum.mariadb.org/{{ mariadb_version }}/{{ ansible_distribution|lower }}{{ ansible_distribution_major_version|int}}-amd64'
mariadb_redhat_repo_key: 'https://yum.mariadb.org/RPM-GPG-KEY-MariaDB'
mariadb_version: 10.0
mysql_allow_remote_connections: false  #defines if mysql should listen on loopback (default) or allow remove connections
mysql_port: 3306  #defines the port for mysql to listen on
```
NOTE 1: Here we are using the public dns dynamic set by amazon as hostname, ofcourse it's just for testing purposes, at production you need a registered domain, or you can configure the already mentioned freeipa to be a DNS resolver, then you can use the package freeipa-client to register your new EC2 instance to the freeipa resolver.
NOTE 2: For example if you change the gerrit_version your will need to change the gerrit_dl_sha256sum according the new file to be downloaded from the https://www.gerritcodereview.com/download.


## Usage
After set the configuration listed above, just try the following line:
```
$ vagrant up --provider=aws
```

To access the command line of your EC2 instance execute the following line:
```
$ vagrant ssh
```

To access the Web UI just use the AWS Public DNS generated for the EC2 instance.
The http link will be printed at the end of the provision.

Never forget to destroy your EC2 instance after testing:
```
$ vagrant destroy
```


## License

BSD

Thank you for trying this environment and be free to contribute :.
