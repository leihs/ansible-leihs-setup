# ansible-leihs-setup

An Ansible playbook for setting up [leihs](http://github.com/zhdk/leihs) servers and instances could look like. It supports Debian GNU/Linux 8.1 (jessie).

## Very Quick Start

This assumes you have Ansible 1.9 or later and Ansible Galaxy installed on your local machine.

In an ideal world, all you would then have to do is this:

 1. Clone this repository with the `--recursive` option so you get the submodules immediately: `git clone --recursive https://github.com/leihs/ansible-leihs-setup.git`
 1. Change to the directory that created.
 1. Add the DNS names of all the servers you want to install a leihs test instance on to the file `hosts.staging`.
 1. Choose a random MySQL root password and add it `host_vars`. You do that by creating a directory named like the host you're installing leihs on, and a YAML file that sets the `mysql_root_password` variable. See `host_vars/leihs.local/mysql_credentials` for an example.
 1. Copy the entry from site.yml so that it matches your new host and customize it to suit your needs. See below for the variables you can change.
 1. (Optional) Add the SSH public keys of all the users that you want to authorize to deploy the leihs instances to the directory `files/ssh_keys`.
 1. Run `ansible-playbook site.yml -i hosts.staging -u root` as whichever user can connect to your staging hosts as root.

If all goes well, you will end up with a working leihs instance that is seeded with the default user `admin` with password `password`. It will be available via HTTP at all the hosts you entered in your staging inventory.

Once you're happy testing leihs on those hosts, proceed to create a production inventory file and add your production hosts there.

## Example entry in site.yml

This should get you started with a test instance:


```yaml
---
- hosts: leihs.local
  roles:
    - { role: leihs-instance,
        deploy_to: "/home/leihs/test/current",
        version: "master",
        ruby_version: "2.3.1",
        user: leihs,
        authorized_for_deployment:
          ["files/ssh_keys/rca.pub",
           "files/ssh_keys/fs.pub",
           "files/ssh_keys/nimaai.pub"],
        mysql_user: leihs,
        mysql_password: "hNM8f7ej2sjjnlk02hd",
        mysql_database: "leihs_prod",
        secret_token: abc123,
        logrotate_filename: "leihs.local",
        test_instance: True
        }
    - { role: phusion-passenger-vhost,
        server_name: "leihs.local" }
```

You can of course set some of these variables as host_vars as well, such as secret_token, to keep your site.yml a bit cleaner.

## Variables explained

The variables available in this playbook are explained in the README of the [leihs-instance role](https://github.com/leihs/ansible-leihs-instance) itself. Most of the variables used in this role are simply passed on to that role. Since phusion-passenger-vhost uses many of the same variable names and can get them from host vars, the only one you really have to set manually is the `server_name`. This is what gets set as `ServerName` in the Apache virtual host configuration.

The leihs-instance role takes care of setting those variables. If you want to use phusion-passenger-vhost without leihs-instance, you would of course have to set them yourself.
