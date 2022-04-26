# Lab 07 - Jinja2 Filters

This lab introduces and explores several common Jinja filters.  Jinja filters provide ways to manipulate and work with data in a clean and consumable way.  Remember you already saw one filter in the Compliance lab when we converted a string to an integer.  Now we'll take a look at several more filters.

**In this lab you will:**
* Create a playbook that debugs variables and modifies the output with filters.

**Additional Resources:**
* [Using filters to manipulate data](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)
* [Filter field guide](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_filters.html)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Ansible Debug Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)

### Task 1 - Using Jinja2 Filters

The Jinja2 library ships with many filters already, but in addition to those Ansible also ships with it's own that are only available in Ansible. It is even possible to add custom filters with knowing just a little bit of Python.


##### Step 1 - Create the playbook

`cd` into the `ansible_labs/lab07` folder and create `lab07.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab07 && touch lab07.yml
student@student-training:~/ansible_labs/lab07$ 
```


Open this file with a text editor and input the play definition as follows:

```yaml

---

- name: TEST JINJA FILTERS
  hosts: localhost
  connection: local
  gather_facts: no
      
```

>Note: Since we are not targeting any remote devices lets use `localhost` as the value for `hosts`.

##### Step 2 - Debug a variable with a filter

Add a `vars` key and our first variable `hostname` and task to debug and apply the `upper` filter. This filter will convert a value to uppercase as we will see after running the playbook.


```yaml

---

  - name: TEST JINJA FILTERS
    hosts: localhost
    connection: local
    gather_facts: no

    vars:
      hostname: router-1
      
    tasks:
    
    
      - name: UPPERCASE HOSTNAME
        debug:
          var: hostname | upper
          
      - name: UPPERCASE HOSTNAME IN A MESSAGE
        debug:
          msg: "The hostname is {{ hostname | upper }}"
            
```

##### Step 3 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

student@student-training:~/lab07/playbooks/$ ansible-playbook lab07.yml

PLAY [TEST JINJA FILTERS] ***************************************************************************************

TASK [UPPERCASE HOSTNAME] ***************************************************************************************
ok: [localhost] => {
    "hostname | upper": "ROUTER-1"
}

TASK [UPPERCASE HOSTNAME IN A MESSAGE] **************************************************************************
ok: [localhost] => {
    "msg": "The hostname is ROUTER-1"
}

PLAY RECAP ******************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0

student@student-training:~/ansible_labs/lab07$

```


You can see already the filter simplify manipulated a string by "upper-casing" it here.

##### Step 5 - Debug the length of a variable


Lets add another variable under `vars` called `vlans` and make a list of vlans and it's name for each.  We will represent VLANs as a list of dictionaries. 

Also add a new `debug` task using the `length` filter which will return the number of items of a sequence (list) or mapping (dictionary). 

```yaml

---

  - name: TEST JINJA FILTERS
    hosts: localhost
    connection: local
    gather_facts: no

    vars:
      hostname: router-1

      vlans:
        - id: 10
          name: data_vlan
        - id: 200
          name: voice_vlan
        - id: 1000
          name: mgmt_vlan
      
    tasks:
    
      - name: UPPERCASE HOSTNAME
        debug:
          var: hostname | upper
          
      - name: UPPERCASE HOSTNAME IN A MESSAGE
        debug:
          msg: "The hostname is {{ hostname | upper }}"

      - name: VERIFY LENGTH OF LIST
        debug:
          msg: "There are {{ vlans | length }} VLANs on the switch."

```

Many of these filters make sense after you see the result.  Again, `length` is just returning a value of how many elements are in the list.

##### Step 6 - Run the playbook

After running the playbook, you should see the following relevent output:


```commandline

TASK [UPPERCASE HOSTNAME] ***************************************************************************************
ok: [localhost] => {
    "hostname | upper": "ROUTER-1"
}

TASK [UPPERCASE HOSTNAME IN A MESSAGE] **************************************************************************
ok: [localhost] => {
    "msg": "The hostname is ROUTER-1"
}

TASK [VERIFY LENGTH OF LIST] **************************************************************************************
ok: [localhost] => {
    "msg": "There are 3 VLANs on the switch."
}


```

##### Step 7  - Use the 'selectattr' filter

Add a new variable under `vars` called `interfaces_config` and a new `debug` task to use the `selectattr` and `list` filters chained together. 

The `selectattr` filter reads through a sequence of objects by applying a test to the specified attribute of each object, and only selecting the objects with the test succeeding. If no test is specified, the attribute’s value will be evaluated as a boolean.

Our test is checking to see which interfaces have a `status` key that is equal to `true`:

```yaml

---

  - name: TEST JINJA FILTERS
    hosts: localhost
    connection: local
    gather_facts: no

    vars:
      hostname: router-1

      vlans:
        - id: 10
          name: data_vlan
        - id: 200
          name: voice_vlan
        - id: 1000
          name: mgmt_vlan
          
      interfaces_config:
        - name: GigabitEthernet1
          speed: 1000
          duplex: full
          status: true
        - name: GigabitEthernet2
          speed: 1000
          duplex: full
          status: true
        - name: GigabitEthernet3
          speed: 1000
          duplex: full
          status: false
      
    tasks:
    
      - name: UPPERCASE HOSTNAME
        debug:
          var: hostname | upper
          
      - name: UPPERCASE HOSTNAME IN A MESSAGE
        debug:
          msg: "The hostname is {{ hostname | upper }}"

      - name: VERIFY LENGTH OF LIST
        debug:
          msg: "There are {{ vlans | length }} VLANs on the switch."
          
      - name: GET ELEMENTS THAT HAVE A TRUE VALUE FOR STATUS AS A LIST
        debug:
          var: interfaces_config | selectattr("status") | list

