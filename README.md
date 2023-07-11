### overview

An ansible playbook that populates a host group containing the IP addresses of available nodes in a proxmox cluster.

---

### prerequisites

Ideally, you should have key-based password-less SSH login set up. Even more ideal that you have a low-power node like a thin client as part of your PVE cluster that is trivial to keep powered on with a UPS.

PVE is an environment variable pointing to the IP address or hostname of that low-power node. It should be defined in the .zshrc or .bashrc of the terminal where ansible is installed:

    export PVE=192.168.254.1

jq should be installed/present on that proxmox node.

---

### strategy

Parse the JSON formatted file at /etc/pve/.members and strip it down to just the IP addresses of nodes that are currently online and then return that list to ansible as a variable which will then populate a host group using ansible.builtin.add_host

---

### code

    - name: Assemble proxmox_nodes Inventory Group
      hosts: localhost
      connection: local
      gather_facts: false

      tasks:
      - name: Gather online proxmox nodes
        register: online_members
        ansible.builtin.shell: "ssh root@{{ lookup('ansible.builtin.env', 'PVE') }} cat /etc/pve/.members | jq -r '.nodelist[] | select(.online == 1) | .ip'"
        changed_when: false

      - name: Populate proxmox_nodes group
        when: online_members.stdout | length > 0
        ansible.builtin.add_host:
          name: "{{ item }}"
          groups: proxmox_nodes
        loop: "{{ online_members.stdout_lines }}"
        changed_when: false

---

### extending

Import this playbook in your main playbook for proxmox related tasks:

    - import_playbook: ansible-inventory_proxmox.yml
