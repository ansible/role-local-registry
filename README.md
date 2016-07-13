Create Local Registry 
=====================

Create a local Docker registry suitable for testing and development. The resulting registry includes self-signed certificates and 
basic username/password authentication.

Requirements
------------

- Ansible 2.1 with the lastest changes to the [docker_container](https://github.com/ansible/ansible-modules-core/blob/devel/cloud/docker/docker_container.py) module. 
  If you're not running Ansible from source, grab a copy of the latest docker_container.py and place it in a [library directory](http://docs.ansible.com/ansible/intro_configuration.html#library).
- Docker daemon

Role Variables
--------------
registry_name
> Name of the container running the registry service. Defaults to *registry*.

registry_auth_path
> Path to the directory containing the password file. Defaults to *files/auth*. 

registry_auth_file
> Name of the password file. Defaults to *htpasswd*.

registry_cert_path
> Path to the directory containing the domain certifiate and key files. Defaults to *files/certs*. 

registry_cert_file
> Name of the domain certificate file. Defaults to *domain.crt*.

registry_key_file
> Name of the domain key file. Defaults to *domain.key*.

registry_port
> Map the registry port to this host port. Defaults to *5000*.

registry_users
> List of users. Each user is an object containing username and password keys. See [defaults/main.yml] for an example.

registry_host:
> The hostname or IP address. Defaults to *localhost*.

Example Playbook
----------------

Below is an example playbook that stands up a registry and then makes some assertions. It tests that the registry is running and users can actuall authenticate.

```
#
# example_playbook.yml
#
- name: Test role-local-registry 
  hosts: localhost
  connection: local
  gather_facts: no
  roles:
    - role: role-local-registry

- name: Test the registry
  hosts: localhost
  connection: local
  gather_facts: no
  tasks:

    - command: "{% raw %}docker inspect --format='{{ .State.Running }}' registry{% endraw %}"
      register: container_state

    - name: Should be running
      assert:
        that:
          - container_state.stdout == 'true' 
          - 
    - name: Authenticate each user with the registry
      docker_login:
        registry_url: "{{ registry_host }}:{{ registry_port }}"
        username: "{{ item.username }}"
        password: "{{ item.password }}"
      with_items: "{{ registry_users }}"
```

Here's a sample vars file, defining the set of users to create and the host IP and port:

```
---
registry_users:
  - username: user0
    password: Apassword! 
  - username: user1 
    password: Bpassword! 
registry_host: 192.168.99.100 
registry_port: 5000
```

And finally, to execute the above:

```
$ ansible-playbook example_playbook.yml -e"@vars.yml"
```

License
-------

MIT

Authors
-------

[@chouseknecht](https://github.com/chouseknecht)
