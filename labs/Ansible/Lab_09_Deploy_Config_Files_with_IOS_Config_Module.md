# Lab 09 - Deploying Configs From a File


In the last lab, you deployed configurations while hard-coding commands in a playbook.  In this lab, you will deploy network configurations from a pre-built configuration file.

**In this lab you will:**
* Create config files that will contain snmp configurations
* Create a playbook that uses the **ios_config** module to send commands to devices using previously created config files

**Additional Resources:**
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible IOS Config Module](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_config_module.html)


### Task 1 - Deploy SNMP Configurations from files

##### Step 1 - Create the inventory file

`cd` into the `ansible_labs/lab09` folder and create `inventory.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab09 && touch inventory.yml
student@student-training:~/ansible_labs/lab09$ 
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
        - orko-lab09-red-router
        - orko-lab09-red-switch

    - Vars:
        - enclave: red

* **An `orko_yellow` group:**
    - Hosts:
        - orko-lab09-yellow-router
        - orko-lab09-yellow-switch

    - Vars:
        - enclave: yellow


**At this point, your inventory should look like the following:**

```yaml
all:
  children:

    orko_red:
      hosts:
        orko-lab09-red-router:
        orko-lab09-red-switch:

      vars:
        enclave: red

    orko_yellow:
      hosts:
        orko-lab09-yellow-router:
        orko-lab09-yellow-switch:

      vars:
        enclave: yellow

```

##### Step 3 - Create 'configs' folder and SNMP configuration files


Create two files that will contain the SNMP configuration - one for each enclave:

```
student@student-training:~/ansible_labs/lab09$ mkdir configs
student@student-training:~/ansible_labs/lab09$ touch configs/red-snmp.cfg configs/yellow-snmp.cfg

```

##### Step 4 - Update each SNMP config file

Add the following contents to each file:

**red-snmp.cfg:**

```
snmp-server community orko_red RO
snmp-server location FORT_BRAGG      
snmp-server contact JOHN_SMITH     

```

**yellow-snmp.cfg:**

```
snmp-server community orko_yellow RO
snmp-server location FORT_BRAGG      
snmp-server contact CHUCK_NORRIS
```


##### Step 5 - Create the playbook

Create a playbook called `lab09.yml` with the following contents: 


```yaml

---

  - name: PLAY 1 - DEPLOYING SNMP CONFIGURATIONS
    hosts: all
    connection: network_cli
    gather_facts: no

    tasks:

      - name: TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST
        ios_config:
          src: ./configs/{{ enclave }}-snmp.cfg

```

Here, we are using each device's `enclave` variable to determine **which SNMP configuration file is used**.

##### Step 7 - Run the playbook

Run the playbook.


```
student@student-training:~/ansible_labs/lab09$ ansible-playbook lab09.yml -i inventory.yml
PLAY [PLAY 1 - DEPLOYING SNMP CONFIGURATIONS] **********************************************************

TASK [TASK 1 in PLAY 1 - ENSURE SNMP COMMANDS EXIST] *******************************************
changed: [orko-lab09-red-router]
changed: [orko-lab09-red-switch]
changed: [orko-lab09-yellow-router]
changed: [orko-lab09-yellow-switch]

PLAY RECAP ****************************************************************************************************
orko-lab09-red-router                       : ok=0    changed=1    unreachable=0    failed=0   
orko-lab09-red-switch                       : ok=0    changed=1    unreachable=0    failed=0   
orko-lab09-yellow-router                    : ok=0    changed=1    unreachable=0    failed=0   
orko-lab09-yellow-switch                    : ok=0    changed=1    unreachable=0    failed=0  

student@student-training:~/ansible_labs/lab09$
```


##### Step 8 - Verify correct configurations were deployed

If we console into the necessary devices, you should see that the SNMP configuration should change depending upon the device's enclave.

CODE BLOCK WILL GO HERE