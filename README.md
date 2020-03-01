# My Ansible Playbooks
Distro-hopping can be a pain when you expect to have your favorite tools on a new distribution, but don't. These playbooks are an attempt to remedy that situation.

They add packages commonly used (by me), across different flavors of Linux.

The star of the show is **packages.yaml**, which invokes two different task lists using **`include_tasks`** depending on the OS of the node:
```yaml
---
- hosts: localhost
  become: yes
  tasks:
    - name: Red Hat
      include_tasks: redhat.yaml
      when: ansible_facts['os_family'] == "RedHat"
    - name: Ubuntu
      include_tasks: debian.yaml
      when: ansible_facts['os_family'] == "Debian"
```

**debian.yaml** is pretty straightforward, but **redhat.yaml** has some interesting block statements. For example, on Red Hat systems in order for `snapd` to work you have to manually create a symlink. The syntax of this playbook ensures that Red Hat systems create this symlink in an idempotent manner.
```yaml
- name: Install snapd
  block:
    - name: snapd
      dnf: name=snapd state=latest
    # On Red Hat systems, `snapd` requires a link to be manually created
    - name: Create link for snapd to work
      file: src=/var/lib/snapd/snap dest=/snap state=link
```