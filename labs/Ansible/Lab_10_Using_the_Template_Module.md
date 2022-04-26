# Lab 10 - Working with Jinja Templates

In this lab you will be using the Ansible **template** module.  This module allows you to generate text-based files using Jinja templates.  

**In this lab you will:**
* Create a basic inventory
* Create a template for VLAN configurations
* Remove extra whitespace from the template output
* Extend the template to add functionality

**Additional Resources:**
* [Ansible Template Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
* [Jinja Template Design](https://jinja.palletsprojects.com/en/3.1.x/templates/)
* [Live Jinja Parser](https://j2live.ttl255.com/)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

### Task 1 - Build the base configs

##### Step 1 - Create the inventory file

`cd` into the `ansible_labs/lab10` folder and create `inventory.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab10 && touch inventory.yml
student@student-training:~/ansible_labs/lab10$ 
```

##### Step 2 - Build the inventory

On your own, attempt to build the inventory.

Your inventory should include the following groups and their associated hosts, vars, and children:

* **An explicit `all` group:**
    - Children:
        - orko_routers
        - orko_switches

* **An `orko_routers` group:**
    - Hosts:
        - orko-lab10-red-router
        - orko-lab10-yellow-router

* **An `orko_switches` group:**
    - Hosts:
        - orko-lab10-red-switch
        - orko-lab10-yellow-switch

**At this point, your inventory should look like the following:**

```yaml
all:
  children:

    orko_routers:
      hosts:
        orko-lab10-red-router:
        orko-lab10-yellow-router:

    orko_switches:
      hosts:
        orko-lab10-red-switch:
        orko-lab10-yellow-switch:

```

##### Step 3 - Create the host_vars and group_vars directories


**Create the following directories:**
  * group_vars
  * templates

```
student@student-training:~/ansible_labs/lab10$ mkdir group_vars templates
```

Your file structure should look like this while issuing the `tree` command:

```
student@student-training:~/ansible_labs/lab10$ tree
.
├── group_vars
├── inventory.yml
└── templates

2 directories, 1 files
```

##### Step 4 - Create the playbook


Create a playbook called `lab10.yml` with the following contents:

```yaml
---

  - name: GENERATE VLANS CONFIGS FOR SWITCHES
    hosts: orko_switches
    connection: local
    gather_facts: false

    tasks:

      - name: ENSURE DIRECTORY EXISTS
        file:
          path: "./configs/{{ inventory_hostname }}"
          state: directory
```

This playbook will create a directory for each device in the **orko_switches** group.  These directories are where we will store the outputted VLAN configuration files for each device.

##### Step 5 - Run the playbook

Execute the `lab10.yml` playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR ALL DEVICES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]


PLAY RECAP *********************************************************************
orko-lab10-red-switch                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-yellow-switch               : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 6 - Check directory structure

Issue a `tree` command to see the directories that have been created.

```
student@student-training:~/ansible_labs/lab10$ tree
.
├── configs
│   ├── orko-lab10-red-switch
│   └── orko-lab10-yellow-switch
├── group_vars
├── inventory.yml
├── templates
└── lab10.yml

7 directories, 3 files
```

This sets you up to store different types of configs for each device throughout the lab.

##### Step 6 - Define the `vlans` variable for the `all` group

Now the base project structure is setup and you are ready to build switch device configurations. You'll do this by creating the required YAML data and Jinja templates, and then render the two together to create configurations using the Ansible **template** module.

Create a YAML file called `group_vars/all.yml` and store a `vlans` variable in there like this:

```yaml
---

vlans:
  - id: 10
    name: data_vlan
  - id: 200
    name: voice_vlan
  - id: 1000
    name: management_vlan
```

> Remember, even though we didn't explicitly define the `all` group in our inventory, Ansible creates it by default and adds all hosts to it.  Therefore, we can create the groupvars file `all.yml` and it will take effect as intended.

##### Step 7 - Create the initial template

Add a Jinja template called `vlans-1.j2` in the `templates` directory:

```
{% for vlan in vlans %}
vlan {{ vlan.id }}
   name {{ vlan.name }}
{% endfor %}
```

**This template is going to loop through the `vlans` variable and use each vlan's *id* and *name***

##### Step 8 - Configure playbook to use the template

Add a new task to the `lab10.yml` playbook to generate the required VLAN configurations for each switch:

```yaml
      - name: GENERATE VLAN CONFIGS 1
        template:
          src: vlans-1.j2
          dest: "./configs/{{ inventory_hostname }}/vlans-1.conf"
```

> Note: Ansible will automatically look inside the `templates` sub-directory when using the `template` module, e.g. you didn't need to write `src: ./templates/vlans-1.j2`.

##### Step 9 - Run the playbook

Execute the playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]


TASK [GENERATE VLAN CONFIGS 1] *************************************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

PLAY RECAP *********************************************************************
orko-lab10-red-switch                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-yellow-switch               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 10 - Verify the files were created

Verify the configs were built with the `tree` command and inspect the file contents:

```
student@student-training:~/ansible_labs/lab10$$ tree
.
├── configs
│   ├── orko-lab10-red-switch
│   │   └── vlans-1.conf
│   └── orko-lab10-yellow-switch
│       └── vlans-1.conf
├── group_vars
│   └── all.yml
├── inventory.yml
├── templates
│   └── vlans-1.j2
└── lab10.yml

7 directories, 9 files
```

Display the contents of one of the `.conf` files to ensure it has the proper contents:

```conf
vlan 10
   name data_vlan
vlan 200
   name voice_vlan
vlan 1000
   name management_vlan
```

##### Step 11 - Edit the `vlans` variable

Remove one of the VLAN names from the `group_vars/all.yml` files so the updated `vlans` variable looks like this:

```yaml
vlans:
  - id: 10
    name: data_vlan
  - id: 200
  - id: 1000
    name: management_vlan
```

##### Step 12 - Run the playbook

Re-run the playbook.

```
student@student-training:~/ansible_labs/lab10$$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ***********************************************************************************************************************************

TASK [ENSURE DIRECTORY EXISTS] ***********************************************************************************************************************************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE VLAN CONFIGS 1] ***********************************************************************************************************************************************
fatal: [orko-lab10-red-switch]:    FAILED! => {"changed": false, "msg": "AnsibleUndefinedVariable: 'dict object' has no attribute 'name'"}
fatal: [orko-lab10-yellow-switch]: FAILED! => {"changed": false, "msg": "AnsibleUndefinedVariable: 'dict object' has no attribute 'name'"}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab10-red-switch                   : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
orko-lab10-yellow-switch                : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0 
```

You should see that playbook fails because there is a missing key. The existing template assumes the `name` is always going to be there. That needs to be accounted for.

##### Step 13 - Create a new Jinja template

Create a new template that will account for missing keys and call the template `vlans-2.j2`:

```jinja
{% for vlan in vlans %}
vlan {{ vlan.id }}
{% if vlan.get('name') %}
   name {{ vlan.name }}
{% endif %}
{% endfor %}
```

Take note of the conditional statement checking to see there is a `name` key with some value assigned to it.

##### Step 14 - Update the playbook

Update the playbook to account for the new template name. You'll need to update `src` and `dest` parameters in the `template` task.

```yaml

      - name: GENERATE VLAN CONFIGS 1
        template:
          src: vlans-2.j2
          dest: "./configs/{{ inventory_hostname }}/vlans-2.conf"
```

##### Step 15 - Run the playbook

Re-run the playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE VLAN CONFIGS 1] *************************************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

