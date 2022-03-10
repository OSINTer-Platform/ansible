# OSINTansible

[![OSINTer](https://raw.githubusercontent.com/bertmad3400/OSINTer/master/logo.png)](https://osinter.dk)

## Welcome to OSINTer
This repo is a part of a larger project called
![OSINTer](https://github.com/bertmad3400/OSINTer). For more information on the
project as a whole, you can find OSINTer at
![https://github.com/bertmad3400/OSINTer](https://github.com/bertmad3400/OSINTer).

## What is OSINTansible?
OSINTansible is series of ansible playbooks, which will aid in the process of
deploying the software needed for a full OSINTer installation. While
OSINTansible is designed for being runned against a host/a series of hosts
which has just been installed, it can also be runned against againsts host(s)
with existing software installed. Do keep in mind that doing so will probably
interfere with the following list of software if it's already installed:
- Nginx
  - It will replace the nginx config file found at /etc/nginx/nginx.conf
	- This will make the nginx service run as the osinter-web user
  - It will create a new site in /etc/nginx/site-available and create a symlink
	to that in /etc/nginx/sites-enabled
  - It will restart the service
- PostgreSQL
  - On non-Debian based systems, it will try to initialize a new DB cluster in
	the default location.
	- For Arch this is var/lib/postgres/data as defined in the Archlinux.json
	  file and for CentOS/Rocky Linux this is /var/lib/pgsql/data as pre-defined
	  by the OS
  - It will replace the postgres.conf file and the pg_hba.conf file
	  - This will keep the authenification method for the postgres user as being
		peer-authentification
	  - This will prevent any kind of connection to the DB using anything else
		but the unix socket
  - It will create a new DB and a whole range of new users described in the
	pg_hba.conf file
  - It will restart the service

## Quickstart
- Install the ansible package from your repos along with python 3 and the
  "cryptography" python package.
- Setup a remote server (which is going to host the OSINTer server)
- Setup a new server using one of the supported distributions listed below and
  configure a regular user with sudo and SSH access using an SSH key.
  While it is not neccessarily needed to install python 3 on the remote machine,
  it is recommended, to limit the possibilities of bugs.
- Clone this git repo and navigate to that directory
- Execute ``` ansible-playbook -K playbooks/main.yml -u [regular_user] -i
  [remotes] --key-file [private_key_location] ``` with remotes being a comma
  seperated list of remote servers (add a single trailing comma if only using
  one remote server), regular_user being the regular, non-root user you set up
  just before and private_key_location being the path to the private key for
  your ssh connection.
- When using an SSH key, remember to protect it with ```chmod 400
  [private_key_location]```.
- Supply the password for the [regular_user] when asked for the "BECOME
  password". This will be used for sudo priviledge escalation when needed.
- For distributions with a firewall pre-installed (CentOS and Rocky Linux) you
  can use the firewall.yml playbook to allow HTTP and HTTPS traffic or do this
  manually.

#### Examples:
- Setup OSINTer on a Debian server running on 10.0.0.25 using the user mark:
  - ``` ansible-playbook -K playbooks/main.yml -u mark -i 10.0.0.25, --key-file ./id_rsa ```
- Setup OSINTer on two Arch servers running on 172.56.4.21 and 154.32.34.2 using
  the user emma:
  - ``` ansible-playbook -K playbooks/main.yml -u emma -i 172.56.4.21,154.32.34.2 --key-file ./id_rsa ```
- Setup OSINTer on a Rocky Linux server running on 10.0.0.2 using the user john
  and disable the firewall:
  - ``` ansible-playbook -K playbooks/main.yml -u john -i 10.0.0.2, --key-file ./id_rsa ```
  - ``` ansible-playbook -K playbooks/firewall.yml -u john -i 10.0.0.2, --key-file ./id_rsa ```

## Supported distributions:
Currently, the only supported distributions is the latest version of the
following distributions on the x86_64 architecture:
- Debian 11 and 10
- Arch
- Ubuntu Server 20
- Rocky Linux 8
We do recommend running OSINTer on Arch Linux, as this has proven to have a
substantial perfomance increase over the other distributions during our very
extensive testing, but we do realize that this unfortunatly isn't possible for
everyone, and therefore we fully support the other platforms listed.

## Using own CA for TLS certificate
OSINTansible will automatically setup certificates for use with TLS
certificates, but these will be self signed. If it is needed to have those
signed by an existing CA, the proccess is as follows:
- Rename a copy of the CA private key to "ca.key" and move it to the "./vars/CA"
  directory in the OSINTansible
- Rename a copy of a certificate signed by the CA to "ca.crt" and move it to the
  same directory
- Now simply follow the instructions in the guick guide. The playbooks will
  automatically recognize the CA and use it for signing the certificates.

### Using SELinux
OSINTansible is designed to be able to deal with systems that has SELinux
enabled as standard (like CentOS 8), and as such, it shouldn't be a problem to
have SELinux enabled. The functionality is still tending to the buggy side,
however, and therefore we recommend that if OSINTer doesn't work after
deployment using OSINTansible, to try and deploy it again, but this time with
SELinux being set temporarily to permissive. If this solves the problem, please
contact us so that we can fix the issue.

There is also a few things you should keep in mind if using OSINTansible with
SELinux:
- It will give nginx full read-access to the whole /srv/OSINTwebserver directory
  using the ```httpd_sys_content_t``` context.
- It will make the socket file created by the OSINTwebserver services using
  gunicorn in the /srv/OSINTwebserver writeable for nginx using the
  ```httpd_var_run_t``` context.
- It will make every file in the /srv/OSINTwebserver/OSINTwebserverenv/bin
  executable using the ```bin_t``` context.
- It will make every file in the /srv/elasticsearch/elasticsearch-*/bin
  executable using the ```bin_t``` context.
