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
        server_name: "leihs.local",
        deploy_to: "/home/leihs/test/current",
        version: "master",
        document_root: "/home/leihs/test/current/public",
        passenger_ruby: "/home/leihs/.rbenv/versions/2.1.5/bin/ruby",
        user: leihs,
        mysql_user: leihs,
        mysql_password: "hNM8f7ej2sjjnlk02hd",
        mysql_database: "leihs_prod",
        secret_token: fdb800e432b92b5421209ce179506b563e176d5ad6b309a3dd6c882c0563f60b206a4eeab26ada269ad3b7c5ca4d357611d387fe13a8572964123ea16cfe9d0a,
        rotate_logs: "/home/leihs/test/shared/log/*.log",
        test_instance: True
        }
```

You can of course set some of these variables as host_vars as well, such as secret_token, to keep your site.yml a bit cleaner.


## Only deploying an instance, not setting up a server

The leihs-instance role used in this playbook can be used in deploy mode if all you want to do is deploy any version of leihs to an already configured server. Keep in mind that you will have to have run the role connecting as root before, so that the server is in the right state.

The advantage here is that if you just want to deploy a new version of the application, not configure a server, you don't need root. This allows you to e.g. give your leihs developers key-based access to your server as the leihs user, while you give your sysadmins root access, and both use the same playbooks.

Make sure that your SSH public key has been added to `files/ssh_keys` and that ansible-playbook has been run as root once before you attempt this. This is necessary because only root can authorize additional users to connect and deploy.

To use deploy mode, make sure you've set the role's `version` var to the version you want to deploy, then:

    ansible-playbook site.yml -i hosts.production -u leihs --tags=deploy

Substituting the user you've actually configured for that instance in place of `-u leihs`.

## One-time override of the version of leihs to deploy

You can pick a version of leihs to deploy on a one-by-one basis, just for one single server and just for one single execution of ansible-playbook. This is useful if, for example, you have a test server that has a copy of your production data and leihs at e.g. version 3.28.1 and you want to try out 3.29.0 with the same data before you make the same migration in production.

Since it might be cumbersome to already commit to that version by adding it to your site.yml file or to a host var, you can set the variable for one single execution, **but you must make absolutely sure that you also limit the host using the `-l` option**. Since the `version` var is global to all leihs instances when you declare it on the command-line, you would otherwise move *all* your hosts to that version, and that's exactly what you don't want.

    ansible-playbook site.yml -i hosts.staging -l leihs.local -u root --extra-vars "version=3.29.0"

I repeat, do not ever use the `version` option against your production hosts file. This example also demonstrates how smart the Ansible people are by suggesting to split your host inventories into production and staging to prevent nuclear catastrophes.

## Variables explained

The variables available in this playbook are explained in the README of the [leihs-instance role](https://github.com/psy-q/ansible-leihs-instance) itself. Most of the variables used in this role are simply passed on to that role.
