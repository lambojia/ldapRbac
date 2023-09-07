ldapRbac
=========

installs & configures sssd on a host and writes files w/in /etc/sudoers.d/ to allow sudo access for defined binaries to defined groups.

  **Group Access Control** is achieved thru the use of ldap_access_filter w/in /etc/sssd/sssd.conf as an ALLOW list.

    [domain/example.com]
    ...
    _members of the ldap_admins group is granted sssd access_
    ldap_access_filter = (|(memberOf=cn=ldap_admins,ou=Groups,dc=example,dc=com))
    ...

  **Sudo'er Access Control** is achived thru the creation of two configuration files (**00-ldap-operator-roles** & **00-ldap-operator-policies**) w/in /etc/sudoers.d/.

    /etc/sudoers.d/00-ldap-operator-roles 
    ...
    _sample config for allowing sudo command access for cat , nano & vi to the OPERATOR role:_
    Cmnd_Alias     OPERATOR = /usr/bin/cat,/usr/bin/nano,/usr/bin/vi
    ...

    /etc/sudoers.d/00-ldap-operator-policies
    ...
    _allows ldap_admins to run sudo_
    %ldap_admins ALL=(ALL) NOPASSWD:OPERATOR
    ...

  **Custom Json Policy** - the ansible role expects a custom json policy that contains the ldap authentication parameters , access control defintions.

  **Policy syntax:**

    {
      "domain_info": {
        "domain": "example.com",
        "cert": (local path to the ldaps certificate),
        "key": (local path to the ldaps certificate key),
        "uri": "ldaps://ldap.google.com:636"
      },
      "roles": [
        {
          "name": "OPERATOR",
          "permissions": ["/usr/bin/cat", "/usr/bin/nano", "/usr/bin/vi"]
        },
        {
          "name": "NOC",
          "permissions": ["/usr/bin/systemctl","/usr/bin/apt"]
        },
        {
          "name": "ADMIN",
          "permissions": ["ALL"]
        }
      ],
      "groups": [
        {
          "name": "ldap_users",
          "roles": ["OPERATOR","NOC"]
        },
        {
          "name": "ldap_admins",
          "roles": ["ADMIN"]
        }
      ],
      "exclude" : []
    }

Requirements
------------

Tested on Ubuntu 22 

Role Variables
--------------

json_policy:          (required) (file_path) To be used for installing sssd auth and defining policies 

policy_exclude_group: (optional) (string []) Ldap groups to be excluded on specific hosts. Can be used to limit access for specific ldap groups.

Dependencies
------------

N/A

Example Playbook
----------------

Deploy same policies accross the target hosts

    - hosts: devenv
      become: yes
      vars
        - json_policy: sample-policy.json
      roles:
        - ldapRbac

Deploy different policies accross the target hosts


    - hosts: devenv
      become: yes
      roles:
        - ldapRbac

inventory.yml

    devenv:
      hosts:
        host01:
          ...
          json_policy: admin-policy.json
        host02:
          ...
          json_policy: user-policy.json

Deploy same policies accross the target hosts

    - hosts: devenv
      become: yes
      vars:
        - json_policy: sample-policy.json
      roles:
        - ldapRbac

inventory.yml - the ldap_users group is removed from the ldap_filter for host01

    devenv:
      hosts:
        host01:
          ...
          policy_exclude_group: ["ldap_users"]
  

License
-------

BSD

Author Information
------------------

N/A
