# Lab 12 - Dynamic Inventory Scripts

In this lab you will be creating a dynamic inventory using a Python script.  The script will make an API call to your own Nautobot server in order to pull in device information for the `orko_lab12` site.

You will then use the output from the API call to create the hostvars portion of the inventory, and then you will create a few groups as well.  

**In this lab you will create a python script that:**
* Creates a basic Ansible inventory
* Executes an API call to Nautobot and pulls devices from a particular site
* Adds hostvars to each device in the inventory
* Adds groups to the inventory

**Additional Resources:**
* [Intermediate python requests usage](https://www.youtube.com/watch?v=p71SWzjeqtk)
* [Ansible Custom Inventory Scripts](https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible)
* [Creating custom dynamic inventories for Ansible](https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible)


### Task 1 - Writing a basic python script

##### Step 1 - Create the initial script

In the `lab12` directory, create a file called `inventory.py` and open it in your text editor.

**Type in the following contents:**

```python
#!/usr/bin/env python3

# Import necessary modules
import requests
import urllib3
import json

# Disable warnings due to self-signed SSL certificates
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Define API token and URL
nautobot_token = '99959315b2c64f730610cc44085840b4ff5d75be'
devices_url = 'https://192.168.128.15/api/dcim/devices'

# Define required HTTP headers
headers = {
    'Application': 'application/json',
    'Content-Type': 'application/json',
    'Authorization': f'Token {nautobot_token}'
}

# Execute API call
devices = requests.get(devices_url, headers=headers, verify=False).json()
```

**Here's what's going on here:**
* The first line in the script is called the **shebang**.  Ansible requires it for dynamic inventory scripts.
* We are importing the necessary modules
* We are disabling the warnings that will be the result of making an API call to an endpoint that is using self-signed certificates.
* We are defining the API token that we will use to authenticate as well as the specific URL that we will use.
* We are defining the headers that we want to put in our HTTP packet.  We use the typical values for the **Application** and **Content-Type** headers, and we specify the required value for the **Authorization** header as defined by the Nautobot documentation.
* Lastly, we are using the **requests** module to make a simple API call to Nautobot.  This API call is executing a GET request and is using all of the components that we defined previously.  This particular API call will pull all the **devices** from the Nautobot server.

If we run this script interactively, we can see that it returns X devices:

```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> devices['count']
388
>>> exit()
```

##### Step 2 - Allow Ansible to execute the script

We will have to make the script executable so that Ansible can use it:

```bash
student@student-training:~/ansible_labs/lab12$ sudo chmod a+x inventory.py
```


##### Step 3 - Filter devices by site

Now let's add the `site` parameter to our query so we only pull information for devices from a particular **site** in Nautobot. 

We do this by creating a `parameters` dictionary with each filter we want to apply to the query:
```python
parameters = {
    'site': 'orko_lab12'
}
```

We then utilize the `params` argument in the `requests.get()` function call:
```python
devices = requests.get(devices_url, params=parameters, headers=headers, verify=False).json()
```

Now if we run our script interactively again, we can see that only the devices from our intended site are returned:

```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> devices['count']
4
>>> exit()
```

Now that we have the information for our 4 intended devices, we can use the output from the API call to create the `hostvars` portion of our inventory.  

### Task 2 - Creating the Hostvars

In the inventory you will be creating in this lab, we want each host to have 2 hostvars, the `ansible_host` and `device_type`.  

In Nautobot, each device should have a primary IPv4 address.  This attribute is what we will use for the `ansible_host` variable.

For the `device_type` variable, we will use the device type that is specified in Nautobot.

In order to set these two variables for each device, we will need to know how to find the necessary information in the output of the API call.  Lets take a look at the JSON output for a single device so we can figure out how to reference the specific attributes we need.  

##### Step 1 - Isolate a single device from the API call result

Run your script interactively again.  Once in the Python terminal, create a variable that contains the information for a single device:

```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> device = devices['results'][0]
```

**DO NOT EXIT THE PYTHON TERMINAL**

##### Step 2: Retrieving a device's primary IPv4 address

First, lets find the primary IPv4 address for the device.  We can start by looking at the dictionary keys available for the device:

```
>>> device.keys()
dict_keys(['id', 'url', 'name', 'device_type', 'device_role', 'tenant', 'platform', 'serial', 'asset_tag', 'site', 'rack', 'position', 'face', 'parent_device', 'status', 'primary_ip', 'primary_ip4', 'primary_ip6', 'secrets_group', 'cluster', 'virtual_chassis', 'vc_position', 'vc_priority', 'comments', 'local_context_schema', 'local_context_data', 'tags', 'custom_fields', 'config_context', 'created', 'last_updated', 'display'])
```

From the output, we can see there is a key called `primary_ip4`.  This is the key that should contain the information pertaining to the primary IPv4 address for the device. Now let's see what information is contained in the `primary_ip4` key:

```
>>> type(device['primary_ip4'])
<class 'dict'>
```

We can see that the value of the `primary_ip4` key is a dictionary, so lets see what keys the dictionary contains:

```
>>> device['primary_ip4'].keys()
dict_keys(['id', 'url', 'family', 'address', 'display'])
```

If we want to see the entire structure of the `primary_ip4` key, we can use `pprint` to view the contents of the key in a formatted fashion:
```
>>> from pprint import pprint
>>> pprint(device['primary_ip4'])
{'address': '10.99.3.1/24',
 'display': '10.99.3.1/24',
 'family': 4,
 'id': 'abbee205-bbed-4cd6-b084-40496cf747bd',
 'url': 'https://192.168.128.15/api/ipam/ip-addresses/abbee205-bbed-4cd6-b084-40496cf747bd/'}
```

We can see from the output that the only keys that we could use to get the information we want would be the `address` and `display` keys.  We can use the `address` key in our script.  

In the next step, we will create the `hostvars` portion of our inventory and set the `ansible_host` variable for each device.

##### Step 3 - Create the 'hostvars'

In our python script, lets go ahead and create the empty `hostvars` dictionary:

```python
hostvars = {
    '_meta': {
        'hostvars': {}
    }
}
```

> For the `hostvars` portion of our inventory, Ansible requires us to embed the `hostvars` dictionary inside of another dictionary called `_meta`.  Structuring the `hostvars` in this manner tells Ansible to cache all `hostvars` so that it doesn't execute inventory lookups for each device individually during the course of a playbook's execution.  This saves on performance.

##### Step 4 - Populate the 'hostvars'

Now that we have an empty `hostvars` dictionary defined, we can loop through the devices returned from our API call and add each device to the `hostvars` dictionary:

```python
# Loop through each device in the API call results
for device in devices['results']:  

    # Update the hostvars dictionary with each device
    hostvars['_meta']['hostvars'].update(
        {
            # The device name should be the parent key.  Sub-keys will be individual variables.
            device['name']: {

                # Set the ansible_host variable to the device's primary IPv4 address using the necessary keys
                'ansible_host': device['primary_ip4']['address']
            }
        }
    )
```

**Here we are using the `.update()` dictionary method to add each device to the `hostvars` dictionary.**


##### Step 5 - Create an empty `inventory` dictionary and add the 'hostvars'

Now lets create an empty `inventory` dictionary and add the `hostvars` variable to it.  Later on, we will add our groups to the `inventory` variable as well.  For now it will just include our hostvars:

```python
inventory = {}
inventory.update(hostvars)
```

##### Step 6 - Output the inventory in JSON

Ansible will require us to output the inventory in a `JSON` format.  We can do this by using the `json` python module.

Add the following to your script:

```python
print(json.dumps(inventory, indent=4))
```

Now run your script and take a look at the output:

```bash
student@student-training:~/ansible_labs/lab12$ python3 inventory.py
{
    "_meta": {
        "hostvars": {
            "orko-lab12-red-router": {
                "ansible_host": "10.99.3.1/24"
            },
            "orko-lab12-yellow-router": {
                "ansible_host": "10.99.3.2/24"
            },
            "orko-lab12-red-switch": {
                "ansible_host": "10.99.3.3/24"
            },
            "orko-lab12-yellow-switch": {
                "ansible_host": "10.99.3.4/24"
            }
        }
    }
}
```

**You can see that each device's hostname becomes a key in the `hostvars` dictionary, and each device has been given the `ansible_host` variable.**

##### Step 7 - Create the 'all' group

Ansible requires that all hosts in the inventory belong to at least one group.  Since we haven't defined any groups yet, lets go ahead and create the `all` group and add every host to it.

We will create a `groups` variable, add the `all` group to it, and then add the `groups` variable to the `inventory` variable at the end of the script just above where we print the inventory as JSON:


```python
groups = {
    'all': {
        'hosts': []
    }
}

for host in hostvars['_meta']['hostvars'].keys():
    groups['all']['hosts'].append(host)

inventory.update(groups)

# This line should already be there
print(json.dumps(inventory, indent=4))
```

**In this code you are:**
* Creating a `groups` variable.  In order to create the `all` group, we create a key called `all`, and then add the `hosts` key to the `all` dictionary.
* Looping through each device in our hostvars dictionary and appending the device's hostname to the `hosts` list for the `all` group.
* Combining the `groups` variable with our master inventory variable using the `.update()` dictionary method.

Now, if you run your script interactively and `pprint` the `inventory`, we should see the following structure:

```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> from pprint import pprint
>>> pprint(inventory)
{'_meta': {'hostvars': {'orko-lab12-red-router': {'ansible_host': '10.99.3.1/24'},
                        'orko-lab12-yellow-router': {'ansible_host': '10.99.3.2/24'},
                        'orko-lab12-red-switch': {'ansible_host': '10.99.3.3/24'},
                        'orko-lab12-yellow-switch': {'ansible_host': '10.99.3.4/24'}}},
 'all': {'hosts': ['orko-lab12-red-router',
                   'orko-lab12-yellow-router',
                   'orko-lab12-red-switch',
                   'orko-lab12-yellow-switch']}}
```


##### Step 8 - Run a playbook

Now lets test our inventory against a playbook.  Create a playbook called `lab12.yml` and give it the following contents:

```yaml
---
- name: Debug Inventory
  hosts: all
  gather_facts: false
  connection: local

  tasks:
    - name: Debug Variables
      debug:
        var: ansible_host
```

> In this playbook we are targeting the `all` group.  

Run the playbook to verify the `ansible_host` variable was set for each device:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook lab12.yml -i inventory.py

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-red-router] => {
    "ansible_host": "10.99.3.1/24"
}
ok: [orko-lab12-yellow-router] => {
    "ansible_host": "10.99.3.2/24"
}
ok: [orko-lab12-red-switch] => {
    "ansible_host": "10.99.3.3/24"
}
ok: [orko-lab12-yellow-switch] => {
    "ansible_host": "10.99.3.4/24"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-red-router        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-router     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-red-switch        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-switch     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

##### Step 9 - Remove the CIDR from the `ansible_host` variable

Normally the `ansible_host` variable should be set to just an IP address, not an IP address with subnet mask like we have currently.  We can use some logic in our inventory script to get rid of the subnet mask.  

Change the value of the `ansible_host` key in your script as follows:

```python
                'ansible_host': device['primary_ip4']['address'].split('/')[0]
```

##### Step 10 - Demonstration

Lets see how this code would work with a sample IP address with CIDR mask.

Open a python terminal and run the following:

```
>>> a = '10.10.10.1/32' # Create initial IP address
>>>
>>> a.split('/') # Split the 'a' variable on the forward slash.  This creates a list with two elements.
['10.10.10.1', '32']
>>> a.split('/')[0] # Grab the first element of the outputted list
'10.10.10.1'
```

**Here, in a python terminal, we can see that when we split our initial IP address on the `/` character, it creates a *list* of two elements:**
* The IP address
* The CIDR notation subnet mask.  

You can then select the 0 index of the list to return the IP address **without the CIDR notation subnet mask.**

##### Step 10 - Run the playbook

Run your playbook again:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook ansible-playbooks lab12.yml -i inventory.py 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-red-router] => {
    "ansible_host": "10.99.3.1"
}
ok: [orko-lab12-yellow-router] => {
    "ansible_host": "10.99.3.2"
}
ok: [orko-lab12-red-switch] => {
    "ansible_host": "10.99.3.3"
}
ok: [orko-lab12-yellow-switch] => {
    "ansible_host": "10.99.3.4"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-red-router        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-router     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-red-switch        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-switch     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Notice that **now** the `ansible_host` variable is just the IP address without the CIDR notation subnet mask.

At this point, your inventory script should match the following:

```python
#!/usr/bin/env python3

# Import necessary modules
import requests
import urllib3
import json

# Disable warnings due to self-signed SSL certificates
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Define API token and URL
nautobot_token = '99959315b2c64f730610cc44085840b4ff5d75be'
devices_url = 'https://192.168.128.15/api/dcim/devices'

# Define required HTTP headers
headers = {
    'Application': 'application/json',
    'Content-Type': 'application/json',
    'Authorization': f'Token {nautobot_token}'
}

# Define site parameter
parameters = {
    'site': 'orko_lab12'
}

# Execute API call
devices = requests.get(devices_url, params=parameters, headers=headers, verify=False).json()

# Create hostvars structure
hostvars = {
    '_meta': {
        'hostvars': {}
    }
}

# Add each device to hostvars
for device in devices['results']:
    hostvars['_meta']['hostvars'].update(
        {
            device['name']: {
                'ansible_host': device['primary_ip4']['address'].split('/')[0]
            }
        }
    )

# Create empty master inventory
inventory = {}

# Add hostvars to the inventory
inventory.update(hostvars)

# Create group structure
groups = {
    'all': {
        'hosts': []
    }
}

# Add each host to the all group
for host in hostvars['_meta']['hostvars'].keys():
    groups['all']['hosts'].append(host)

# Add the groups to the master inventory
inventory.update(groups)

# Print the inventory to STDOUT
print(json.dumps(inventory, indent=4))
```

##### Step 11 - Add the `device_type` host-level variable

Now lets add the `device_type` variable for each device.  First, we will need to take a look at the output of our API call to find out how we can reference each device's `device_type`.

Comment out the last line of our inventory script:

```python
# print(json.dumps(inventory, indent=4))
```



Now run your script interactively and isolate a single device to see what **keys** it has:
```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> device = devices['results'][0]
>>> device.keys()
dict_keys(['id', 'url', 'name', 'device_type', 'device_role', 'tenant', 'platform', 'serial', 'asset_tag', 'site', 'rack', 'position', 'face', 'parent_device', 'status', 'primary_ip', 'primary_ip4', 'primary_ip6', 'secrets_group', 'cluster', 'virtual_chassis', 'vc_position', 'vc_priority', 'comments', 'local_context_schema', 'local_context_data', 'tags', 'custom_fields', 'config_context', 'created', 'last_updated', 'display'])
```

If we look at the keys available for the first device in the API call results, we can see that there is a `device_type` key.  Let's use `pprint` to take a look at what it contains:

```bash
>>> from pprint import pprint
>>> pprint(device['device_type'])
{'display': 'orko router',
 'id': 'ce759dd3-7ded-4a62-84f3-2758cf967df8',
 'manufacturer': {'display': 'orko',
                  'id': '8842cd3c-4176-439b-9d42-6271ff222947',
                  'name': 'orko',
                  'slug': 'orko',
                  'url': 'https://192.168.128.15/api/dcim/manufacturers/8842cd3c-4176-439b-9d42-6271ff222947/'},
 'model': 'router',
 'slug': 'router',
 'url': 'https://192.168.128.15/api/dcim/device-types/ce759dd3-7ded-4a62-84f3-2758cf967df8/'}
```

> In Nautobot, the `display` key actually references the `manufacturer + model`, not just the **model** itself.  In this case, we actually want the `model` key, which is just the model and doesn't include the manufacturer.

From the output, we can see that for each device, we will need to reference `device['device_type']['model']` in order to set the `device_type` variable for each device.

Lets **update** our python code to add the `device_type` variable to each device:

```python
for device in devices['results']:   # Loop through each device in the API call results

    # Update the hostvars dictionary with each device
    hostvars['_meta']['hostvars'].update(
        {
            device['name']: {

                # Set the ansible_host variable to the device's primary IPv4 address using the necessary keys
                'ansible_host': device['primary_ip4']['address'].split('/')[0],

                # Set the device_type variable
                'device_type': device['device_type']['model'],
            }
        }
    )
```

Uncomment the last line of your script
```python
print(json.dumps(inventory, indent=4))
```

##### Step 12 - Update the playbook

Now lets edit our playbook to debug the `device_type` variable instead of `ansible_host`:

```yaml
---
- name: Debug Inventory
  hosts: all
  gather_facts: false
  connection: local

  tasks:
    - name: Debug Variables
      debug:
        var: device_type

```

##### Step 13 - Run the playbook

Run your playbook again:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook lab12.yml -i inventory.py 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-red-router] => {
    "device_type": "router"
}
ok: [orko-lab12-yellow-router] => {
    "device_type": "router"
}
ok: [orko-lab12-red-switch] => {
    "device_type": "switch"
}
ok: [orko-lab12-yellow-switch] => {
    "device_type": "switch"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-red-router        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-router     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-red-switch        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-switch     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

From the output you can see that we have successfully debugged the new `device_type` variable for each device.

### Task 3 - Creating groups

Now that we have gotten our hostvars squared away, we can start creating some groups to allow us to assign variables at the group level.

In this section, you will create two more groups.  Each group will contain devices with a different `platform` as it is defined in Nautobot.  The two platforms we will create groups for are the `orko_red` and `orko_yellow` platforms.  

> In our production Nautobot server, we use the `platform` to indicate the enclave of the device.  That's why the platforms in your lab Nautobot server have `red` and `yellow` in the names.

##### Step 1 - Create an empty `orko_red` group

In this step, we will create a group that contains all devices that have a platform of `orko_red`.

In your inventory script, replace the `groups` variable with the following:

```python
groups = {
    'all': {
        'hosts': []
    },
    'orko_red': {
        'hosts': []
    }
}
```

**Here we are adding a key to the existing `groups` dictionary.**  The key will be the name of our group, `orko_red`, and the value of this key will be a dictionary.  This dictionary will have a `hosts` key, with the value being an empty list. 


##### Step 2 - Isolate platform information on a single device

In order to add hostnames to the new group, we will have to loop through the devices in the results of our API call, and add each device to the new group if the device platform's name is `orko_red`.

Before we do that, however, we need to take a look at the data structure to see how to refence the data we need.

Lets comment out the last line of our inventory script and then run it interactively:

```python
# print(json.dumps(inventory, indent=4))
```

```bash
student@student-training:~/ansible_labs/lab12$ python3 -i inventory.py
>>> device = devices['results'][0]
>>> device.keys()
dict_keys(['id', 'url', 'name', 'device_type', 'device_role', 'tenant', 'platform', 'serial', 'asset_tag', 'site', 'rack', 'position', 'face', 'parent_device', 'status', 'primary_ip', 'primary_ip4', 'primary_ip6', 'secrets_group', 'cluster', 'virtual_chassis', 'vc_position', 'vc_priority', 'comments', 'local_context_schema', 'local_context_data', 'tags', 'custom_fields', 'config_context', 'created', 'last_updated', 'display'])
```

We can see from the output that each device should have a `platform` key.  Lets use `pprint` to see what the `platform` key contains:

```
>>> from pprint import pprint
>>>
>>> pprint(device['platform'])
{'display': 'orko_red',
 'id': '65573532-7693-4ed4-8b69-cc18d09610de',
 'name': 'orko_red',
 'slug': 'orko_red',
 'url': 'https://192.168.128.15/api/dcim/platforms/65573532-7693-4ed4-8b69-cc18d09610de/'}
```

From the output we can see that there are actually three keys we can reference within the `platform` key: display, name, and slug.  For our script, lets use the `name` key.  

##### Step 3 - Add hosts to the `orko_red` group

Add the following to your python script just above where you add the `groups` variable to the master inventory:

```python
for device in devices['results']:
    if device['platform']['name'] == 'orko_red':
        groups['orko_red']['hosts'].append(device['name'])

inventory.update(groups)
```

This code loops through all of the devices returned in our API call, and **IF** the device platform's `name` is equal to `orko_red`, then we add the ***device's*** `name` to the host list of the `orko_red` group in our `groups` variable.

##### Step 4 - Print the JSON output

Uncomment the last line of your inventory script:

```python
print(json.dumps(inventory, indent=4))
```

##### Step 5 - Update the playbook

Now, we can verify the correct hosts were added to our new group.  

Edit your playbook so that it targets the new `orko_red` group instead of the `all` group:

```yaml
hosts: orko_red
```

##### Step 6 - Run the playbook

Run your playbook:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook lab12.yml -i inventory.py 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-red-router] => {
    "device_type": "router"
}
ok: [orko-lab12-red-switch] => {
    "device_type": "switch"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-red-router            : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-red-switch            : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

We can see from the output that the playbook was only run against the devices in our new `orko_red` group.

The playbook correctly ran against our two red devices only.

##### Step 7 - Create the `orko_yellow` group

Now lets finish our inventory script by creating a group for our yellow devices as well.

Replace the `groups` variable with the following:

```python
groups = {
    'all': {
        'hosts': []
    },
    'orko_red': {
        'hosts': []
    },
    'orko_yellow': {
        'hosts': []
    }
}
```

As you can see, just like before, we are just creating a new group called `orko_yellow` and giving it an empty hosts list.

##### Step 8 - Add hosts to the `orko_yellow' group

Now add the following loop to your script in the appropriate spot so that we can add devices to our new group:

```python
for device in devices['results']:
    if device['platform']['name'] == 'orko_yellow':
        groups['orko_yellow']['hosts'].append(device['name'])
```

##### Step 9 - Update the playbook

Now edit your playbook to target the new `orko_yellow` group:

```yaml
hosts: orko_yellow
```

##### Step 10 - Run the playbook

Run your playbook:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook lab12.yml -i inventory.py 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-yellow-router] => {
    "device_type": "router"
}
ok: [orko-lab12-yellow-switch] => {
    "device_type": "switch"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-yellow-router         : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-switch         : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

You can see from the output that we successfully ran the playbook only against the `orko_yellow` group.

##### Step 10 - Remove all hosts from the `all` group

If you remember, after we created the `all` group, we simply added every device from the API call to that group under the `hosts` key.  Now that we have all of our devices under other groups, we can get rid of the `hosts` key altogether.  

In your inventory script, **remove** the following code:

```python
for host in hostvars['_meta']['hostvars'].keys():
    groups['all']['hosts'].append(host)
```

Then, **replace** your `groups` variable with the following:

```python
groups = {
    'all': {
        'children': [
            'orko_red',
            'orko_yellow'
        ]
    },
    'orko_red': {
        'hosts': []
    },
    'orko_yellow': {
        'hosts': []
    }
}
```

Here, instead of using the previous for-loop to add each device to the `hosts` key of the `all` group, we are just creating a `children` key under the `all` group, and adding each child group name to the list.

##### Step 11 - Run the playbook

Run your playbook again to ensure it still runs as intended:

```bash
student@student-training:~/ansible_labs/lab12$ ansible-playbook lab12.yml -i inventory.py 

PLAY [Debug Inventory] *******************************************************************************************************************************************************

TASK [Debug Variables] *******************************************************************************************************************************************************
ok: [orko-lab12-red-router] => {
    "device_type": "router"
}
ok: [orko-lab12-red-switch] => {
    "device_type": "switch"
}
ok: [orko-lab12-yellow-router] => {
    "device_type": "router"
}
ok: [orko-lab12-yellow-switch] => {
    "device_type": "switch"
}

PLAY RECAP *******************************************************************************************************************************************************************
orko-lab12-red-router      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-red-switch      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-router   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
orko-lab12-yellow-switch   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```