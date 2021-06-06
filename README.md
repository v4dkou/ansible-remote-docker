remote-docker for Ansible
=========

⚠️ WIP! Does not work reliably

`raw: echo "Success!"` does not always respond with the string "Success!".

This in turn makes other tasks fail with
```
failed to resolve remote temporary directory from ansible-tmp-<...>: `( umask 77 && mkdir -p \"` echo / `\"&& mkdir \"` echo /ansible-tmp-<...> `\" && echo ansible-tmp-<...>=\"` echo /ansible-tmp-<...> `\" )` returned empty string
```

Works by 
1. forwarding the remote Docker socket to a local file with an SSH tunnel
2. starting a `python:3.8-slim-buster` container on remote
3. using [community.docker.docker_api](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_api_connection.html) with DOCKER_HOST set to connect to it with Ansible 

Intended to be used to run tasks inside Docker networks (i.e. to create users in a DBMS that's not accessible outside of a Docker network)

Dependencies
------------

`ansible-galaxy collection install community.docker`

Example Playbook
----------------

    - hosts: servers
      vars:
        target_host: root@<YOUR SERVER IP>
        local_tunnel_path: /tmp/docker.sock
        ansible_docker_docker_host: "unix://{{local_tunnel_path}}"

    - name: Start remote docker connection
      include_role:
        name: "remote-docker"
        tasks_from: start

    # Run your stuff here
    - name: run command in container
      delegate_to: "ansible-runner"
      vars:
        ansible_python_interpreter: /bin/python3
      raw: echo "Success!!"
      register: hello

    - debug: msg="{{ hello.stdout }}"
    # ...

    - name: Stop remote docker connection
      include_role:
        name: "remote-docker"
        tasks_from: stop
License
-------

MIT