```

> Note: technically, the `selectattr` filter returns an advanced Python object, so we need to use `| list` to convert that object to a list.

> This is also show that it is possible _chain_ filters together too.


##### Step 8 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [GET ELEMENTS THAT HAVE A TRUE VALUE FOR STATUS AS A LIST] ***************************************************
ok: [localhost] => {
    "interfaces_config | selectattr(\"status\") | list": [
        {
            "duplex": "full",
            "name": "GigabitEthernet1",
            "speed": 1000,
            "status": true
        },
        {
            "duplex": "full",
            "name": "GigabitEthernet2",
            "speed": 1000,
            "status": true
        }
    ]
}

```

##### Step 9 - Use the 'map' filter

Add two more `debug` tasks to the playbook using the `map` filter that is applied on a sequence of objects or looks up an attribute. This filter can be useful when dealing with lists of objects but you are only really interested in a certain value of it.

The basic usage is mapping on an attribute, e.g. key. Imagine you have a list of  `interfaces` or `vlans` but you are only interested in a list of __names__ of the interfaces or vlans.



```yaml

      - name: RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES
        debug:
          var: interfaces_config | map(attribute="name") | list

      - name: RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES
        debug:
          var: vlans | map(attribute="name") | list

```

##### Step 10 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES] *******************************************************
ok: [localhost] => {
    "interfaces_config | map(attribute=\"name\") | list": [
        "GigabitEthernet1",
        "GigabitEthernet2",
        "GigabitEthernet3"
    ]
}

TASK [RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES] ********************************************************ok: [localhost] => {
    "vlans | map(attribute=\"name\") | list": [
        "web_vlan",
        "app_vlan",
        "db_vlan"
    ]
}


```

##### Step 11 - Chain filters together


Add another task to the playbook but this time we are going to chain together `selectattr`, `map` and `list` filters. This is going to allow us to just return a list of interface names that are up.

Chaining these filters together will allow `selectattr` to just target the status of the interfaces that are `true` and using the `map` filter will target the names of the interfaces then `list` filter will return a list.  


```yaml

      - name: RETURN JUST LIST OF INTERFACE NAMES THAT ARE UP (TRUE)
        debug:
          var: interfaces_config | selectattr("status") | map(attribute="name") | list

```

##### Step 12 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [RETURN JUST LIST OF INTERFACE NAMES THAT ARE UP (TRUE)] ***************************************************
ok: [localhost] => {
    "interfaces_config | selectattr(\"status\") | map(attribute=\"name\") | list": [
        "GigabitEthernet1",
        "GigabitEthernet2"
    ]
}

```

##### Step 13 - Use the 'ternary' filter

Lets try out one more test. Add another variable and call it `interface_state` and one more `debug` task using the `ternary` filter. 

This filter allows us to apply logic to a variable without having to use `python` syntax. Both tasks will return the same output based on the variable value but are written differently.

```yaml


vars:

  interface_state: false
  
  #....omitted
  
tasks:
  
  #....omitted

      - name: CONVERT BOOLEAN T/F TO SOMETHING MORE CONTEXTUAL FOR NETWORKING
        debug:
          var: interface_state | ternary("up", "down")

      - name: CONVERT BOOLEAN T/F USING PROGRAMING LOGIC
        debug:
          msg: "{{ 'up' if interface_state else 'down' }}"

```

##### Step 14 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [CONVERT BOOLEAN T/F TO SOMETHING MORE CONTEXTUAL FOR NETWORKING] *******************************************************************
ok: [localhost] => {
    "interface_state | ternary(\"up\", \"down\")": "down"
}

TASK [CONVERT BOOLEAN T/F USING PROGRAMING LOGIC] ****************************************************
ok: [localhost] => {
    "msg": "down"
}


```

Give it another try, except this time change `interface_state` to **true**


Check the final playbook

```yaml

---

  - name: TEST JINJA FILTERS
    hosts: localhost
    connection: local
    gather_facts: no

    vars:
      interface_state: false

      hostname: router-1

      vlans:
        - id: 10
          name: data_vlan
        - id: 200
          name: voice_vlan
        - id: 1000
          name: mgmt_vlan
          
      interfaces_config:
        - name: GigabitEthernet1
          speed: 1000
          duplex: full
          status: true
        - name: GigabitEthernet2
          speed: 1000
          duplex: full
          status: true
        - name: GigabitEthernet3
          speed: 1000
          duplex: full
          status: false
          
    tasks:
    
      - name: UPPERCASE HOSTNAME
        debug:
          var: hostname | upper
          
      - name: UPPERCASE HOSTNAME IN A MESSAGE
        debug:
          msg: "The hostname is {{ hostname | upper }}"

      - name: VERIFY LENGTH OF LIST
        debug:
          msg: "There are {{ vlans | length }} VLANs on the switch."
        
      - name: GET ELEMENTS THAT HAVE A TRUE VALUE FOR STATUS AS A LIST
        debug:
          var: interfaces_config | selectattr("status") | list

      - name: RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES
        debug:
          var: interfaces_config | map(attribute="name") | list

      - name: RETURN LIST OF ALL NAME KEYS IN THE INTERFACES_CONFIG LIST OF DICTIONARIES
        debug:
          var: vlans | map(attribute="name") | list

      - name: RETURN JUST LIST OF INTERFACE NAMES THAT ARE UP (TRUE)
        debug:
          var: interfaces_config | selectattr("status") | map(attribute="name") | list

      - name: CONVERT BOOLEAN T/F TO SOMETHING MORE CONTEXTUAL FOR NETWORKING
        debug:
          var: interface_state | ternary("up", "down")

      - name: CONVERT BOOLEAN T/F USING PROGRAMING LOGIC
        debug:
          msg: "{{ 'up' if interface_state else 'down' }}"
          
```


