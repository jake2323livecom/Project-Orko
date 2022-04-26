# Lab 11 - Using Jinja Templates with the IOS Config Module

In this lab, you will create a Jinja template and use it with the `ios_config` module in order to update configurations of Cisco devices.

**In this lab you will:**
* Create a basic inventory
* Create a **group_vars** file for the **all** group
* Create Jinja Template for DHCP configurations
* Create a playbook that will use the template to send the DHCP configurations to devices

**Additional Resources:**
* [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible IOS Config Module](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_config_module.html)
* [Ansible Template Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
* [Live Jinja Parser](https://j2live.ttl255.com/)

### Task 1 - Prep

##### Step 1 - Create the inventory file

`cd` into the `ansible_labs/lab11` folder and create `inventory.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab11 && touch inventory.yml
student@student-training:~/ansible_labs/lab11$ 
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
        - orko-lab11-red-router
        - orko-lab11-yellow-router

* **An `orko_switches` group:**
    - Hosts:
        - orko-lab11-yellow-switch
        - orko-lab11-red-switch

**At this point, your inventory should look like the following:**

```yaml
all:
  children:

    orko_red:
      hosts:
        orko-lab11-red-router:
        orko-lab11-yellow-router:

    orko_yellow:
      hosts:
        orko-lab11-red-switch:
        orko-lab11-yellow-switch:

```


### Task 2 - Creating the Group Variables

In this section, you will be creating the necessary group variables that you will need in order to generate DHCP pool configurations.

You will be applying these variables to the `all` group.

##### Step 1 - Create the 'group_vars' directory

Create the `group_vars` directory within the `playbooks` directory:

```bash
student@student-training:~/ansible_labs/lab11$ mkdir group_vars
```

##### Step 2 - Create variables file for the `all` group

Create a file called `all.yml` within the `group_vars` directory:

```bash
student@student-training:~/ansible_labs/lab11$ touch group_vars/all.yml
```

##### Step 3 - Create the `dhcp` variable for the `all` group

Using your text-editor of choice, create a variable called `dhcp` in `all.yml`:

```yaml
dhcp:
  excluded:
    - 22.47.1.254
    - 172.24.1.254
  pools:
    - name: orko_data
      network: 22.47.1.0 255.255.255.0
      default_router: 22.47.1.254
    - name: orko_voice
      network: 172.24.1.0 255.255.255.0
      default_router: 172.24.1.254
```

**The `dhcp` variable includes:**
* A key called `excluded` which has a value that is a list of IP addresses
* A key called `pools` which is a **list of dictionaries**.  Each dictionary represents one DHCP pool.  Each pool includes three keys: **name, network, and default_router**.  This is the minimum information we need to configure working DHCP pools on a Cisco device.

> This is the variable you will reference in your Jinja templates in later steps. In this lab, the DHCP configuration will be the same for all devices, but this is just for the sake of learning.  In a production environment, you'd use advanced methods to create a custom DHCP configuration for each device.


### Task 3 - Creating the Template

In this section, you will be creating a Jinja template that will generate DHCP pool configurations that you will later push to devices using the `ios_config` module.


##### Step 1 - Create the 'templates' directory

Create a `templates` directory:

```bash
student@student-training:~/ansible_labs/lab11$ mkdir templates
```

##### Step 2 - Create the Jinja template

Create the empty `dhcp.j2` template in the `templates` directory:

```bash
student@student-training:~/ansible_labs/lab11$ touch templates/dhcp.j2
```

##### Step 3 - Update the template

In your text-editor of choice, add the following to your `dhcp.j2` template:

```jinja
{% for address in dhcp.excluded %}
ip dhcp excluded-address {{ address }}
{% endfor %}
!
{% for pool in dhcp.pools %}
ip dhcp pool {{ pool.name | upper }}
  network {{ pool.network }}
  default-router {{ pool.default_router }}
{% endfor %}
```

**In this template we are:**
* Looping through each **IP address** in the `dhcp` variable's `excluded` list.  We reference the `excluded` key of the `dhcp` variable by using dotted notation.
* Looping through each **pool** in the `dhcp` variable's `pools` list.  In this case, each pool in the list of pools is actually a dictionary, so we can reference each pool's **name, network, and default router** by using dotted notation.
* When we reference each pool's **name**, we are using the **upper** Jinja filter to convert it to UPPERCASE.

### Task 4 - Verifying Template Output

In this section, you will test the template and check the output to make sure it looks correct.  Later on, when you actually update device configurations using the `ios_config` module with your template, you will need to make sure the template output matches what a real device's configuration would look like.  This is so the `ios_config` module can appropriately determine if the target device's configuration is in compliance or not.

##### Step 1 - Create the playbook

Create a playbook called `lab11.yml` with the following contents:

```yaml
---
- name: Test Templates
  hosts: lab11
  gather_facts: false
  connection: local

  tasks:

    - name: Create Configs Directory
      file:
        path: configs
        state: directory

    - name: Test DHCP Template
      template:
        src: dhcp.j2
        dest: "./configs/{{ inventory_hostname }}.cfg"
```

##### Step 2 - Run the playbook

Test out your new playbook:

```bash
student@student-training:~/ansible_labs/lab11$ ansible-playbook lab11.yml -i inventory.yml

PLAY [Test Templates] ***********************************************************

TASK [Create Configs Directory] *************************************************
changed: [orko-lab11-red-router]
ok: [orko-lab11-yellow-router]

TASK [Test DHCP Template] *******************************************************
changed: [orko-lab11-red-router]
changed: [orko-lab11-yellow-router]

PLAY RECAP **********************************************************************
orko-lab11-red-router           : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab11-yellow-router        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

##### Step 3 - Verify the outputted files

Check the contents of one of the outputted config files to make sure it looks like this:

```bash
student@student-training:~/ansible_labs/lab11$ cat configs/orko-lab11-red-router.cfg
ip dhcp excluded-address 22.47.1.254
ip dhcp excluded-address 172.24.1.254
!
ip dhcp pool ORKO_DATA
  network 22.47.1.0 255.255.255.0
  default-router 22.47.1.254
ip dhcp pool ORKO_VOICE
  network 172.24.1.0 255.255.255.0
  default-router 172.24.1.254
```

You are now ready to actually use the `ios_config` module to send these templated configs to devices.


### Task 5 - Updating Device Configurations

Now that you have created the necessary variables and templates, you can now send the configuration to target devices using the `ios_config` module.

##### Step 1 - Create SSH variables

Add necessary group variables for SSH connection to `group_vars/all.yml`:

```yaml
ansible_user: orko
ansible_ssh_pass: P@ssw0rd
ansible_network_os: ios
```
>**Note:** The specific values you assign to these special variables will depend on your actual environment.

##### Step 2 - Update the playbook

Add **a new play** to your playbook:

```yaml
- name: Update Device DHCP Configurations
  hosts: routers
  gather_facts: false
  connection: network_cli

  tasks:
    - name: Update DHCP Configurations
      ios_config:
        src: dhcp.j2
```

##### Step 3 - Run the playbook

Run your playbook:

```bash
student@student-training:~/lab11/playbooks$ ansible-playbook lab11.yml -i inventory.yml 

PLAY [Test Templates] ********************************************************************************************************************************************************

TASK [Create Configs Directory] **********************************************************************************************************************************************
ok: [orko-lab11-red-router]
ok: [orko-lab11-yellow-router]

TASK [Test DHCP Template] ****************************************************************************************************************************************************
ok: [orko-lab11-red-router]
ok: [orko-lab11-yellow-router]

PLAY [Update Device DHCP Configurations] *************************************************************************************************************************************

TASK [Update DHCP Configurations] ********************************************************************************************************************************************
changed: [orko-lab11-red-router]
changed: [orko-lab11-yellow-router]

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab11-red-router           : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab11-yellow-router        : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Notice how each router was **changed**.

##### Step 4 - Run the playbook

Run your playbook again:

```bash
student@student-training:~/lab11/playbooks$ ansible-playbook lab11.yml -i inventory.yml

PLAY [Test Templates] ********************************************************************************************************************************************************

TASK [Create Configs Directory] **********************************************************************************************************************************************
ok: [orko-lab11-yellow-router]
ok: [orko-lab11-red-router]

TASK [Test DHCP Template] ****************************************************************************************************************************************************
ok: [orko-lab11-red-router]
ok: [orko-lab11-yellow-router]

PLAY [Update Device DHCP Configurations] *************************************************************************************************************************************

TASK [Update DHCP Configurations] ********************************************************************************************************************************************
ok: [orko-lab11-red-router]
ok: [orko-lab11-yellow-router]

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab11-red-router        : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab11-yellow-router        : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Now each router in the `PLAY RECAP` just says ***ok***, which demonstrates that the **ios_config** module uses idempotency.