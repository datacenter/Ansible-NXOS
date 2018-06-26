This README contains some additional background information for managing Docker on the Nexus 9000 series switch.

Using Ansible to manage docker in the native bash environment is only supported on Nexus 9000 switches running image version `9.2(1)` or later.


# Table of contents 

1. [Create Ansible Hosts File](#hosts)
1. [Create Ansible Playbook](#playbook)


## <a name="hosts">Create Ansible Hosts File</a>
Create the Ansible hosts file that will allow you to manage the NXOS device using both the `vegas-shell(vsh)` and `bash -shell` environments.

Add the following to your `/etc/ansible/hosts` file but replace the host information with your specific host.
```yaml
# This Ansible inventory file defines connections to both the
# NXOS vegas-shell and the native linux bash-shell.
#

# -----------------------------------------------------#
# Define connections and variables for the vegas-shell #
# -----------------------------------------------------#

[nxos_vsh]
n9k.example.com

[nxos_vsh:vars]
ansible_connection=local

# -----------------------------------------------------------#
# Define connections and variables for the native bash-shell #
# -----------------------------------------------------------#

[nxos_bash]
n9k-bash ansible_host=n9k.example.com

[nxos_bash:vars]
ansible_connection=ssh
ansible_user=devops
ansible_ssh_pass=devopspassword
```

This yaml file is included in this repository and can be accessed here: [Hosts Inventory File](./host_inventory)

As you can see we specify different connection information under the `[nxos_vsh]` and `[nxos_bash]` tags.  This allows us to manage the same nxos device using two different user accounts.

The `admin` account is mapped to the vsh environment.
The `devops` account is mapped to the bash shell environment.

**NOTE:** We are using the `ansible_ssh_pass` parameter to allow easy access to the bash environment once the devops user is created by the playbook below.  The [`SSHPass`]( https://gist.github.com/arunoda/7790979) application must be installed on the Ansible server to make use of this.

This password in practice should be stored using [variables and vaults](http://docs.ansible.com/ansible/latest/playbooks_best_practices.html#best-practices-for-variables-and-vaults).

## <a name="playbook">Create Ansible Playbook To Manage Docker</a>
Create a playbook called `manage_docker_on_nxos.yaml`

Contents of `manage_docker_on_nxos.yaml`:

```yaml
---
# This playbook demonstrates how to manage docker on the Nexus 9000 switch.
#
# Using Ansible to manage docker in the native bash environment is only
# supported on Nexus 9000 switches running image version 9.2(1) or later.

# ----------------------------------------------------------------------#
# This first play will setup a user called 'devops' that can be used    #
# to access and manage the linux bash shell environment on NXOS.        #
# ----------------------------------------------------------------------#
- name: Setup bash shell user account.
  hosts: nxos_vsh
  gather_facts: no

  vars:
    network_connection:
      username: "admin"
      password: "password"
      transport: cli
      host: "{{ inventory_hostname }}"

  tasks:
    - name: Configure nxapi and devops user
      nxos_config:
        lines:
          - feature nxapi
          - nxapi http port 80
          - feature ssh
          - username devops password devopspassword role network-admin
          - username devops shelltype bash
        provider: "{{ network_connection }}"


# --------------------------------------------------------------------------#
# This second play will access the linux bash environment and manage docker #
# --------------------------------------------------------------------------#
- name: Manage linux bash environment on NXOS.
  hosts: nxos_bash
  gather_facts: no

  environment:
    http_proxy: http://proxy-example.com:80
    https_proxy: https://proxy-example.com:80

  tasks:
    - name: Start The Docker Service
      service:
        name: docker
        state: started
      become: yes
      become_method: sudo

    # Uncomment the following 3 tasks if a proxy is needed for
    # Network connectivity to the PyPY server.

    #- name: Configure http proxy info in /etc/sysconfig/docker
    #  lineinfile:
    #    path: /etc/sysconfig/docker
    #    regexp: 'export http_proxy'
    #    line: 'export http_proxy=http://proxy-example.com:80'
    #  become: yes
    #  become_method: sudo

    #- name: Configure https proxy info in /etc/sysconfig/docker
    #  lineinfile:
    #    path: /etc/sysconfig/docker
    #    regexp: 'export https_proxy'
    #    line: 'export https_proxy=http://proxy-example.com:80'
    #  become: yes
    #  become_method: sudo

    #- name: Re-start Docker Service Following Configuration of docker configuration file
    #  service:
    #    name: docker
    #    state: restarted
    #  become: yes
    #  become_method: sudo

    - name: Pip install docker-py package
      command: ip netns exec management pip install docker-py
      become: yes
      become_method: sudo

    - name: Pull Ubuntu Docker Image
      docker_image:
        name: ubuntu
      become: yes
      become_method: sudo

    - name: Create a container
      docker_container:
        name: ansible_test
        image: ubuntu
      become: yes
      become_method: sudo
```

This yaml file is included in this repository and can be accessed here: [Manage Docker Playbook](./manage_docker_on_nxos.yaml)
