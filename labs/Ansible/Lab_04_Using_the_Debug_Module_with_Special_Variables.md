# Lab 04 - Using the debug module


**In this lab you will:**
* Debug **host-level** variables
* Debug **group-level** variables
* Debug a message
* Debug **special** variables

This lab highlights the use of the `debug` module.  It offers you the ability to "print" variables to the terminal, which is often very helpful for verifying what a variable is set to.  As you can see from lab 02, you can see that variables can be defined in many locations in the inventory.yml file, e.g. same variable for different groups.

Let's review a few ways the `debug` module helps with troubleshooting.

**Additional Resources:**
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Ansible Debug Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)


### Task 1 - Debugging Variables

##### Step 1 - Create the inventory file

`cd` into the `ansible_labs/lab04` folder and create `inventory.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab04 && touch inventory.yml
student@student-training:~/ansible_labs/lab04$ 
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
        - orko-lab04-red-router
        - orko-lab04-yellow-router

    - Vars:
        - mgmt_int: Loopback0

* **An `orko_switches` group:**
    - Hosts:
        - orko-lab04-red-switch
        - orko-lab04-yellow-switch

    - Vars:
        - mgmt_int: Vlan1000

**At this point, your inventory should look like the following:**

```yaml
all:
  children:

    orko_routers:
      hosts:
        orko-lab04-red-router:
        orko-lab04-yellow-router:
      vars:
        mgmt_int: Loopback0

    orko_switches:
      hosts:
        orko-lab04-red-switch:
        orko-lab04-yellow-switch:
      vars:
        mgmt_int: Vlan1000

```

##### Step 3 - Create the playbook

Create a playbook file called `lab04.yml`.

```
student@student-training:~/ansible_labs/lab04$ touch lab04.yml
student@student-training:~/ansible_labs/lab04$
```

Open the file in your text editor.

The playbook will consist of a single play and a single task.


Use the following for the starting point of the playbook.  This will execute for all devices in the inventory.

The task will simply print the variable `mgmt_int` for each device in the group.

```yaml

---

  - name: USING THE DEBUG MODULE
    hosts: all
    connection: local
    gather_facts: no


    tasks:
      - name: DEBUG AND PRINT TO TERMINAL
        debug: 
          var: mgmt_int
```

Remember the `mgmt_int` variable is simply a variable we created in the inventory for each group of devices and now we are going to print it for all devices in the **all** group.

##### Step 4 - Run the playbook

After running the playbook, you should see the following relevent output:

```
student@student-training:~/ansible_labs/lab04$ ansible-playbook lab04.yml -i inventory.yml

PLAY [USING THE DEBUG MODULE] ************************************************************************************************************************************************************************************************************************************************************************************************************************************************

TASK [DEBUG AND PRINT TO TERMINAL] *******************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab04-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
orko-lab04-red-router      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab04-red-switch      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab04-yellow-router   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab04-yellow-switch   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  

```

Note, this just printed the variable to the terminal.


### Task 2 - Adding & Printing More Group Variables

##### Step 1 - Add the `orko_riverbeds` group to the inventory

In your inventory, add the `orko_riverbeds` group:

* Hosts:
    - riverbed-1
    - riverbed-2

Your inventory should now look like the following:

```yaml
all:
  children:

    orko_routers:
      hosts:
        orko-lab04-red-router:
        orko-lab04-yellow-router:
      vars:
        mgmt_int: Loopback0

    orko_switches:
      hosts:
        orko-lab04-red-switch:
        orko-lab04-yellow-switch:
      vars:
        mgmt_int: Vlan1000

    orko_riverbeds:
      hosts:
        riverbed-1:
        riverbed-2:
```

##### Step 2 - Run the playbook

After running the playbook, you should see the following relevent output:

```
TASK [DEBUG AND PRINT TO TERMINAL] *******************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab04-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [riverbed-1] => {
    "mgmt_int": "VARIABLE IS NOT DEFINED!"
}
ok: [riverbed-2] => {
    "mgmt_int": "VARIABLE IS NOT DEFINED!"
}

```

Notice that since we did not define the `mgmt_int` variable for the `riverbed` group, we get this Ansible error in the output for the two riverbed devices:

**"mgmt_int": "VARIABLE IS NOT DEFINED!"**

##### Step 3 - Set `mgmt_int` default for all devices

In your inventory, add the `mgmt_int` variable to the `all` group.  Set the value to `unknown`:

```yaml
all:
  vars:
    mgmt_int: unknown

  children:

    orko_routers:
      hosts:
        orko-lab04-red-router:
        orko-lab04-yellow-router:
      vars:
        mgmt_int: Loopback0

    orko_switches:
      hosts:
        orko-lab04-red-switch:
        orko-lab04-yellow-switch:
      vars:
        mgmt_int: Vlan1000

    orko_riverbeds:
      hosts:
        riverbed-1:
        riverbed-2:
