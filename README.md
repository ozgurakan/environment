# Environment Setup

We will create a terminal server where we will have nova client to create instances.
Then we will create base images for Ubuntu and CentOS. These images will have latest 
updates to bootstrapping a new instance will be faster and we will know exactly what 
we are getting.


## Create Terminal Server

Create the terminal server on mycloud.rackspace.com

OS: Ubuntu 13.04 4GB

#### Login to terminal server as root with password

    MK40T8DKQ4:~ oz$ ssh 166.78.147.186 -l root
    The authenticity of host '166.78.147.186 (166.78.147.186)' can't be established.
    root@166.78.147.186's password:
    Last login: Wed May  1 14:03:06 2013 from 10.181.130.221

#### Create ssh Key

    root@terminal:~# ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa):
    Created directory '/root/.ssh'.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /root/.ssh/id_rsa.
    Your public key has been saved in /root/.ssh/id_rsa.pub.

#### Create tools folder

    root@terminal:~# mkdir tools
    root@terminal:~# cd tools/
    root@terminal:~/tools#

#### Install pip

    root@terminal:~/tools# apt-get install python-pip python-dev build-essential  -y

#### Install screen

Screen is not essential but makes it really easy to move between servers

    root@terminal:~/tools# apt-get install screen

#### Install nova client

    root@terminal:~/tools# pip install rackspace-novaclient

#### Configure nova Credentials

Create a file named nova-credentials with this content for US Data Center:

    # source <this file>
    export OS_AUTH_URL=https://identity.api.rackspacecloud.com/v2.0/
    export OS_AUTH_SYSTEM=rackspace
    export OS_REGION_NAME=DFW
    export OS_USERNAME=<username>
    export OS_TENANT_NAME=<tenant_id>
    export NOVA_RAX_AUTH=1
    export OS_PASSWORD=<api_key>
    export OS_PROJECT_ID=<tenant_id>
    export OS_NO_CACHE=1

For UK Datacenter Auth URL is different

    export OS_AUTH_URL=https://lon.identity.api.rackspacecloud.com/v2.0/

On terminal server

    root@terminal:~/tools# source nova-credentials

Test If nova client works fine

    root@terminal:~/tools# nova credentials


## Create Template Instances

We will use these instances to create base images.

### Create Ubuntu Template Instance

On terminal server find the image id for desired Ubuntu version

    root@terminal:~/tools# nova image-list | grep "Ubuntu 13.04" | awk {'print $2'}
    9922a7c7-5a42-4a56-bc6a-93f857ae2346

Create a 512 MB Instance With the Image

    nova boot ubuntu-template01 --image "9922a7c7-5a42-4a56-bc6a-93f857ae2346" --flavor 2 --file /root/.ssh/authorized_keys=/root/.ssh/id_rsa.pub

    +------------------------+--------------------------------------+
    | Property               | Value                                |
    +------------------------+--------------------------------------+
    | status                 | BUILD                                |
    | updated                | 2013-05-01T15:58:42Z                 |
    | hostId                 |                                      |
    | image                  | Ubuntu 13.04 (Raring Ringtail)       |
    | OS-EXT-STS:task_state  | scheduling                           |
    | OS-EXT-STS:vm_state    | building                             |
    | flavor                 | 512MB Standard Instance              |
    | id                     | e3d3b622-b501-46c5-b371-4269d69835d5 |
    | user_id                | <user id>                            |
    | name                   | ubuntu-template01                    |
    | adminPass              | Rxyzysyzyzy2                         |
    | tenant_id              | <tenant id>                          |
    | created                | 2013-05-01T15:58:42Z                 |
    | OS-DCF:diskConfig      | AUTO                                 |
    | accessIPv4             |                                      |
    | accessIPv6             |                                      |
    | progress               | 0                                    |
    | OS-EXT-STS:power_state | 0                                    |
    | metadata               | {}                                   |
    +------------------------+--------------------------------------+

Trick here is the public key of terminal server that we passed to the instance.

#### Login Ubuntu Template Instance

Find Service Net IP for _ubuntu-template01_ Instance

    root@terminal:~# nova show e3d3b622-b501-46c5-b371-4269d69835d5 | grep private|awk {'print $5'}
    10.180.35.100

As we had copied public key of terminal server to this instance, it will let us login without a password.

ssh to _ubuntu-template01_

    root@terminal:~# ssh 10.180.35.100
    The authenticity of host '10.180.35.100 (10.180.35.100)' can't be established.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.180.35.100' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 13.04 (GNU/Linux 3.8.0-19-generic x86_64)

#### Configure Ubuntu Template Instance

Install salt minion

    root@salt-master01:~# apt-get install software-properties-common -y
    root@salt-master01:~# add-apt-repository ppa:saltstack/salt
    root@salt-master01:~# apt-get update
    root@ubuntu-template01:~# apt-get install salt-minion -y 


salt-minion is going to start running with default configuration. It won't be able to connect to master until we
change the minion configuration file. We would like to change it while we are booting up a new instance for the first
time.

#### Create an Image From Ubuntu Template Instance

