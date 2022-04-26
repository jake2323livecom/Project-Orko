# Lab 02 - Working with Static Inventories in Ansible

In this lab, you will be creating a static inventory using a YAML file.  

You will learn how to create hosts and groups, and then apply variables to each.

**In this lab you will:**
* Create hosts
* Create hostvars
* Create basic groups
* Create parent groups

**Additional Resources:**
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Using variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible Debug Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)

### Task 1 - Creating hosts

##### Step 1 - Creating a basic inventory

`cd` into the **ansible_labs/lab02** directory, create an **inventory.yml** file with the following contents:

```yaml
all:
  hosts:
    orko-lab02-red-router:
    orko-lab02-yellow-router:
    orko-lab02-red-switch:
    orko-lab02-yellow-switch:
```

At the root level of any static inventory are **groups**.

In this inventory, we are defining an *all* group at the root level of our inventory with **4 hosts**: 2 routers and 2 switches.  It probably looks strange that each device's hostname ends with a colon and nothing else, but this is still a valid Ansible inventory.

When creating a static inventory, we just have to make sure that every host is a member of ***at least one group***.  In this case, we can use the group called `all`.

##### Step 2 - Create a basic playbook

Create a playbook called 'lab02.yml' with the following contents:

```yaml
---
- name: Debug Inventory
  hosts: all
  gather_facts: false
  connection: local

  tasks:

    - name: debug variables
      debug:
        var: inventory_hostname
```

This is the playbook you will use to test your inventory.

**For now, just remember these key things:**
* The `hosts` key towards the top of the playbook specifies which hosts in our *inventory* we want to target in the playbook.  The value of the hosts key can be either a group or a host.  In this case, we can target the **all** group in our inventory.
* A few lines below that, there is a **tasks** list containing everything we want to do.  For now, we just have one task that will output the `inventory_hostname` variable for each device that is targeted in the playbook.  This is accomplished by using the **debug** module.
* `inventory_hostname` is a special variable that Ansible creates for us by default.  The value of this variable will be the hostname of a given device in the *inventory*.


##### Step 3 - Run the playbook

Execute your playbook with the following command:

```bash
student@student-training:~/ansible_labs/lab02$ ansible-playbook lab02.yml -i inventory.yml
```

**Few notes about this command:**
* We are using the `ansible-playbook` program to execute the playbook
* `lab02.yml` is the playbook we are executing
* The `-i` flag maps to the inventory file to used when running this playbook - in our case, that's `inventory.yml`

There are a few other common options you can use with the `ansible-playbook` command that you will see in later labs.

**After running the playbook you should see the following relevent output:**

```bash
student@student-training:~/ansible_labs/lab02$ ansible-playbook lab02.yml -i inventory.yml

PLAY [Debug Inventory] *************************************************************************************************************

TASK [debug inventory hostname] ****************************************************************************************************
ok: [orko-lab02-red-router] => {
    "inventory_hostname": "orko-lab02-red-router"
}
ok: [orko-lab02-yellow-router] => {
    "inventory_hostname": "orko-lab02-yellow-router"
}
ok: [orko-lab02-red-switch] => {
    "inventory_hostname": "orko-lab02-red-switch"
}
ok: [orko-lab02-yellow-switch] => {
    "inventory_hostname": "orko-lab02-yellow-switch"
}

PLAY RECAP *************************************************************************************************************************
orko-lab02-red-router                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-router                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-red-switch                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-switch                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

**Notice that the `inventory_hostname` variable for each device was outputted**


### Task 2 - Creating host_vars

##### Step 1 - Assign 'ansible_host' variable

In order to assign host-level variables, known as **hostvars**, we can create `key:value` pairs under each device in our inventory.

Replace the contents of **inventory.yml** with the following:

```yaml
all:
  hosts:
    orko-lab02-red-router:
      ansible_host: 1.1.1.1

    orko-lab02-yellow-router:
      ansible_host: 2.2.2.2

    orko-lab02-red-switch:
      ansible_host: 3.3.3.3

    orko-lab02-yellow-switch:
      ansible_host: 4.4.4.4
```

**Notice here we are setting the `ansible_host` variable for each device.**

> The `ansible_host` variable is the IP address that Ansible will use to actually connect to the devices.  It is an Ansible **special variable**. 


##### Step 2 - Debug the ansible_host
Now lets edit our playbook so that it outputs the `ansible_host` variable instead of the `inventory_hostname`:

```yaml
  tasks:

    - name: debug variables
      debug:
        var: ansible_host
