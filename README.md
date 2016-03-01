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
#####2
```
ansible-vault create vault-password-file file encrypt.yml
```
execute a encrypted playbook
```
ansible-playbook -i hostname --valut-password-file file encrypt.yml -vv
```

if main playbook includes encrypted file, use
```
--ask-vault-pass
```

#####3 Jinja2
macros
```
{% macro test(var_a='string') -%}
{{test.argumants}}
{{test.defaults}}
{%- endmacro %}

{{test()}}
```
will return ('var_a',)('string',)
```
{% macro test() -%}
{{kwargs}}
{{test.catch_kwargs}}
{%- endmacro %}

{{test(key='value')}}
```
will return {'key':'value'}(True)
```
{% macro test() -%}
{{varargs}}
{{test.catch_varargs}}
{%- endmacro %}

{{test('value')}}
```
will return ('value',)(True)  
  
caller  
```
{% macro test() -%}
1
{{caller()}}
{%- endmacro %}

{% call test() %}
2
{% endcall %}
```
will return 1 2  

pass arguments in call block:
```
{% macro test(group, hosts) -%}
{% for host in hosts %}
{{ host }} {{ caller(host) }}
{%- endfor %}
{%- endmacro %}
{% call(host) test('db', ['1', '2','3']) %}
ssh_host_name={{ host }}.example.name
{% endcall %}
```

filters:  
```
{{var | replace('a','b')|lower| default('default')|count}}
```
some others:
```
basename
dirname
shuffle
random
```
#####4
using failed_when
```
- name:
  command: /sbin/icsciadm -m sessions
  register: sessions
  failed_when sessions.rc not in (1,3)
```

rc: return code  

judge a git branch exist, delete it if exists, skip otherwise.  
1st version
```
- name:
  command: git branch
  register: branches
- name:
  command: git branch -D feature
  args:
    chdir: /app
  when: branches.stdout | search('feature')
```

2nd version:
```
- name:
  command: git branch -D feature
  args:
    chdir: /app
  register: outresult
  failed_when: outresult.rc!=0 and not outresult.stderr |search ('branch.*not found')
```

changed_when: mark changed=1.  


creates and removes argument in command family modules.(command,shell,script)
1st version
```
- name: exist?
  stat:
    path: /srv
  register: srv

- name: run script
  script: files/script --initialize /srv
  when: not srv.stat.exists
```

using `creates`
```
- name: run script
  script: files/script creates=/srv
```


#####5 roles 
file.yaml
```
---
- name: create leading path
  file:
    path: "{{ path }}"
    state: directory

- name: touch the file
  file:
    path: "{{ path + '/' + file }}"
    state: touch
```
this is untuitive, to use this file. we first include this file, then pass along the params.  
master.yaml
```
---
- name: touch files
  hosts: localhost
  gather_facts: false

  tasks:
    - include: files.yaml
      path: /tmp/foo
      file: a.txt

    - include: files.yaml
      path: /tmp/foo
      file: b.txt
```

with_dict directive usage:
```
---
- name: create leading path
  file:
    path: "{{ item.value.path }}"
    state: directory
  with_dict: files

- name: touch the file
  file:
    path: "{{ item.value.path + '/' + item.key }}"
    state: touch
  with_dict: files
```

file module has two directives: path and state (touch,directory)  
`files` is a user-defined dict. item is built-in variable. one key may has multiple values.  
Hence:
```
---
- name: touch files
  hosts: localhost
  gather_facts: false

  tasks:
    - include: files.yaml
      vars:
        files:
          a.txt:
            path: /tmp/foo
          b.txt:
            path: /tmp/foo
```

--extra-vars = -e  
-e can also pass filename. Ex   
```
ansible-playbook file.yaml -e @var.yaml
```

`include_vars`: dynamically get arg from filenames. use with `with_first_found`  
Ex:
```
---
- name:
  hosts: localhost
  gather_facts: true

  tasks:
    - name: get var from first found file
      include_vars: "{{ item }}"
      with_first_found:
        - "{{ ansible_distribution }}.yaml"
        -  variables.yaml

    - name:
      debug:
        msg: "{{ name }}"
```

using roles.(defaults,files,handlers,templates,vars,tasks,meta,library)  

meta/main.yaml
```
---
dependencies:
  - role: a
  - role: b
```
This role depends on role a and b.  

More complex: assign vars with roles
```
---
dependencies:
  - role: common
    a: True
    b: False
  - role: apache
    complex_var:
      key1: value1
      key2: value2
    short_list:
      - 8080
      - 80
```

in main.yaml we ca assign var to override var is child files.  
main.yaml without override
```
---
- hosts: localhost
  gather_facts: false

  roles:
    - role: simple
```
override some vars:
```
---
- hosts: localhost
  gather_facts: false

  roles:
    - role: simple
      a: newval
```

using multiple roles:
```
---
- hosts: localhost
  gather_facts: false

  roles:
    - role: simple
      a: newval
    - role: second_role
      othervar: value
    - role: third_role
    - role: another_role
```

ansible-galaxy usage
```
ansible-galaxy install -p roles/ user.rolename
```

other commands
```
ansible-galaxy list -p roles/
ansible-galaxy info -p roles/ user.rolename
```

create a new role (w/ 7 folders)
```
ansible-galaxy init -p roles/ newrole
```
rename downloaded rolename
```
ansible-galaxy install -p /opt/ansible/roles rengokantai@git...git,version(canomit),newname
```
To install all the roles within a file, use the --roles-file (-r) option.
```
ansible-galaxy install --roles-file y.yaml
```
