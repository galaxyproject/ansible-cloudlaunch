Ansible playbook for deploying [CloudLaunch](https://github.com/galaxyproject/cloudlaunch/tree/dev).

## Install
To install, just clone this repository:

```
git clone https://github.com/galaxyproject/ansible-cloudlaunch.git
```

## Use
This playbook is intended to run on Ubuntu 16.04, using Ansible 2.2.1.0 or higher.

Create the inventory file called `inventory` to point to the server you want
to configure with the following content, adjusting for variables:

```
[cloudlaunch-webserver]
149.165.157.111 ansible_ssh_user="ubuntu" ansible_ssh_private_key_file="key.pem" ansible_ssh_common_args='-o StrictHostKeyChecking=no -o CheckHostIP=no -o "UserKnownHostsFile /dev/null"'
```

Edit the configuration values. In particular, the cloudlaunch admin password (for django admin, the database password and the server name should be changed. You will also need to provide your own SSL cert
files. The default values are stored in `roles/cloudlaunch/defaults.yml` but
you should use `local_vars.yml` file in the repo root directory (as defined
in `playbook.yml`) to change the most frequently edited values. Note that this
file must exist for the playbook to run. Here is an example of that file's
content:

```
cloudlaunch_admin_password: "CHANGEMEONINSTALL"
dbpassword: "CHANGEMEONINSTALL"
server_name: beta.launch.usegalaxy.org
# These files need to be placed into roles/cloudlaunch/files/secret and vaulted
ssl_cert_files:
  - beta.launch.usegalaxy.org_certchain.pem
  - cl.crt
  - cl.key
```

Store the vault password to `.vault_pass` in the repo root dir and run the
playbook with the following command:

```
$ ansible-playbook -i inventory playbook.yml
```

## Restoring default application data

Once installed, you can populate the server with default data, such as the list of public clouds. The default data is stored in cloudlaunch_clouds_and_apps.json in the playbook folder. To restore default data, run:

```
$ ansible-playbook -i inventory restoredb.yml
```

## Copying the database from server1 to server2

Run these commands after you have run the playbook.
On _server1_, create a dump of the `cloudlaunch` database:

```
$ sudo su postgres
$ pg_dump -Fc -f cl.dump cloudlaunch
```

Copy `cl.dump` file from _server1_ to _server2_ and run the following commands
to import the database:
```
# Stop the CloudLaunch server so the empty database can be deleted
$ sudo supervisorctl stop cloudlaunch celeryd
# Become the postgres system user
$ sudo su postgres
# Delete the (empty) database
$ dropdb cloudlaunch
# Create the database, using an empty template
$ createdb -T template0 cloudlaunch
# Restore the database from the file copyied from server1
$ pg_restore -d cloudlaunch cl.dump
# Return to the default user
$ exit
# Start the CloudLaunch server
$ sudo supervisorctl start cloudlaunch celeryd
```
