ansible\_ssh\_server\_static\_keys
==================================

Manages static SSH keys for servers, keeping the ssh keys out of the standard
`host_vars` directory, while keeping them secure.

Requirements
------------

Currently this has only been tested on Ubuntu (Trusty). Please raise a pull
request or GitHub issue if you'd like support for other OSes. Adding a new OS
should be pretty easy to do.

By default, this code expects a ECDSA SSH key, which is only available in later
releases of OpenSSH.

Example Playbook
----------------

In site.yml (for example)

    - name: Server name over here
      vars_files:
        # This is where your ssh keys are - in a file in the base Ansible
        # directory like 'ssh_server_static_keys/servername.example.com.yml'
        - "ssh_server_static_keys/{{ inventory_hostname }}.yml"
      roles:
        - ssh_server_static_keys
        - some_other_role
        - another_role

Role Variables
--------------

Since the private keys are confidential information, we store want to store
them in encrypted form.

Unfortunately Ansible doesn't yet support encrypted files - only encrypted
variable (yaml) files. So we need to store the ssh keys into variables.

Each host that uses this role is expected to have six variables - one for each
type of key:

* `ssh_host_rsa_key` - The RSA private key
* `ssh_host_rsa_key_pub` - the public key associated with `ssh_host_rsa_key`
* `ssh_host_dsa_key` - The DSA private key
* `ssh_host_dsa_key_pub` - the public key associated with `ssh_host_dsa_key`
* `ssh_host_ecdsa_key` - The ECDSA private key
* `ssh_host_ecdsa_key_pub` - the public key associated with `ssh_host_ecdsa_key`

We don't want to mix the encrypted key variables in with standard Ansible
variables - otherwise it's difficult to view the standard variable changes via
version control or with your normal editor. The SSH keys are thus stored in
separate 'vault' files.

The best way to do this is to create a new directory, `ssh_server_static_keys`,
and store the keys in one file per role. If you are using the recommended
directory layout from http://docs.ansible.com/playbooks_best_practices.html
your new directory structure will look as follows:

    ansible/
      group_vars/
      host_files/
      host_vars/
      roles/
      site.yml
      ssh_server_static_keys/              <--- NOTE

Configuration Overview
----------------------

For the purposes of this example, assume we are configuring a machine called
`servername.example.com`

Run this to create the yaml file that will contain your encrypted SSH keys:

    ansible-vault create ssh_server_static_keys/servername.example.yml

Add the following blocks to the file, substituting in the SSH keys you want,
and then save and exit.

Note that the '|' at the end of the key name is important, since it allows the
value to span multiple lines.

    ssh_host_rsa_key: |
      -----BEGIN RSA PRIVATE KEY-----
      ... Private key over here ...
      -----END RSA PRIVATE KEY-----
    ssh_host_rsa_key_pub: |
      ssh-rsa  ...

    ssh_host_dsa_key: |
      -----BEGIN DSA PRIVATE KEY-----
      ... Private key here ...
      -----END DSA PRIVATE KEY-----
    ssh_host_dsa_key_pub: |
      ssh-dss ...

    ssh_host_ecdsa_key: |
      -----BEGIN EC PRIVATE KEY-----
      ... Private key here ...
      -----END EC PRIVATE KEY-----

    ssh_host_ecdsa_key_pub: |
      ecdsa-sha2-nistp256 ...

License
-------

GPLv2

Author Information
------------------

http://www.unboxedconsulting.com/


Contributing
------------

Pull requests are welcome - please create them on GitHub.