```

##### Step 4 - Run the playbook

After running the playbook, you should see the following relevent output:

```bash

TASK [DEBUG AND PRINT TO TERMINAL] *******************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab04-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [riverbed-1] => {
    "mgmt_int": "unknown"
}
ok: [riverbed-2] => {
    "mgmt_int": "unknown"
}

```

Now we have a default value provided for all devices.  

Also, notice how the more specific group variables are taking priority over the _all_ group.


### Task 3 - Adding & Printing Host Variables

In this task, you'll add "host based variables" to two hosts and print them to the terminal using the same playbook as the previous task.

##### Step 1 - Add host-level variables

Add two host variables in your inventory.  Set the value of the `mgmt_int` variable to "BDI1000" for both the **orko-lab04-red-router** and **orko-lab04-red-switch** devices:

```YAML
all:
  vars:
    mgmt_int: unknown

  children:

    orko_routers:
      hosts:
        orko-lab04-red-router:
          mgmt_int: BDI1000
        orko-lab04-yellow-router:
      vars:
        mgmt_int: Loopback0

    orko_switches:
      hosts:
        orko-lab04-red-switch:
          mgmt_int: BDI1000
        orko-lab04-yellow-switch:
      vars:
        mgmt_int: Vlan1000

    orko_riverbeds:
      hosts:
        riverbed-1:
        riverbed-2:
```


##### Step 2 - Run the playbook

After running the playbook, you should see the following relevent output:

```bash

TASK [DEBUG AND PRINT TO TERMINAL] *******************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "mgmt_int": "BDI1000"
}
ok: [orko-lab04-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab04-red-switch] => {
    "mgmt_int": "BDI1000"
}
ok: [orko-lab04-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [riverbed-1] => {
    "mgmt_int": "unknown"
}
ok: [riverbed-2] => {
    "mgmt_int": "unknown"
}

 
```

Take a minute to think about the variable priority occurring.  The **all** group is serving as the default, then specific group variables take priority over the **all** group, and then host variables take priority over the specific group variables.


### Task 4 - Using the msg Parameter

This task will introduce the `msg` parameter for the `debug` module.  Using `msg` is mutually exclusive with the `var` parameter.

When you just want to print a single variable, you use the `var` parameter.  If you want to add context (add a full sentence), you should use the `msg` parameter.

##### Step 1 - Add debug task to the playbook

Add a new task to the playbook to debug the `inventory_hostname` and `mgmt_int` variables.

**IMPORTANT**: The `inventory_hostname` variable is a built-in variable that's equal to the hostname of the device as you've defined it in the inventory.yml file.

```yaml
---

  - name: USING THE DEBUG MODULE
    hosts: all
    connection: local
    gather_facts: no


    tasks:
      - name: DEBUG AND PRINT TO TERMINAL
        debug: 
          var: mgmt_int

      - name: DEBUG AND PRINT THE MANAGEMENT INTERFACE
        debug: 
          msg: "The management interface for {{ inventory_hostname }} is {{ mgmt_int }}."
```

##### Step 2 - Run the playbook

After running the playbook, you should see the following relevent output:

```

TASK [DEBUG AND PRINT THE MANAGEMENT INTERFACE] ******************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "msg": "The management interface for orko-lab04-red-router is BDI1000."
}
ok: [orko-lab04-yellow-router] => {
    "msg": "The management interface for orko-lab04-yellow-router is Loopback0."
}
ok: [orko-lab04-red-switch] => {
    "msg": "The management interface for orko-lab04-red-switch is BDI1000."
}
ok: [orko-lab04-yellow-switch] => {
    "msg": "The management interface for orko-lab04-yellow-switch is Vlan1000."
}
ok: [riverbed-1] => {
    "msg": "The management interface for riverbed-1 is unknown."
}
ok: [riverbed-2] => {
    "msg": "The management interface for riverbed-2 is unknown."
}