With nova client create take an image of ubuntu template instance. We will use this image as base image later on.
We will use the instance id of ubuntu-template01 : _e3d3b622-b501-46c5-b371-4269d69835d5_

    root@terminal:~/tools# nova image-create e3d3b622-b501-46c5-b371-4269d69835d5 ubuntu1304-salt-base01

This will take some time. 

### Create CentOS Template Instance

On terminal server find the image id for the desired CentOS version

    root@terminal:~/tools# nova image-list | grep -i "centos 6.3" | awk {'print $2'}
    da1f0392-8c64-468f-a839-a9e56caebf07

Create a 512MB instance with this image

    root@terminal:~/tools# nova boot centos63-template --image "da1f0392-8c64-468f-a839-a9e56caebf07" --flavor 2 --file /root/.ssh/authorized_keys=/root/.ssh/id_rsa.pub

### Login to CentOS Template Image

Find IP Address of centos63-template instance.

    root@terminal:~# nova show 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef | grep private|awk {'print $5'}

ssh to centos63-template, It will ask the password that was displayed in the output of nova boot command.

    root@terminal:~# ssh 10.182.22.104
    The authenticity of host '10.182.22.104 (10.182.22.104)' can't be established.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.182.22.104' (RSA) to the list of known hosts.
    root@10.182.22.104's password:
    [root@centos63-template ~]#
    
Update the system:
    
    root@terminal:~# yum update -y
    .....
    Complete!


### Create An Image From CentOS Template Instance

    root@terminal:~/tools# nova image-create 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef centos63-base01

Check the status:

    root@terminal:~/tools# nova image-list|grep centos63-base01 | awk {'print $4" status: "$6'}
    centos63-base01 status: SAVING
    root@terminal:~/tools# nova show 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef | grep task_state | awk {'print $4'}
    image_uploading
    root@terminal:~/tools# nova show 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef | grep task_state | awk {'print $4'}
    None
    root@terminal:~/tools# nova image-list|grep centos63-base01 | awk {'print $4" status: "$6'}
    centos63-base01 status: ACTIVE

Our image is ready so we can terminate centos63-template

    root@terminal:~/tools# nova delete 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef


## Create Salt Master

### Start Salt Master Instance (Ubuntu)

We will use the base image we created from ubuntu-template01 instance. It only had minion. We will start the instance, disable salt minion, install salt master and a few other packages.

#### Find Image Id for ubuntu1304-salt-base01

    root@terminal:~/tools# nova image-list | grep "ubuntu1304-salt-base01" | awk {'print $2'}
    e8c15811-0852-47ab-b4d9-bcacc6f82119

Get the id from the output of the previous command and find IP address of the instance.

    root@terminal:~/tools# nova show 41b1ea8f-ff35-4c97-9a01-e07d3e2d78ef | grep private|awk {'print $5'}
    10.182.22.104


#### Create Salt Master Instance

    root@terminal:~/tools# nova boot salt-master01 --image "e8c15811-0852-47ab-b4d9-bcacc6f82119" --flavor 4
    +------------------------+--------------------------------------+
    | Property               | Value                                |
    +------------------------+--------------------------------------+
    | status                 | BUILD                                |
    | image                  | ubuntu1304-salt-base01               |
    | OS-EXT-STS:task_state  | scheduling                           |
    | OS-EXT-STS:vm_state    | building                             |
    | flavor                 | 2GB Standard Instance                |
    | id                     | 0f646c90-d251-43db-ad62-4b7052e8b34f |
    | name                   | salt-master01                        |
    | created                | 2013-05-01T18:09:49Z                 |
    +------------------------+--------------------------------------+

I removed some of the rows from above. 

#### Check Progress

    root@terminal:~/tools# nova show 0f646c90-d251-43db-ad62-4b7052e8b34f | grep progress | awk {'print "Instance Creation Progress " $4"%"'}
    Instance Creation Progress 100%

Once we see 100% we can login to the server

#### Login To Salt Master

Find Service Net IP Address for salt-master01

    root@terminal:~/tools# nova show 0f646c90-d251-43db-ad62-4b7052e8b34f | grep "private network" | awk {'print $5'}
    10.181.139.92

ssh to the server

    root@terminal:~# ssh 10.181.139.92
    The authenticity of host '10.181.139.92 (10.181.139.92)' can't be established.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.181.139.92' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 13.04 (GNU/Linux 3.8.0-19-generic x86_64)


### Configure Salt Master Instance

#### Install pip

    root@salt-master01:~# apt-get install python-pip -y

#### Install emacs _optional_

On this server we will do lots of editing so install your favorite text editor.

    root@salt-master01:~# apt-get install emacs -y

#### Install salt packages

We don't really need to run these three commands as they were run for the template image.
If you are booting up this instance from a plain Ubuntu image, then you need to run them.

    root@salt-master01:~# apt-get install software-properties-common -y
    root@salt-master01:~# add-apt-repository ppa:saltstack/salt
    root@salt-master01:~# apt-get update