PLAY RECAP *********************************************************************
orko-lab10-yellow-switch                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-red-switch                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 16 - Check outputted files

Open at least one of the new configurations and ensure the correct configuration is there.

```
student@student-training:~/ansible_labs/lab10$ cat configs/orko-lab10-yellow-switch/vlans-2.conf
vlan 10
   name data_vlan
vlan 200
vlan 1000
   name management_vlan
```

### Task 2 - Manage Jinja Whitespace

In the last task, you used the following template as `vlans-2.j2`:

```jinja
{% for vlan in vlans %}
vlan {{ vlan.id }}
{% if vlan.get('name') %}
   name {{ vlan.name }}
{% endif %}
{% endfor %}
```

Jinja template syntax doesn't follow Python indentation practices, since it is a pure text templating engine.

This task explores using indentation in Jinja templates to make them more readable, while learning how to handle whitespace.

##### Step 1 - Add indentation to the template

If you were to follow block-based indentation for the template previously used, you would end up with the following:

```jinja
{% for vlan in vlans %}
vlan {{ vlan.id }}
  {% if vlan.get('name') %}
    name {{ vlan.name }}
  {% endif %}
{% endfor %}
```

**You should notice the subtle difference where the `if` statement is indented under the `for` block.**

Take the template above with the indented `if` statement and save it as `vlans-3a.j2`.