```


### Task 5 - Exploring built-in Variables

This task will introduce built-in variables or another way to call them are __magic__ or __special__ variables. These variables are built-in to Ansible to reflect internal state. 


##### Step 1 - Add the `ansible_host` variable to a device

In the __inventory__ file add `ansible_host=1.1.1.1` as a host variable to `orko-lab04-red-router`. The `ansible_host` variable is helpful if the inventory hostname is not resolvable via DNS. You can set it to the IP address of the host and use it instead of `inventory_hostname` to access IP/FQDN.


```yaml
all:
  vars:
    mgmt_int: unknown

  children:

    orko_routers:
      hosts:
        orko-lab04-red-router:
          mgmt_int: BDI1000
          ansible_host: 1.1.1.1
        orko-lab04-yellow-router:
      vars:
        mgmt_int: Loopback0

    orko_switches:
      hosts:
        orko-lab04-red-switch:
          mgmt_int: BDI1000
        orko-lab04-yellow-switch:
      vars:
        mgmt_int: Vlan1000

    orko_riverbeds:
      hosts:
        riverbed-1:
        riverbed-2:
```


##### Step 2 - Add a debug task with a message

Add a new task to the existing playbook to see the difference between `inventory_hostname` and `ansible_host`.

```yaml

      - name: DEBUG AND PRINT INVENTORY_HOSTNAME VS ANSIBLE_HOST
        debug: 
           msg: " inventory_hostname: '{{ inventory_hostname }}'    -    ansible_host: '{{ ansible_host }}' "
```

##### Step 3 - Run the playbook

After running the playbook, you should see the following relevent output:


```

}

TASK [DEBUG AND PRINT INVENTORY_HOSTNAME VS ANSIBLE_HOST] ********************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "msg": " inventory_hostname: 'orko-lab04-red-router'    -    ansible_host: '1.1.1.1' "
}
ok: [orko-lab04-yellow-router] => {
    "msg": " inventory_hostname: 'orko-lab04-yellow-router'    -    ansible_host: 'orko-lab04-yellow-router' "
}
ok: [orko-lab04-red-switch] => {
    "msg": " inventory_hostname: 'orko-lab04-red-switch'    -    ansible_host: 'orko-lab04-red-switch' "
}
ok: [orko-lab04-yellow-switch] => {
    "msg": " inventory_hostname: 'orko-lab04-yellow-switch'    -    ansible_host: 'orko-lab04-yellow-switch' "
}
ok: [riverbed-1] => {
    "msg": " inventory_hostname: 'riverbed-1'    -    ansible_host: 'riverbed-1' "
}
ok: [riverbed-2] => {
    "msg": " inventory_hostname: 'riverbed-2'    -    ansible_host: 'riverbed-2' "
}
 

```

From the output, you will notice that you successfully outputted the `ansible_host` for `orko-lab04-red-router`, but what happened to the devices that did not have this variable defined?

By default, Ansible will use the `inventory_hostname` in order to reach a device if the `ansible_host` is not defined.  The same goes for any situation where you attempt to reference the `ansible_host` special variable.

##### Step 5 - Replace existing tasks

In your playbook, remove all existing tasks and add a new task to the playbook to debug a variable called `play_hosts`.  It will return a list of inventory hostnames that are in scope for the current play:

```yaml
---

  - name: USING THE DEBUG MODULE
    hosts: all
    connection: local
    gather_facts: no


    tasks:
      - name: DEBUG AND PRINT LIST OF PLAY_HOSTS
        debug: 
          var: play_hosts
```

##### Step 5 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [DEBUG AND PRINT LIST OF PLAY_HOSTS] ************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}
ok: [orko-lab04-yellow-router] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}
ok: [orko-lab04-red-switch] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}
ok: [orko-lab04-yellow-switch] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}
ok: [riverbed-1] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}
ok: [riverbed-2] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}


```

You'll also note that EVERY device actually has a `play_hosts` variable. 

##### Step 6 - Set the task to only run once

In the task in your playbook, add the `run_once` option:

```yaml
      - name: DEBUG AND PRINT LIST OF PLAY_HOSTS
        debug: 
          var: play_hosts
        run_once: true
```

**Since every device in the play has the same list of `play_hosts`, we really only need to execute this task once.  The `run_once` option will execute the task against the first device only.**

##### Step 7 - Run the playbook

After running the playbook, you should see the following relevent output:

```

TASK [DEBUG AND PRINT LIST OF PLAY_HOSTS] ************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "play_hosts": [
        "orko-lab04-red-router",
        "orko-lab04-yellow-router",
        "orko-lab04-red-switch",
        "orko-lab04-yellow-switch",
        "riverbed-1",
        "riverbed-2"
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
orko-lab04-red-router      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

From the output you can see that the task only ran for the first device, which was `orko-lab04-red-router`.


##### Step 7 - Add parent group to inventory

In your inventory, add a child group called `orko` to the `all` group, and set its *children* to include **every other** group in the inventory:

```yaml
all:
  vars:
    mgmt_int: unknown

  children:

    orko:
      children:

        orko_routers:
          hosts:
            orko-lab04-red-router:
              mgmt_int: BDI1000
              ansible_host: 1.1.1.1
            orko-lab04-yellow-router:
          vars:
            mgmt_int: Loopback0

        orko_switches:
          hosts:
            orko-lab04-red-switch:
              mgmt_int: BDI1000
            orko-lab04-yellow-switch:
          vars:
            mgmt_int: Vlan1000

        orko_riverbeds:
          hosts:
            riverbed-1:
            riverbed-2:
