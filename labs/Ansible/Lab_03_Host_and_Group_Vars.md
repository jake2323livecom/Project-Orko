# Lab 03 - Working with the `group_vars` and `host_vars` directories

In this lab you will be working with the `host_vars` and `group_vars` directories in Ansible.

They allow you to organize both host and group variables in specific files, rather than by being defined in your inventory. 

>**Note:** The main benefit of organizing variables in the `host_vars` and `group_vars` directories is really only seen in a production deployment.  In advanced Ansible configurations, all Ansible files are actually kept in a remote Git repository, and so by keeping variables within the Git repository, you are given the ability to manage variables with version control.


**In this lab you will:**
* Create a basic inventory
* Work with the **host_vars** directory
* Work with the **group_vars** directory

**Additional Resources:**
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Organizing host and group variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#organizing-host-and-group-variables)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible Debug Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)

### Task 1 - Creating the Inventory

##### Step 1 - Create the inventory file

`cd` into the **ansible_labs/lab03** folder and create an **inventory.yml**:

```bash
student@student-training:~$ cd ansible_labs/lab03 && touch inventory.yml
student@student-training:~/ansible_labs/lab03$ 
```

##### Step 2 - Build the inventory

On your own, attempt to build the inventory.

Your inventory should include the following groups and their associated hosts, vars, and children:

* **An explicit `all` group:**
    - Children:
        - orko_red
        - orko_yellow

* **An `orko_red` group:**
    - Hosts:
        - orko-lab03-red-router
        - orko-lab03-red-switch


* **An `orko_yellow` group:**
    - Hosts:
        - orko-lab03-yellow-router
        - orko-lab03-yellow-switch


**Once completed, compare your inventory to the following:**

```yaml
all:
  children:
    orko_red:
      hosts:
        orko-lab03-red-router:
        orko-lab03-red-switch:
    orko_yellow:
      hosts:
        orko-lab03-yellow-router:
        orko-lab03-yellow-switch:
```

> We aren't defining any variables in the inventory itself because we will be defining variables within the `host_vars` and `group_vars` directories in later steps.

### Task 2 - Working with the 'host_vars' directory

By default, at playbook runtime, Ansible will look for a directory called `host_vars`.  

By creating a `YAML` or `JSON` file in the `host_vars` directory, you can assign variables to specific devices.  The only requirement is that the **name of each file has to match a device hostname in the inventory**.

For example, if there was a device in your inventory named `router-1`, you could create a `router-1.yml` file in your `host_vars` directory, and the variables you create in that file would be applied to the `router-1` device at playbook runtime.

Lets create all of the files necessary to define host variables using the `host_vars` directory.

##### Step 1 - Create the `host_vars` directory

```bash
student@student-training:~/ansible_labs/lab03$ mkdir host_vars
```

##### Step 2 - Create variable files for each device in the inventory

Within your `host_vars` directory, create four files, with each filename correlating to a device in the inventory:

```bash
student@student-training:~/ansible_labs/lab03$ touch host_vars/orko-lab03-red-router.yml
student@student-training:~/ansible_labs/lab03$ touch host_vars/orko-lab03-red-switch.yml
student@student-training:~/ansible_labs/lab03$ touch host_vars/orko-lab03-yellow-router.yml
student@student-training:~/ansible_labs/lab03$ touch host_vars/orko-lab03-yellow-switch.yml
```

##### Step 3 - Define `ansible_host` variable for each device

Within the variable file for each device, define the `ansible_host` variable using the YAML standard `key: value` pair.


**orko-lab03-red-router.yml:**

```yaml
ansible_host: 10.99.3.1
```

**orko-lab03-red-switch.yml:**

```yaml
ansible_host: 10.99.3.2
```

**orko-lab03-yellow-router.yml:**

```yaml
ansible_host: 10.99.3.3
```

**orko-lab03-yellow-switch.yml:**

```yaml
ansible_host: 10.99.3.4
```

##### Step 5 - Create a basic playbook

In the `ansible_labs/lab03` directory, create a `lab03.yml` playbook with the following contents:

```yaml
- name: Debug Variables
  hosts: all
  gather_facts: false
  connection: local

  tasks:
    - name: Debug ansible-host variable
      debug:
        var: ansible_host
```

##### Step 6 - Execute the playbook

After running your playbook, you should see the following relevent output:

```bash
student@student-training:~/ansible_labs/lab03$ ansible-playbook lab03.yml -i inventory.yml 

PLAY [Debug Variables] *******************************************************************************************************************************************************

TASK [Debug ansible-host variable] *******************************************************************************************************************************************
ok: [orko-lab03-red-router] => {
    "ansible_host": "10.99.3.1"
}
ok: [orko-lab03-red-switch] => {
    "ansible_host": "10.99.3.2"
}
ok: [orko-lab03-yellow-router] => {
    "ansible_host": "10.99.3.3"
}
ok: [orko-lab03-yellow-switch] => {
    "ansible_host": "10.99.3.4"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab03-red-router      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab03-red-switch      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab03-yellow-router   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab03-yellow-switch   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

**From the output we can see that the `ansible_host` was successfully assigned to each device.**


### Task 3 - Working with the 'group_vars' directory

The **group_vars** directory works similarly to the **host_vars** directory, except you would create variable files that correlate to group names instead of hostnames.

For example, if you had a group in your inventory called `ios`, you could create a file called `ios.yml` and define variables for the `ios` group in that file.

In this section, you will create a variable file for the `orko_red` and `orko_yellow` groups from your inventory.


##### Step 1 - Create the `group_vars` directory

```bash
student@student-training:~/ansible_labs/lab03/$ mkdir group_vars
```

##### Step 2 - Create variable files for each group in the inventory

Within your `group_vars` directory, create 2 files, with each filename correlating to a group in the inventory:

```bash
student@student-training:~/ansible_labs/lab03$ touch group_vars/orko_red.yml
student@student-training:~/ansible_labs/lab03$ touch group_vars/orko_yellow.yml
```

##### Step 3 - Define the `domain_name` variable for each group

Within the variable file for each group, define the `domain_name` variable using the YAML standard `key: value` pair.

**orko_red.yml:**

```yaml
domain_name: red.local
```

**orko_yellow.yml:**

```yaml
domain_name: yellow.local
```

##### Step 4 - Edit the playbook

In the first task of the playbook, change the debug variable to `domain_name`:

```yaml
    - name: Debug ansible-host variable
      debug:
        var: domain_name
```

##### Step 5 - Run your playbook

After running your playbook, you should see the following relevent output:

```bash

TASK [Debug ansible-host variable] *******************************************************************************************************************************************
ok: [orko-lab03-red-router] => {
    "domain_name": "red.local"
}
ok: [orko-lab03-red-switch] => {
    "domain_name": "red.local"
}
ok: [orko-lab03-yellow-router] => {
    "domain_name": "yellow.local"
}
ok: [orko-lab03-yellow-switch] => {
    "domain_name": "yellow.local"
}
 
```

**You can see from the output that we successfully assigned the `domain_name` variable at the group level.**

Using the **host_vars** and **group_vars** directories provide a convient way of managing hostvars and groupvars.