##### Step 2 - Update the playbook

**ADD** (do not replace) a task to the `lab10.yml` playbook to generate VLAN configs using this template.

```yaml
      - name: GENERATE CONFIGS USING INDENTED IF - 1
        template:
          src: vlans-3a.j2
          dest: ./configs/{{ inventory_hostname }}/vlans-3a.conf
```

##### Step 3 - Run the playbook

Execute the playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE VLAN CONFIGS 1] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 1] **********************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

PLAY RECAP *********************************************************************
orko-lab10-yellow-switch                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-red-switch                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 4 - Check the output

Verify the configs that are generated. You should see the following:

```
student@student-training:~/ansible_labs/lab10$ cat configs/orko-lab10-yellow-switch/vlans-3a.conf
vlan 10
      name data_vlan
  vlan 200
  vlan 1000
      name management_vlan

```

This has extra white space at the front of the line that should not really be there. We need to strip leading white space, since the added indentation in the template is causing the resulting config to be wrong.

##### Step 5 - Remove extra whitespace

Add the line `#jinja2: lstrip_blocks: True` to the first line of the previous template, but save it as `vlans-3b.j2` so we can easily compare and contrast the resulting configuration file.

This is what should be in the `vlans-3b.j2` file:

```jinja
#jinja2: lstrip_blocks: True

{% for vlan in vlans %}
vlan {{ vlan.id }}
  {% if vlan.get('name') %}
   name {{ vlan.name }}
  {% endif %}
{% endfor %}
```

##### Step 6 - Update the playbook

Add a task to the playbook to generate VLAN configs using this template.

```yaml
      - name: GENERATE CONFIGS USING INDENTED IF - 2
        template:
          src: vlans-3b.j2
          dest: ./configs/{{ inventory_hostname }}/vlans-3b.conf
```

##### Step 7 - Run the playbook

Execute the playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE VLAN CONFIGS 1] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 1] **********************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 2] **********************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

PLAY RECAP *********************************************************************
orko-lab10-yellow-switch                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-red-switch                     : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 8 - Compare the output

Open, and then compare and contrast, the configuration files created: `vlans-3a.conf` and `vlans-3b.conf`.

The Jinja rendering engine has now removed (stripped) the left side whitespace from any indented code blocks - in this case, the `if` block.

> Note: you can also control the `lstrip_blocks` parameter from the `template` module arguments in Ansible.


### Task 3 - Extend the VLANs YAML Data and Template

Building on previous work, this task will update the YAML data to allow the template designer to auto-generate commands to either configure or remove a VLAN from a switch.

##### Step 1 - Update the `vlans` variable for the `all` group

Update the `vlans` variable in `group_vars/all.yml` so it has one more VLAN. In addition, ensure all VLANs have a `state` parameter. This will be used to build the right configuration for that VLAN. When the `state` is set to `present`, the configs will get built to ensure the VLAN exists on the switch; when the `state` is set to `absent`, the config will get built to remove the VLAN from the switch.

```yaml
---

vlans:
  - id: 10
    name: data_vlan
    state: present
  - id: 200
    state: absent
  - id: 1000
    name: management_vlan
    state: present
  - id: 88
    state: absent
```

For example, the configs desired for this data should look like the following:

```
vlan 10
    name data_vlan
no vlan 200
vlan 1000
    name management_vlan
no vlan 88

```

##### Step 2 - Create a new Jinja template

Create a new template that checks for the `state` parameter using good indentation for readability. Save this template as `vlans-4a.j2`:

```jinja
{% for vlan in vlans %}
  {% if vlan.state == "present" %}
vlan {{ vlan.id }}
    {% if vlan.get('name') %}
    name {{ vlan.name }}
    {% endif %}
  {% elif vlan.state == "absent" %}
no vlan {{ vlan.id }}
  {% endif %}
{% endfor %}
```

##### Step 3 - Update the playbook

Add the following task to the `lab10.yml` playbook to use the new template:

```yaml
      - name: GENERATE CONFIGS - 4A
        template:
          src: vlans-4a.j2
          dest: ./configs/{{ inventory_hostname }}/vlans-4a.conf
```

##### Step 4 - Run the playbook

Execute the playbook.