```

##### Step 8 - Debug `group_names` special variable


Add a new task to the existing playbook. To debug `group_names` which will return a list of all groups that the **current host** is a member of.

```yaml
      - name: DEBUG AND PRINT GROUP_NAMES
        debug: 
          var: group_names
```

##### Step 9 - Execute the playbook

After running the playbook, you should see the following relevent output:

```commandline

TASK [DEBUG AND PRINT GROUP_NAMES] *******************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "group_names": [
        "orko",
        "orko_routers"
    ]
}
ok: [orko-lab04-yellow-router] => {
    "group_names": [
        "orko",
        "orko_routers"
    ]
}
ok: [orko-lab04-red-switch] => {
    "group_names": [
        "orko",
        "orko_switches"
    ]
}
ok: [orko-lab04-yellow-switch] => {
    "group_names": [
        "orko",
        "orko_switches"
    ]
}
ok: [riverbed-1] => {
    "group_names": [
        "orko",
        "orko_riverbeds"
    ]
}
ok: [riverbed-2] => {
    "group_names": [
        "orko",
        "orko_riverbeds"
    ]
}
  
```

From the output, you can see that the `group_names` special variable includes **every** group that a given device is in, including parent groups.

> **Note**: Since the `group_names` special variable might be different for each device, you would not want to add the `run_once` task option.


##### Step 10 - Debug the 'groups' special variable

Add a new task to the existing playbook. Now we'll debug a special variable called `groups` which will return a dictionary (or hash) in which all the keys are all group names defined in the inventory file and values are list of hosts that are members of the group.

```yaml
      - name: DEBUG AND PRINT GROUPS
        debug: 
          var: groups
        run_once: true
```

Since the `groups` special variable is the same for all devices, we are adding the `run_once` option yet again.

##### Step 11 - Run the playbook

After running the playbook, you should see the following relevent output:

```commandline


TASK [DEBUG AND PRINT GROUPS] ************************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "groups": {
        "all": [
            "orko-lab04-red-router",
            "orko-lab04-yellow-router",
            "orko-lab04-red-switch",
            "orko-lab04-yellow-switch",
            "riverbed-1",
            "riverbed-2"
        ],
        "orko": [
            "orko-lab04-red-router",
            "orko-lab04-yellow-router",
            "orko-lab04-red-switch",
            "orko-lab04-yellow-switch",
            "riverbed-1",
            "riverbed-2"
        ],
        "orko_riverbeds": [
            "riverbed-1",
            "riverbed-2"
        ],
        "orko_routers": [
            "orko-lab04-red-router",
            "orko-lab04-yellow-router"
        ],
        "orko_switches": [
            "orko-lab04-red-switch",
            "orko-lab04-yellow-switch"
        ],
        "ungrouped": []
    }
}


```

##### Step 12 - Debug Ansible version

Remove all existing tasks and replace them with a new task so we can verify and see what version of Ansible is being used to execute the playbook.  The variable `ansible_version` will return a dictionary representing the Ansible major, minor, revision of the release.

```yaml
---

  - name: USING THE DEBUG MODULE
    hosts: all
    connection: local
    gather_facts: no


    tasks:
      - name: DEBUG AND PRINT ANSIBLE_VERSION
        debug: 
           msg: "Ansible Version: '{{ ansible_version }}'"
        run_once: true
```

##### Step 13 - Run the playbook

After running the playbook, you should see the following relevent output:


```commandline
PLAY [USING THE DEBUG MODULE] ************************************************************************************************************************************************************************************************************************************************************************************************************************************************

TASK [DEBUG AND PRINT ANSIBLE_VERSION] ***************************************************************************************************************************************************************************************************************************************************************************************************************************************
ok: [orko-lab04-red-router] => {
    "msg": "Ansible Version: '{'string': '2.9.6', 'full': '2.9.6', 'major': 2, 'minor': 9, 'revision': 6}'"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
orko-lab04-red-router      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

There are several more built-in variables, but this is a good start to understand what's possible while printing different variables with the `debug` module.



