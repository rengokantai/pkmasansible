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
