[block]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html
[when]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html
[include_tasks]: https://docs.ansible.com/ansible/latest/modules/include_tasks_module.html

# My Ansible Playbooks
Distro-hopping can be a pain when you expect to have your favorite tools on a new distribution, but don't. These playbooks are an attempt to remedy that situation.

They add packages commonly used (by me), across different flavors of Linux.

The star of the show is[ **packages.yaml**](packages.yaml), which invokes two different task lists using [**`include_tasks`**][include_tasks] depending on the OS of the node:
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

[**debian.yaml**](debian.yaml) is pretty straightforward, but [**redhat.yaml**](redhat.yaml) has some interesting **[`block`][block]** and **[`when`][when]** statements. 

For example, on Red Hat systems in order for `snapd` to work you have to manually create a symlink. The syntax of this playbook ensures that Red Hat systems create this symlink in an idempotent manner.
```yaml
- name: Install snapd
  block:
    - name: snapd
      dnf: name=snapd state=latest
    # On Red Hat systems, `snapd` requires a link to be manually created
    - name: Create link for snapd to work
      file: src=/var/lib/snapd/snap dest=/snap state=link
```
Also the following ensures that [**GNOME Tweaks**](https://github.com/GNOME/gnome-tweaks) is installed, but only on GNOME desktops (based on the value of the `DESKTOP_SESSION` environment variable):
```yaml
- name: GNOME tools
  when: lookup('env','DESKTOP_SESSION') == "gnome"
  dnf: name=gnome-tweak-tool state=latest
```