```

##### Step 3 - Run the playbook

Save and execute your playbook.

You should see the following output:

```bash
student@student-training:~/ansible_labs/lab02$ ansible-playbook lab02.yml -i inventory.yml 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "ansible_host": "1.1.1.1"
}
ok: [orko-lab02-yellow-router] => {
    "ansible_host": "2.2.2.2"
}
ok: [orko-lab02-red-switch] => {
    "ansible_host": "3.3.3.3"
}
ok: [orko-lab02-yellow-switch] => {
    "ansible_host": "4.4.4.4"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab02-red-router                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-router                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-red-switch                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-switch                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

**Notice that now, the `ansible_host` of each device is outputted.**

##### Step 4 - Assign `mgmt_int` variable

Let's assume that we want to create a `mgmt_int` variable for each device, and we want the value of the variable to be `Loopback0` if the device is a router, and `Vlan1000` if the device is a switch.  

Let's add this new variable to each device in the inventory:


```yaml
all:
  hosts:
    orko-lab02-red-router:
      ansible_host: 1.1.1.1
      mgmt_int: Loopback0

    orko-lab02-yellow-router:
      ansible_host: 2.2.2.2
      mgmt_int: Loopback0

    orko-lab02-red-switch:
      ansible_host: 3.3.3.3
      mgmt_int: Vlan1000

    orko-lab02-yellow-switch:
      ansible_host: 4.4.4.4
      mgmt_int: Vlan1000
```

##### Step 5 - Debug the `mgmt_int` variable

Edit the task in your playbook so that it debugs the `mgmt_int` variable:

```yaml
  tasks:

    - name: debug variables
      debug:
        var: mgmt_int
```

##### Step 6 - Run the playbook

Save and execute your playbook.

You should see the following relevent output:

```bash

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab02-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab02-red-router                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-router                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-red-switch                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-switch                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

You might have noticed that creating the same variable for each host was a bit of a pain, and probably wouldn't be scalable if you had hundreds or even thousands of devices.  This is where inventory groups come into play.  In the next sections, you will create some groups and learn how to apply variables at the group-level instead of the host-level.

### Task 3 - Creating Groups

Groups are a convenient way of applying variables to multiple devices at once.  Groups are normally created based on device roles or locations, for example, core switch, distribution switch, or access switch.

**Each group in an Ansible inventory can have three things:**
* hosts
* vars
* children

In this section, you will create a group for `routers` and a group for `switches`.  These two groups will *initially* be children of the `all` group.  You will be able to completely remove the `all` group from your inventory.

This is because Ansible will ***automatically*** add all grouped devices to the `all` group for you.

You will then move the `mgmt_int` variable from the host-level up to the group-level so that you don't have to specify it for each device.

##### Step 1 - Create groups

Replace your inventory with the following:

```yaml
all:
  children:

    routers:

      hosts:
        orko-lab02-red-router:
          ansible_host: 1.1.1.1
          mgmt_int: Loopback0

        orko-lab02-yellow-router:
          ansible_host: 2.2.2.2
          mgmt_int: Loopback0

    switches:

      hosts:
        orko-lab02-red-switch:
          ansible_host: 3.3.3.3
          mgmt_int: Vlan1000

        orko-lab02-yellow-switch:
          ansible_host: 4.4.4.4
          mgmt_int: Vlan1000
```


**Here's what's changed:**
* The `routers` and `switches` groups ***now*** fall under the `children` key of the `all` group.
* Our 4 hosts no longer reside in the `hosts` key of the `all` group.
* Both the `routers` and `switches` groups have a `hosts` key with the appropriate hosts based on role.


##### Step 2 - Remove the `all` Group

Now that all of our hosts belong to a group, we no longer need the `all` group in our inventory.

**Replace your inventory with the following:**

```yaml
routers:

  hosts:
    orko-lab02-red-router:
      ansible_host: 1.1.1.1
      mgmt_int: Loopback0

    orko-lab02-yellow-router:
      ansible_host: 2.2.2.2
      mgmt_int: Loopback0

switches:

  hosts:
    orko-lab02-red-switch:
      ansible_host: 3.3.3.3
      mgmt_int: Vlan1000

    orko-lab02-yellow-switch:
      ansible_host: 4.4.4.4
      mgmt_int: Vlan1000
```


Since **Ansible creates the `all` group by default**, we can still reference it in the `hosts` key in our playbook.


##### Step 3 - Run your playbook

After running your playbook, you should see the following relevent output:

```bash
PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab02-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab02-red-router                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-router                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-red-switch                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab02-yellow-switch                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

**From the output, you can see that all 4 devices were still targeted in the playbook execution.


##### Step 4 - Create Group-level Variables

Now we will move the `mgmt_int` variable from the **host-level** to the **group-level**.

This means we will assign the variable once for the `routers` group and once for the `switches` group.

**Replace your inventory with the following:**

```yaml
routers:

  vars:
    mgmt_int: Loopback0

  hosts:

    orko-lab02-red-router:
      ansible_host: 1.1.1.1

    orko-lab02-yellow-router:
      ansible_host: 2.2.2.2

switches:

  vars:
    mgmt_int: Vlan1000

  hosts:

    orko-lab02-red-switch:
      ansible_host: 3.3.3.3

    orko-lab02-yellow-switch:
      ansible_host: 4.4.4.4
```

All we've done here is add the `vars` key under both the `routers` and `switches` group.  Then we can put whatever variables we want beneath that as `key:value` pairs, in this case, just the `mgmt_int` variable.  This is how we can say that the value should be `Loopback0` for all routers and `Vlan1000` for all switches.



##### Step 5 - Run the playbook

After running your playbook, you should see the following relevent output:

```bash
PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab02-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}

```
Notice how each device inherited the variables from the group it belonged to, so we were able to output the appropriate value for each device.

Now, if we needed to add another router to our inventory, we wouldn't have to assign the `mgmt_int` variable again, we'd just need to make sure the device was a member of the `routers` group.  

##### Step 6 - Create additional host

Now lets test the theory.  Lets add two more hosts to our inventory, one router and one switch:

```yaml
routers:

  vars:
    mgmt_int: Loopback0

  hosts:
    orko-lab02-red-router:
      ansible_host: 1.1.1.1

    orko-lab02-yellow-router:
      ansible_host: 2.2.2.2

    router_3:

switches:

  vars:
    mgmt_int: Vlan1000

  hosts:
    orko-lab02-red-switch:
      ansible_host: 3.3.3.3

    orko-lab02-yellow-switch:
      ansible_host: 4.4.4.4

    switch_3:
```

We've added a `router_3` and `switch_3`, and given them `ansible_host` variables which are host-specific.  In this case, don't worry about adding the `ansible_host` variable for the two new hosts.  


##### Step 7 - Run the playbook

After running your playbook, you should see the following relevent output:

```bash
TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-yellow-router] => {
    "mgmt_int": "Loopback0"
}
ok: [router_3] => {
    "mgmt_int": "Loopback0"
}
ok: [orko-lab02-red-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [orko-lab02-yellow-switch] => {
    "mgmt_int": "Vlan1000"
}
ok: [switch_3] => {
    "mgmt_int": "Vlan1000"
}

```

Now you can see that the new devices in inherited their group-variables automatically.  

Now what if we wanted both the `routers` and `switches` group to have a variable with the same value, such as a domain-name?  

We will solve this problem in the next section.

### Task 4 - Creating parent groups

In this section you will create a `vars` key for the `all` group, so that all child groups will inherit those variables, and you won't have to define them for multiple groups.  

In this example, you will create a variable called `domain_name`.

##### Step 1: Adding a variable to a parent group

Replace your inventory with the following:

```yaml
all:

  vars:
    domain_name: test.local

  children:
    routers:

      vars:
        mgmt_int: Loopback0

      hosts:
        orko-lab02-red-router:
        orko-lab02-yellow-router:
        router_3:

    switches:

      vars:
        mgmt_int: Vlan1000

      hosts:
        orko-lab02-red-switch:
        orko-lab02-yellow-switch:
        switch_3:

```

**Here are the changes:**
* We've re-added the explicit definition of the `all` group so that we can apply variables to every device.
* We've added a `vars` key to the `all` group.  Since `routers` and `switches` are children of `all`, they will inherit the `domain_name` variable.
* We've gotten rid of the `ansible_host` variable.  We won't need it for the remainder of the lab.


##### Step 2: Edit your playbook

In your playbook, debug the `domain_name` variable instead of the `mgmt_int`:

```yaml
var: domain_name
```

##### Step 3 - Run the playbook

After running your playbook, you should see the following relevent output:

```bash

TASK [debug variables] *******************************************************************************************************************************************************
ok: [orko-lab02-red-router] => {
    "domain_name": "test.local"
}
ok: [orko-lab02-yellow-router] => {
    "domain_name": "test.local"
}
ok: [router_3] => {
    "domain_name": "test.local"
}
ok: [orko-lab02-red-switch] => {
    "domain_name": "test.local"
}
ok: [orko-lab02-yellow-switch] => {
    "domain_name": "test.local"
}
ok: [switch_3] => {
    "domain_name": "test.local"
}


```

**Notice how all devices have the same `domain_name` variable value.**