Install Required Packages
    
    root@salt-master01:~# apt-get install salt-master
    root@salt-master01:~# pip install salt-cloud
    root@salt-master03:~# pip install apache-libcloud
    root@salt-master03:~# pip install botocore
    root@salt-master03:~# apt-get install sshpass

### Configure salt-cloud For Rackspace OpenStack

We need two files to configure salt-cloud. 

First one defines the cloud provider. 

    root@salt-master03:~# mkdir /etc/salt/cloud.providers.d
    root@salt-master03:~/# touch /etc/salt/cloud.providers.d/rackspace.conf

Content of the file:

    marconi-rackspace-dfw:
      minion:
        master: <IP address of salt master>

      identity_url: 'https://identity.api.rackspacecloud.com/v2.0/tokens'
      protocol: ipv4

      compute_region: DFW

      user: <username>
      tenant: <tenant id>
      apikey: <API key>

      provider: openstack

Above _marconi-rackspace-dfw_ is the name of the provider that we choose.

Second file is more generic. It has profiles (definitions of VMs). To be able to 
create this file first we need to find out what tpyes of instaces and images are available.

List instance types:

    root@salt-master03:/etc/salt/cloud.providers.d# salt-cloud --list-sizes marconi-rackspace-dfw
    openstack
      15GB Standard Instance
        disk: 620
        id: 7
        ram: 15360
        uuid: 0ef9c73c90226fb4e49854943d9b97a42ca75d7a
      1GB Standard Instance
        disk: 40
        id: 3
        ram: 1024
        uuid: 916b53726166c76dc51eeccd7ffc79a337a912bc
      2GB Standard Instance
        disk: 80
        id: 4
        ram: 2048
        uuid: b8122b232b105e228f1fd46488a6f731c877063c
      30GB Standard Instance
        disk: 1200
        id: 8
        ram: 30720
        uuid: a925708be0cf852459a1cc9668b7266704e29b32
      4GB Standard Instance
        disk: 160
        id: 5
        ram: 4096
        uuid: 8f0083f719dbc84a16323b7ad37ef6d0f240dba9
      512MB Standard Instance
        disk: 20
        id: 2
        ram: 512
        uuid: 4fdfc2dbc25fe8e7d640be69eb5e201382996d62
      8GB Standard Instance
        disk: 320
        id: 6
        ram: 8192
        uuid: dcb6758ab0f6a1f88b97ffb1156bcc5e4eac6820
 
 List image types:

    root@salt-master03:~# salt-cloud --list-images marconi-rackspace-dfw
    ...clipped....
      CentOS 6.3
        extra:
          created: 2013-04-04T15:18:17Z
          metadata: {u'os_distro': u'centos', u'com.rackspace__1__kickstart_qc': u'pass', u'com.rackspace__1__visible_core': u'1', u'com.rackspace__1__build_rackconnect': u'1', u'com.rackspace__1__release_id': u'212', u'image_type': u'base', u'com.rackspace__1__release_build_date': u'2013-03-06_17-09-15', u'com.rackspace__1__source': u'kickstart', u'org.openstack__1__os_distro': u'org.centos', u'cache_in_nova': u'True', u'com.rackspace__1__visible_rackconnect': u'1', u'com.rackspace__1__release_version': u'2', u'org.openstack__1__os_version': u'6.3', u'auto_disk_config': u'True', u'com.rackspace__1__options': u'0', u'os_type': u'linux', u'com.rackspace__1__build_core': u'1', u'com.rackspace__1__visible_managed': u'1', u'org.openstack__1__architecture': u'x64', u'com.rackspace__1__build_managed': u'1'}
          minDisk: 0
          minRam: 512
          progress: 100
          serverId: None
          status: ACTIVE
          updated: 2013-04-22T17:20:03Z
        id: da1f0392-8c64-468f-a839-a9e56caebf07
        uuid: 56e543f881cad99fa21aa4bf06cb20ba578adc40

Now lets create profiles file for rackspace

    root@salt-master03:~# touch /etc/salt/cloud.profiles.d/rackspace.conf

Content of the file:    

    centos_512:
        provider: marconi-rackspace-dfw
        size: 512MB Standard Instance
        image: CentOS 6.3

    centos63-base_512:
        provider: marconi-rackspace-dfw
        size: 512MB Standard Instance
        image: centos63-base01

    ubuntu_512:
        provider: marconi-rackspace-dfw
        size: 512MB Standard Instance
        image: Ubuntu 13.04 (Raring Ringtail)

    ubi_512:
        provider: marconi-rackspace-dfw
        size: 512MB Standard Instance
        image: ubuntu1304-salt-base02

    ubuntu1204_512:
        provider: marconi-rackspace-dfw
        size: 512MB Standard Instance
        image: Ubuntu 12.10 (Quantal Quetzal)

Lets test to create an instance:

    root@salt-master03:~# salt-cloud -l debug -p centos63-base_512 delete-me-centos-base

Once the instance is ready, lets check if salt master can talk to it:

    root@salt-master03:~# salt 'delete-me-centos-base' test.ping
    delete-me-centos-base:
        True

If you login to the instance, you will see that salt minion has been installed and configured and 
is talking to salt master.

    

# Lets Deploy MongoDB


