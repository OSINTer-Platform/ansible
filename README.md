# OSINTer

![OSINTer](https://github.com/Combitech-DK/OSINTer/blob/master/logo.png)

## OSINTer-ansible
OSINTer-ansible is series of ansible playbooks, which will aid in the process of
deploying the software needed for a full OSINTer installation. While
OSINTer-ansible is designed for being runned against a host/a series of hosts
which has just been installed, it can also be runned against againsts host(s)
with existing software installed. Do keep in mind that doing so will probably
interfere with the following list of software if it's already installed:
- Nginx
- PostgreSQL

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

#### Examples:
- Setup OSINTer on a Debian server running on 10.0.0.25 using the user mark:
  - ``` ansible-playbook -K playbooks/main.yml -u mark -i 10.0.0.25, --key-file ./id_rsa ```
- Setup OSINTer on two Arch servers running on 172.56.4.21 and 154.32.34.2 using
  the user emma:
  - ``` ansible-playbook -K playbooks/main.yml -u emma -i 172.56.4.21,154.32.34.2 --key-file ./id_rsa ```

## Supported distributions:
Currently, the only supported distributions is the latest version of the
following distributions:
- Debian
- Arch

## Using own CA for TLS certificate
OSINTer-ansible will automatically setup certificates for use with TLS
certificates, but these will be self signed. If it is needed to have those
signed by an existing CA, the proccess is as follows:
- Rename a copy of the CA private key to "ca.key" and move it to the "./vars/CA"
  directory in the OSINTer-ansible
- Rename a copy of a certificate signed by the CA to "ca.crt" and move it to the
  same directory
- Now simply follow the instructions in the guick guide. The playbooks will
  automatically recognize the CA and use it for signing the certificates.