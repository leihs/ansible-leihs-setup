# ansible-leihs-setup

An example of what an Ansible playbook for setting up [leihs](http://github.com/zhdk/leihs) servers and instances could look like.

## Very Quick Start

This assumes you have Ansible 1.9 or later and Ansible Galaxy installed on your local machine.

In an ideal world, all you would then have to do is this:

 1. Clone this repository with the `--recursive` option so you get the submodules immediately: `git clone --recursive https://github.com/psy-q/ansible-leihs-setup.git`
 1. Change to the directory that created.
 1. Add the DNS names of all the servers you want to install a leihs test instance on to the file `hosts.staging`.
 1. Choose a random MySQL root password and add it `host_vars`. You do that by creating a directory named like the host you're installing leihs on, and a YAML file that sets the `mysql_root_password` variable. See `host_vars/leihs.local/mysql_credentials` for an example.
 1. Copy the entry from site.yml so that it matches your new host and customize it to suit your needs. See below for the variables you can change.
 1. Run `ansible-galaxy install leonidas.rbenv` as root.
 1. Run `ansible-playbook site.yml -i hosts.staging -u root` as whichever user can connect to your staging hosts as root.

If all goes well, you will end up with a working leihs instance that is seeded with the default user `admin` with password `password`. It will be available via HTTP at all the hosts you entered in your staging inventory.

Once you're happy testing leihs on those hosts, proceed to create a production inventory file and add your production hosts there.

## Example entry

## Variables
