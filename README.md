init_server
=========

This role initializes a server with the following actions :
* upgrade packages
* remove unecessary packages / files
* create users
* configure sudoers
* configure sshd
* collect facts

Requirements
------------

This role mostly uses mainly ansible.builtin modules but also authorized_key and sysctl from ansible.posix collection

Role Variables
--------------

This role uses variables necessary for initialization of the server retrieved from inventory:
* inventory_hostname and inventory_hostname_short : hostname to be set on the server

Variables from default directory :
* host_user : user to be created on the server
* host_password : user password
* ssh_public_key : public SSH key to be used (in ed25519 format)
* default_sshd_port : sshd port on which sshd daemon listens (10022 by default)
* host_admin_user : admin user (root or ubuntu) - used to configure sudoers (root by default)

Variables from vars directory (OS specific):
* OS specific :
  * packages_to_remove : list of packages that we want to remove from default delivered servers
  * files_to_remove : list of files / directories to remove from default delivered servers
  * users_to_remove : list of users to remove from default delivered servers
* Global :
  * default_ssh_public_keys : ssh_public_key + extra_public_key (if exists)
  * sysctl_disable_ipv6_keys : sysctl entries to activate to disable ipv6

This role also makes use of variables gathered from facts :
* ansible_os_family : Family of Operating System (Debian or RedHat)

This role also configures backup servers where daily facts should be pushed :
* backup_sftp_user : user to be configured on backup server used to push facts
* These backup servers should be in group backup_server (if none then corresponding tasks do nothing)
* These backup servers should have the following variables defined in host_vars :
  * host_server_public_key : public key of backup server
  * host_server_known_entry : hashed host to be added to known_hosts

OPTIONAL variables (tasks will not be executed if not defined) :
* extra_public_key : extra public keys to be allowed for created user
* host_user2 / host_password2 / host_user2_pubkey (OPTIONAL) : allows creation of a second user with specific public key

Dependencies
------------

None

Example Playbook
----------------

For initialization (until non privileged user and SSHD are configured) :
      - hosts: all
        become: true
        gather_facts: true
        roles:
        - { role: init_server, tags: init }
        vars:
        - { ansible_become_pass: "toor"}
        - { ansible_user: "root" }
        - { ansible_password: "toor" }
        - { ansible_port: 22 }
        - { host_user: testuser }
        - { host_password: testusernotsocomplicatedpassword }
        - { ssh_public_key: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK0wmN/Cr3JXqmLW7u+g9pTh+wyqDHpSQEIQczXkVx9q testuser }


For upgrade (once non privileged user and SSHD are configured):
      - hosts: all
        become: true
        roles:
        - { role: init_server, tags: init }
        vars:
        - { host_user: testuser }
        - { host_password: testusernotsocomplicatedpassword }
        - { ssh_public_key: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK0wmN/Cr3JXqmLW7u+g9pTh+wyqDHpSQEIQczXkVx9q testuser }


License
-------

AGPL-3

Author Information
------------------

Le Filament (https://le-filament.com)