```
student@student-training:~/ansible_labs/lab10$$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE VLAN CONFIGS 1] *************************************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 1] **********************************
changed: [orko-lab10-yellow-switch]
changed: [orko-lab10-red-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 2] **********************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

TASK [GENERATE CONFIGS - 4A] ***************************************************
changed: [orko-lab10-yellow-switch]
changed: [orko-lab10-red-switch]

PLAY RECAP *********************************************************************
orko-lab10-yellow-switch                  : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-red-switch                     : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 5 - Check file output

Open the new `vlans-4a.conf` files and look at the indentation:

```
student@student-training:~/ansible_labs/lab10$ cat configs/orko-lab10-yellow-switch/vlans-4a.conf
  vlan 10
        name data_vlan
        no vlan 200
    vlan 1000
        name management_vlan
        no vlan 88

```

We still need to remove the extra whitespace as we did before.

##### Step 6 - Remove extra whitespace

Add `#jinja2: lstrip_blocks: True` to the first line of the `vlans-4a.j2` template and then save it as `vlans-4b.j2`.

```
#jinja2: lstrip_blocks: True

{% for vlan in vlans %}
  {% if vlan.state == "present" %}
vlan {{ vlan.id }}
    {% if vlan.get('name') %}
   name {{ vlan.name }}
    {% endif %}
  {% elif vlan.state == "absent" %}
no vlan {{ vlan.id }}
  {% endif %}
{% endfor %}

```

##### Step 7 - Update the playbook

Add the following task to the `lab10.yml` playbook to use the new template:

```yaml
      - name: GENERATE CONFIGS - 4B
        template:
          src: vlans-4b.j2
          dest: ./configs/{{ inventory_hostname }}/vlans-4b.conf
```

##### Step 8 - Run the playbook

Execute the playbook.

```
student@student-training:~/ansible_labs/lab10$ ansible-playbook lab10.yml -i inventory.yml

PLAY [GENERATE VLANS CONFIGS FOR SWITCHES] ******************************************

TASK [ENSURE DIRECTORY EXISTS] *************************************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE VLAN CONFIGS 1] *************************************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 1] **********************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE CONFIGS USING INDENTED IF - 2] **********************************
ok: [orko-lab10-yellow-switch]
ok: [orko-lab10-red-switch]

TASK [GENERATE CONFIGS - 4A] ***************************************************
ok: [orko-lab10-red-switch]
ok: [orko-lab10-yellow-switch]

TASK [GENERATE CONFIGS - 4B] ***************************************************
changed: [orko-lab10-red-switch]
changed: [orko-lab10-yellow-switch]

PLAY RECAP *********************************************************************
orko-lab10-yellow-switch                  : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
orko-lab10-red-switch                     : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Step 9 - Verify file output

Verify the `vlans-4b.conf` configs created. This should be what you want to see:

```
student@student-training:~/ansible_labs/lab10$$ cat configs/orko-lab10-yellow-switch/vlans-4b.conf
vlan 10
    name dat_vlan
no vlan 200
vlan 1000
    name management_vlan
no vlan 88
```

##### Step 10 - Create a Jinja template without indentation

If you don't use the `#jinja2: lstrip_blocks: True` line at the top of your template, you can still have a well formatted config file if no indentation is used for the `if` and `for` blocks in this example.

Add one more template with the following:

```jinja
{% for vlan in vlans %}
{% if vlan.state == "present" %}
vlan {{ vlan.id }}
{% if vlan.get('name') %}
   name {{ vlan.name }}
{% endif %}
{% elif vlan.state == "absent" %}
no vlan {{ vlan.id }}
{% endif %}
{% endfor %}
```

This template should be saved as `vlans-4c.j2`.

##### Step 11 - Update the playbook

Add the required task in the playbook to use the template and execute it by running `ansible-playbook lab10.yml -i inventory.yml`.

```yaml
      - name: GENERATE CONFIGS - 4C
        template:
          src: vlans-4c.j2
          dest: ./configs/{{ inventory_hostname }}/vlans-4c.conf
```

##### Step 12 - Compare the output

If you compare the `4c` and `4b` configs there should be no differences.

```
student@student-training:~/ansible_labs/lab10$$ diff configs/orko-lab10-yellow-switch/vlans-4c.conf configs/orko-lab10-yellow-switch/vlans-4b.conf
student@student-training:~/ansible_labs/lab10$$
```