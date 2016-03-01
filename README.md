#### pkmasansible
#####1
sub-group. syntax:[groupname:children]
```
[gp1]
a.com ansible_ssh_host=192.168.12.1
[gp2]
b.com
[gp3]
c.com
[gp4:children]
gp1
gp2
[gp5:children]
gp3
[gp2:vars]
http_port=88
anible_ssh_port=314
[all:vars]
ansible_ssh_user=ke
```
`all` is a built-in group.  
other built-in vars:
```
ansible_ssh_pass
....
```

dynamic add host
```
- name:
  add_host:
    name: host.com
    groups: gp3
    ansible_ssh_host: 192.168.12.1
```

review syntax:
```
ansible-playbook -i invenpath -c connection -v(verbose) --limit groupname k.yml
```

```
- name: get var port
  debug:
    var: hostvars['hostname']['ansible_ssh_port']
```

other behavior keys:
```
connection
gather_facts
any_errors_fatal   #any failure may cause further tasks being attempted
max_fail_percentage  #% of hosts failed before whole operation halted
no_log
port
```
