# Lab 08 - Deploying SNMP Configurations with Ansible


**In this lab you will:**
* Create a basic inventory
* Create a playbook that uses the **ios_config** module to send commands to devices
* Use special variables to specify the user to login as when reaching devices

**Additional Resources:**
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible IOS Config Module](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_config_module.html)

### Task 1 - Managing SNMP Global Configuration Commands for IOS

This lab provides an introduction to using Ansible creating your first playbook to deploy basic configurations.

You will be using the `ios_config` module which allows you to send multiple configuration settings to Cisco IOS devices

##### Step 1 - Create the inventory file

`cd` into the `ansible_labs/lab08` folder and create `inventory.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab08 && touch inventory.yml
student@student-training:~/ansible_labs/lab08$ 
```


##### Step 2 - Build the inventory

On your own, attempt to build the inventory.

Your inventory should include the following groups and their associated hosts, vars, and children:


* **An `orko_routers` group:**
    - Hosts:
        - orko-lab08-red-router
        - orko-lab08-yellow-router

    - Vars:
        - ansible_network_os: ios

* **Each host should have the following `ansible_host` variable:**
    - orko-lab08-red-router: 1.1.1.1
    - orko-lab08-yellow-router: 2.2.2.2

**At this point, your inventory should look like the following:**

```yaml
orko_routers:
  vars:
    ansible_network_os: ios
  hosts:
    orko-lab08-red-router:
      ansible_host: 1.1.1.1
    orko-lab08-yellow-router:
      ansible_host: 2.2.2.2

```

##### Step 3 - Create the playbook

**Create a playbook called `lab08.yml` with the following contents:**

```yaml

---

- name: PLAY 1 - DEPLOYING SNMP CONFIGURATIONS ON IOS
  hosts: orko_routers
  connection: network_cli
  gather_facts: no

  tasks:

    - name: TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST ON IOS DEVICES
      ios_config:
        commands:
          - snmp-server community orko RO
          - snmp-server location FORT_BRAGG
          - snmp-server contact CHUCK_NORRIS

```

This is a single play with a single task -- you can take note of the descriptions found next to the `name` key, but remember the file itself is the playbook.  The playbook consists of a list of plays and each play consists of a list of tasks.  

In the task above, there are three SNMP commands defined.  This task will ensure those commands exist on the devices that are defined next to the `hosts:` key in the play definition.


**Here are some other details on the task you should be aware of:**
  * `ios_config` is the module name
  * Technically `lines` is the parameter and `commands` is an alias for `lines` since they are just "lines" within a config file.
  * Take note of the data type of `commands` - it is a list as can be inferred from the hyphens in YAML.
  * `name` is an optional task attribute that maps to arbitrary text that is displayed when you run the playbook providing context on where in the playbook execution you are.  You'll see this in the next step!



##### Step 7 - Run the playbook

Execute the playbook using the following Linux command:

```
student@student-training:~/ansible_labs/lab08$ ansible-playbook lab08.yml -i inventory.yml -u orko -k
```

**Few notes about this command:**
* We are using the `ansible-playbook` program to execute the playbook
* `lab08.yml` is the playbook we are executing
* The `-i` flag maps to the inventory file to used when running this playbook - in our case, that's `inventory.yml`
* The `-u` flag maps to the username needed to log in to the network devices
* The `-k` flag states to prompt for the password ("P@ssw0rd") needed to log in to the network devices

Upon executing the playbook:

```
student@student-training:~/ansible_labs$ ansible-playbook lab08.yml -i inventory.yml -u orko -k
SSH password: 

PLAY [PLAY 1 - DEPLOYING SNMP CONFIGURATIONS ON IOS] ***********************************************************************************************************************************

TASK [TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST ON IOS DEVICES] ********************************************************************************************************************
changed: [orko-lab08-yellow-router]
changed: [orko-lab08-red-router]

PLAY RECAP *****************************************************************************************************************************************************************************
orko-lab08-red-router                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab08-yellow-router                    : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

You should have seen **changed** for each device.  This is because the devices didn't have this configuration yet and an actual change was applied to each device.

Also take note of the **PLAY RECAP**.  This is a summary of:

* how many tasks by device occurred successfully denoted by _ok_
* how many tasks by device made a configuration change denoted by _changed_
* how many devices are unreachable and how many tasks failed, also broken down by device.


At this point, you've ran your first playbook and have successfully configured SNMP on several devices!

### Task 2 - Understanding Idempotency

##### Step 1

Re-run the same exact playbook as the last task:

```
student@student-training:~/ansible_labs$ ansible-playbook -i inventory lab08.yml -u orko -k
SSH password:

PLAY [PLAY 1 - DEPLOYING SNMP CONFIGURATIONS ON IOS] **********************************************************

TASK [TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST ON IOS DEVICES] *******************************************
ok: [orko-lab08-yellow-router]
ok: [orko-lab08-red-router]

PLAY RECAP ****************************************************************************************************
orko-lab08-red-router                       : ok=1    changed=0    unreachable=0    failed=0   
orko-lab08-yellow-router                    : ok=1    changed=0    unreachable=0    failed=0   
```

Do you see the difference from the previous output?

This task is highlighting the fact that the **ios_config** module is idempotent.  

**This means you can now run the playbook 10000 times, but Ansible will only ever make the change once.**

> Note: Ansible itself is not idempotent the modules are.  Therefore, there can be modules that are NOT idempotent.  Be sure to understand how each module works before running them in production.




### Task 3 - Using Inventory Parameters (Variables)

In this task, we're going to introduce two inventory parameters that are helpful to be aware of when using Ansible.  They are `ansible_user` and `ansible_ssh_pass` and will be used to simplify passing in credentials when executing playbooks.


##### Step 1

You probably don't want to enter your credentials every time you run a playbook.  

While not necessarily recommended for production, the next for learning Ansible, is to put your credentials in your inventory file.

Lets add the `ansible_user` variable to our `orko_routers` group::

```yaml
orko_routers:
  vars:
    ansible_network_os: ios
    ansible_user: orko
```

This is assigning the value of "orko" to the built-in Ansible variable called `ansible_user` for devices in the **orko_routers** group.  The syntax above the variable is required for "group variables" in the inventory file.

##### Step 2

Execute the same playbook.  This time do not use the `-u` flag.

```
student@student-training:~/ansible_labs/lab08$ ansible-playbook lab08.yml -i inventory.yml -k
SSH password:
# output omitted
```

Notice how the `ansible_user` parameter is being used for your device username now.

##### Step 3

Add the `ansible_ssh_pass` variable to your inventory file.  Assign it the value of `P@ssw0rd`.

```yaml
orko_routers:
  vars:
    ansible_network_os: ios
    ansible_user: orko
    ansible_ssh_pass: P@ssword

```

This is assigning the value of "P@ssword" to the built-in Ansible variable called `ansible_ssh_pass` which is used as the password to log in to your devices.


##### Step 4

Re-run the playbook.  

This time DO NOT USE either the `-u` and `-k` flags.

```
student@student-training:~/ansible_labs/lab08$ ansible-playbook lab08.yml -i inventory.yml

# output omitted - same as Step 11
```


The updated playbook should look like this:

```yaml

---

  - name: PLAY 1 - DEPLOYING SNMP CONFIGURATIONS ON IOS
    hosts: ios
    connection: network_cli
    gather_facts: no

    tasks:

      - name: TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST ON IOS DEVICES
        ios_config:
          commands:
            - snmp-server community orko RO
            - snmp-server location FORT_BRAGG
            - snmp-server contact CHUCK_NORRIS


```


##### Step 5

Re-run the playbook ensuring everything is correct from a syntax perspective and that the playbook still works.

```
student@student-training:~/ansible_labs$ ansible-playbook lab08.yml -i inventory.yml

```

