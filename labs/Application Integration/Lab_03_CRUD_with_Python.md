# Lab 03 - Managing objects using API calls

In this lab you will be using API calls to create, read, update, and delete devices on the [demo Nautobot server](https://demo.nautobot.com).

You will create a separate function for each CRUD operation.

Before you begin coding, open a web browser to the [demo Nautobot server](https://demo.nautobot.com).

Navigate to `Organization > Sites > AMS01 > Devices`

### Task 1 - Creating a device in Nautobot

In this section, you will create a function that can create a new device in Nautobot.  The Nautobot API documentation defines required attributes that must be supplied when creating a device, and your function will take all of these attributes as arguments. 

**Required attributes:**
* device_type
* device_role
* site
* status

> You will use an http POST request to create new devices.  POST requests are typically the method used when **making changes on the API endpoint.**

##### Step 1: Create a basic script

Create a file called `lab03.py` and open it in VScode.

Type the following into the file:

```python

# Import necessary packages/modules
import requests
import urllib3
from pprint import pprint

# Disable warnings that occur due to the API endpoint using a self-signed certificate
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Define the base url for all API calls
api_url = 'https://demo.nautobot.com/api/'

# Define a specific url pattern for querying devices
devices_url = api_url + 'dcim/devices/'

# Use the requests module to execute a query and save the result
result = requests.get(devices_url, headers=headers, verify=False).json()   # Use the .json() method to retrieve json data from the Response object

```

Add the necessary headers dictionary and token to the script to make it work. Once you are able to successfully pull all devices, proceed to the next step.

##### Step 2: Create a function to create a device

You will use a function that takes in arguments and builds a `body` dictionary that you will pass in the API call.  The `body` is required in any API call where you are passing data to the API endpoint.  

**Add the following function and function call to your script:**

```python
def device_create(name, device_type, device_role, site, status):
    
    body = {
        'name': name,
        'device_type': device_type,
        'device_role': device_role,
        'site': site,
        'status': status
    }

    result = requests.post(devices_url, headers=headers, json=body, verify=False).json()
    return result

new_device = device_create('abc-newdevice-01', 'ca43499a-a1c0-49ae-af5b-69ad8623f911', '241550c5-f128-4560-86ba-df8bb0482c5c', '2f370622-d518-49a7-b03e-1fa9fb764ed2', 'active')

device_uuid = new_device['id']

```

**A few things of note here:**
* Even though the `name` isn't a required field, an empty name doesn't comply with the naming standard configured on the demo Nautobot server, so we need to supply one in this case.
* Unfortunately, Nautobot requires you to refer to the device_type, device_role, and site by UUID, not by name which would be more convenient.  
* If a device with the parameters you provide already exists, no new object will be created

**Run your script interactively and make sure the `device_uuid` variable was set:**

```
student@student-training:~$ python3 -i lab03.py
>>> 
>>> device_uuid
55eb0fa1-93f8-4b51-a20b-5fa45babf31d
```

Now if you refresh the page in your web browser, you should see your new device.  

> Make sure you create the `device_uuid` variable as shown in the code above.  You will use it in the next steps.

### Task 2 - Retrieving a single device from Nautobot

According to Nautobot's API documentation, in order to manipulate a single device in the database, we need to use the device's UUID in the URL.  

For this section, you will use a GET request along with the UUID of the device you just created.


##### Step 1: Creating a function to retrieve data for a single device

Create a function called `read_device()`.  It will take in a device's UUID and return the JSON data for that single device.

> The Nautobot API documentation states that in order to retrieve a single device's information, use the following url: `api/dcim/devices/{id}/`, where `id` is referring to the device's UUID.

```python
def read_device(uuid):
    device_url = api_url + f'dcim/devices/{uuid}/'
    device = requests.get(device_url, headers=headers, verify=False).json()
    return device

```

> Ensure there is a trailing forward slash at the end of your url


Run your script interactively and then call your function, passing it the `device_uuid` variable you created previously, and make sure output is returned:

```
student@student-training:~$ python3 -i lab03.py
>>>
>>> pprint(read_device(device_uuid))
{'asset_tag': None,
 'cluster': None,
 'comments': '',
 'config_context': {'acl': {'definitions': {'named': {'PERMIT_ROUTES': ['10 '
                                                                        'permit '
                                                                        'ip '
                                                                        'any '
                                                                        'any']}}},
                    'route-maps': {'PERMIT_CONN_ROUTES': {'seq': 10,
                                                          'statements': ['match '
                                                                         'ip '
                                                                         'address '
                                                                         'PERMIT_ROUTES'],
                                                          'type': 'permit'}}},
 'created': '2022-04-04',
 'custom_fields': {},
 'device_role': {'display': 'edge',
                 'id': '241550c5-f128-4560-86ba-df8bb0482c5c',
                 'name': 'edge',
                 'slug': 'edge',
                 'url': 'https://demo.nautobot.com/api/dcim/device-roles/241550c5-f128-4560-86ba-df8bb0482c5c/'},
 'device_type': {'display': 'Arista DCS-7280CR2-60',
                 'id': 'ca43499a-a1c0-49ae-af5b-69ad8623f911',
                 'manufacturer': {'display': 'Arista',
                                  'id': '832a9d3f-3d7e-40ec-b665-c4f1f056ccfd',
                                  'name': 'Arista',
                                  'slug': 'arista',
                                  'url': 'https://demo.nautobot.com/api/dcim/manufacturers/832a9d3f-3d7e-40ec-b665-c4f1f056ccfd/'},
                 'model': 'DCS-7280CR2-60',
                 'slug': 'dcs-7280cr2-60',
                 'url': 'https://demo.nautobot.com/api/dcim/device-types/ca43499a-a1c0-49ae-af5b-69ad8623f911/'},
 'display': 'abc-newdevice-01',
 'face': None,
 'id': 'c27b7e0c-63b4-4336-9945-2d4bcaf94134',
 'last_updated': '2022-04-04T01:35:40.716841Z',
 'local_context_data': None,
 'local_context_schema': None,
 'name': 'abc-newdevice-01',
 'parent_device': None,
 'platform': None,
 'position': None,
 'primary_ip': None,
 'primary_ip4': None,
 'primary_ip6': None,
 'rack': None,
 'secrets_group': None,
 'serial': '',
 'site': {'display': 'AMS01',
          'id': '2f370622-d518-49a7-b03e-1fa9fb764ed2',
          'name': 'AMS01',
          'slug': 'ams01',
          'url': 'https://demo.nautobot.com/api/dcim/sites/2f370622-d518-49a7-b03e-1fa9fb764ed2/'},
 'status': {'label': 'Active', 'value': 'active'},
 'tags': [],
 'tenant': None,
 'url': 'https://demo.nautobot.com/api/dcim/devices/c27b7e0c-63b4-4336-9945-2d4bcaf94134/',
 'vc_position': None,
 'vc_priority': None,
 'virtual_chassis': None}
```

### Task 3 - Edit an existing device

For this step, you will create a function that uses a PATCH request to update an existing device.  We will update the same device that we pulled in the previous section.

Your function will only be able to edit the name of the device and no other attributes.

> Normally when using an API call to edit a single object, you will have to reference the target object using it's unique identifier, UUID in this case.

##### Step 1:  Create a function to edit a single device


```python
def update_device(uuid, name):
    device_url = api_url + f'dcim/devices/{uuid}/'

    body = {
        'name': name
    }

    result = requests.patch(device_url, json=body, headers=headers, verify=False).json()
    return result

```

**Two things of note here:**
* You have to use this specific device name because the demo Nautobot server is setup to only accept a specified device naming standard.  If you supply a name that doesn't comply to the standard, you will receive an error message.
* Here, we use the `json=` argument in our call to the requests module so that we can supply information in our body.  The body dictionary will contain any values we want to edit.  In this example, we only update the `name` field of the device.

**Run your script interactively again, and test out your update function:**

```
student@student-training:~$ python3 -i lab03.py
>>>
>>> pprint(update_device(device_uuid, 'xyz-mynewdevice-99'))
{'asset_tag': None,
 'cluster': None,
 'comments': '',
 'config_context': {'acl': {'definitions': {'named': {'PERMIT_ROUTES': ['10 '
                                                                        'permit '
                                                                        'ip '
                                                                        'any '
                                                                        'any']}}},
                    'route-maps': {'PERMIT_CONN_ROUTES': {'seq': 10,
                                                          'statements': ['match '
                                                                         'ip '
                                                                         'address '
                                                                         'PERMIT_ROUTES'],
                                                          'type': 'permit'}}},
 'created': '2022-04-04',
 'custom_fields': {},
 'device_role': {'display': 'edge',
                 'id': '241550c5-f128-4560-86ba-df8bb0482c5c',
                 'name': 'edge',
                 'slug': 'edge',
                 'url': 'https://demo.nautobot.com/api/dcim/device-roles/241550c5-f128-4560-86ba-df8bb0482c5c/'},
 'device_type': {'display': 'Arista DCS-7280CR2-60',
                 'id': 'ca43499a-a1c0-49ae-af5b-69ad8623f911',
                 'manufacturer': {'display': 'Arista',
                                  'id': '832a9d3f-3d7e-40ec-b665-c4f1f056ccfd',
                                  'name': 'Arista',
                                  'slug': 'arista',
                                  'url': 'https://demo.nautobot.com/api/dcim/manufacturers/832a9d3f-3d7e-40ec-b665-c4f1f056ccfd/'},
                 'model': 'DCS-7280CR2-60',
                 'slug': 'dcs-7280cr2-60',
                 'url': 'https://demo.nautobot.com/api/dcim/device-types/ca43499a-a1c0-49ae-af5b-69ad8623f911/'},
 'display': 'xyz-mynewdevice-99',
 'face': None,
 'id': 'c27b7e0c-63b4-4336-9945-2d4bcaf94134',
 'last_updated': '2022-04-04T01:42:12.309084Z',
 'local_context_data': None,
 'local_context_schema': None,
 'name': 'xyz-mynewdevice-99',
 'parent_device': None,
 'platform': None,
 'position': None,
 'primary_ip': None,
 'primary_ip4': None,
 'primary_ip6': None,
 'rack': None,
 'secrets_group': None,
 'serial': '',
 'site': {'display': 'AMS01',
          'id': '2f370622-d518-49a7-b03e-1fa9fb764ed2',
          'name': 'AMS01',
          'slug': 'ams01',
          'url': 'https://demo.nautobot.com/api/dcim/sites/2f370622-d518-49a7-b03e-1fa9fb764ed2/'},
 'status': {'label': 'Active', 'value': 'active'},
 'tags': [],
 'tenant': None,
 'url': 'https://demo.nautobot.com/api/dcim/devices/c27b7e0c-63b4-4336-9945-2d4bcaf94134/',
 'vc_position': None,
 'vc_priority': None,
 'virtual_chassis': None}
```

> Notice from the output that the name of our device has indeed changed.  You can also refresh your web browser to see the change.  

### Task 4 - Delete a single device

In this section you will use an HTTP DELETE request to delete the device you created in the first section. 

You will create a function that takes in a device's UUID as an argument.

#### Step 1: Create a function to delete a single device

Add the following to your script: 

```python
def delete_device(uuid):
    device_url = api_url + f'dcim/devices/{uuid}/'
    result = requests.delete(device_url, headers=headers, verify=False)
    return result

```
> In this delete function, we aren't using the `.json()` method on the Response object.  The Response object produced by a DELETE request won't produce json output.  In this case, we just want the HTTP status code.  When you print the output of the function call, you should receive a `<Response [204]>`, which indicates the resource was successfully deleted.


Run your script interactively and execute the following commands:

```
student@student-training:~$ python3 -i lab03.py
>>>
>>> print(delete_device(device_uuid))
<Response [204]>
>>>
>>> pprint(read_device(device_uuid))
{'detail': 'Not found.'}
```

Here we see that when we delete our device, we receive an HTTP status code of 204.

We also see that when we try to get the data for our deleted device, the API says the resource doesn't exist.

