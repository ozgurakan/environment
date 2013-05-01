# Environment Setup


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

    root@terminal:~/tools# nova image-list | grep "Ubuntu 13.04"|awk {'print $2'}
    9922a7c7-5a42-4a56-bc6a-93f857ae2346

Create a 512 MB Instance With the Image

    nova boot ubuntu-template01 --image "9922a7c7-5a42-4a56-bc6a-93f857ae2346" --flavor 2 --file root/.ssh/authorized_keys=/root/.ssh/id_rsa.pub

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

#### Login Ubuntu Template Instance

Find Service Net IP for _ubuntu-template01_ Instance

    root@terminal:~# nova show e3d3b622-b501-46c5-b371-4269d69835d5 | grep private|awk {'print $5'}
    10.180.35.100

ssh to _ubuntu-template01_

    root@terminal:~# ssh 10.180.35.100
    The authenticity of host '10.180.35.100 (10.180.35.100)' can't be established.
    ECDSA key fingerprint is bd:e0:0e:5e:55:e8:90:37:ce:95:6c:37:a4:b3:6a:5a.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.180.35.100' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 13.04 (GNU/Linux 3.8.0-19-generic x86_64)

### Configure Ubuntu Template Instance

### Create CentOS Template Instance
 todo


## Create Salt Master


## Start a Minion Instance 
