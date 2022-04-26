# Creating API calls with Python

### Using the Requests module

In order to execute queries to remote API endpoints, you will use the Requests module.  There are many other python packages that can execute API queries, 
but they typically use the Requests module as a dependency and just add some custom functionality.

**In this lab, you will learn how to:**
* Execute basic GET requests using the Requests module
* Filter query output using parameters

**Prerequisites:**
* Internet connectivity
* Python3 is installed on your machine
* The requests and urllib3 python modules have been installed using PIP 

You will be sending API calls to the demo Nautobot server available via the internet.  The demo server contains sample data we can pull using API calls.


Here is the link to the [demo nautobot server](https://demo.nautobot.com)

**Additional Resources:**
* [Nautobot Swagger](https://demo.nautobot.com/api/docs/)
* [Nautobot API Overview](https://nautobot.readthedocs.io/en/stable/rest-api/overview/)
* [Basic python requests usage](https://www.youtube.com/watch?v=hpc5jyVpUpw)
* [Intermediate python requests usage](https://www.youtube.com/watch?v=p71SWzjeqtk)
* [F-strings in python](https://www.geeksforgeeks.org/formatted-string-literals-f-strings-python/)

##### Step 1: Creating a basic script

Create a file called `lab02.py` and open it in VScode.

Type the following into the file:

```python

# Import necessary packages/modules
import requests
import urllib3

# Disable warnings that occur due to the API endpoint using a self-signed certificate
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Define the base url for all API calls
api_url = 'https://demo.nautobot.com/api/'

# Define a specific url pattern for querying devices
devices_url = api_url + 'dcim/devices/'


# Use the requests module to execute a query and save the result
devices = requests.get(devices_url, verify=False)

```

Lets run the script using the `-i` flag so we can explore the data after running the script:

```

student@student-training: python3 -i nautobot.py
>>>
>>> type(devices)
<class 'requests.models.Response'>
>>>
>>> devices
<Response [403]>

```

**There are two things of note here:**

* By default, the Requests module will use the response from the API endpoint and turn it into a `Response` object
* Response type objects, by default, will be represented by their HTTP status code, not the actual information returned from the API call.  In our case, our response was an HTTP 403, which means authentication isn't working as intended. We can fix this in the next step.

##### Step 2: Using the `Authorization` header

The Nautobot API requires the use of an API token.  The demo Nautobot server has a default superuser called `demo` with an API token of `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`.

We will use this token to authenticate ourselves to the API endpoint and give us authorization to retrieve different data sets.

**Add the following to your python script just below where we define our URLs:**

```python
nautobot_token = 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'

headers = {
    'Application': 'application/json',
    'Content-Type': 'application/json',
    'Authorization': f'Token {nautobot_token}'
}

```

This headers dictionary is what Requests will use to actually manipulate what goes into the HTTP headers in the request packet.

The actual value of the `Authorization` header is **dependent upon the specific API endpoint you are accessing**. In this case, the Nautobot API documentation states that the `Authorization` field should have the word `Token ` and then the actual API token you'd like to use.

In our example, we use an *f-string* to create the appropriate value for the `Authorization` header value.

The other two headers, `Application` and `Content-Type`, are outside the scope of this lab.  For now, just include these headers as a default anytime you make an API call.

Then add our headers dictionary as an argument in our API call:

**Before:**
```python
devices = requests.get(devices_url, verify=False)
```

**After:**
```python
devices = requests.get(devices_url, headers=headers, verify=False)
```

Now lets re-run our script using the interactive mode again:

```python
student@student-training: python3 -i nautobot.py
>>> devices
<Response [200]>
```

Now we can see our response was returned successfully due to the code 200.  In the next step we will learn how to actually access the json data within the response.


##### Step 3: Exploring response data

Now that we have successfully queried the Nautobot API, we will need to output the actual json data within it.

Response type objects have a few different methods for viewing their data, but we will use the `.json()` method.

> The .json() method will create a *python data structure* from the Response data

Replace the following line in your script:

**Before:**
```python
devices = requests.get(devices_url, headers=headers, verify=False)
```

**After:**
```python
devices = requests.get(devices_url, headers=headers, verify=False).json()
```

Now lets run our script interactively again and explore the data:

```
student@student-training: python3 -i nautobot.py
```

1. We can see that devices is actually a python dictionary, thanks to the `.json()` method we used.

```
>>> type(devices)
<class 'dict'>
>>>
```

2. We can use the `.keys()` dictionary method to see that devices has 4 keys. 

```
>>> devices.keys()
dict_keys(['count', 'next', 'previous', 'results'])
```

> Keep in mind that the structure of the returned data for any given API will be dependent upon how the API was designed.

3. Using `type()`, we can see that `devices['results']` is a list.  Using `len()`, we can also see that the length of this list is `50`.

```
>>> type(devices['results'])
<class 'list'>
>>>
>>> len(devices['results'])
50
```

4. We can see that the first item in the results list is a dictionary.  In this case, `devices['results']` is a list of devices, where each device is represented as a dictionary.  

```
>>> type(devices['results'][0])
<class 'dict'>
```

5. Using the `.keys()` dictionary method again, we can see the information available for each device.

```
>>> devices['results'][0].keys()
dict_keys(['id', 'url', 'name', 'device_type', 'device_role', 'tenant', 'platform', 'serial', 'asset_tag', 'site', 'rack', 'position', 'face', 'parent_device', 'status', 'primary_ip', 'primary_ip4', 'primary_ip6', 'secrets_group', 'cluster', 'virtual_chassis', 'vc_position', 'vc_priority', 'comments', 'local_context_schema', 'local_context_data', 'tags', 'custom_fields', 'config_context', 'created', 'last_updated', 'display'])
```

6. Lastly, we can import the pprint module from the pprint package, and use it to print out one of the devices in a formatted fashion in the terminal:

```python
>>> from pprint import pprint
>>>
>>> pprint(devices['results'][0])
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
 'created': '2022-03-08',
 'custom_fields': {},
 'device_role': {'display': 'distribution',
                 'id': 'ab0878f6-2d20-4bf0-9b8d-eacbee5c6ca5',
                 'name': 'distribution',
                 'slug': 'distribution',
                 'url': 'https://demo.nautobot.com/api/dcim/device-roles/ab0878f6-2d20-4bf0-9b8d-eacbee5c6ca5/'},
 'device_type': {'display': 'Cisco Catalyst 6509-E',
                 'id': '160e052f-09ad-4b69-b94c-a7dbc2033b00',
                 'manufacturer': {'display': 'Cisco',
                                  'id': '1943f761-d4a3-45d2-814f-f3623d613789',
                                  'name': 'Cisco',
                                  'slug': 'cisco',
                                  'url': 'https://demo.nautobot.com/api/dcim/manufacturers/1943f761-d4a3-45d2-814f-f3623d613789/'},
                 'model': 'Catalyst 6509-E',
                 'slug': 'catalyst-6509-e',
                 'url': 'https://demo.nautobot.com/api/dcim/device-types/160e052f-09ad-4b69-b94c-a7dbc2033b00/'},
 'display': 'ams01-dist-01',
 'face': None,
 'id': '56b14e70-dd45-4fef-8a58-b5d0fc8bfde6',
 'last_updated': '2022-03-08T14:57:40.679517Z',
 'local_context_data': None,
 'local_context_schema': None,
 'name': 'ams01-dist-01',
 'parent_device': None,
 'platform': {'display': 'Cisco IOS',
              'id': 'dd203154-d827-4362-ae64-b7968f6f4936',
              'name': 'Cisco IOS',
              'slug': 'cisco_ios',
              'url': 'https://demo.nautobot.com/api/dcim/platforms/dd203154-d827-4362-ae64-b7968f6f4936/'},
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
 'tenant': {'display': 'Nautobot Airports',
            'id': '418181c0-6b76-45dc-bf63-a16192fb4b80',
            'name': 'Nautobot Airports',
            'slug': 'nautobot-airports',
            'url': 'https://demo.nautobot.com/api/tenancy/tenants/418181c0-6b76-45dc-bf63-a16192fb4b80/'},
 'url': 'https://demo.nautobot.com/api/dcim/devices/56b14e70-dd45-4fef-8a58-b5d0fc8bfde6/',
 'vc_position': None,
 'vc_priority': None,
 'virtual_chassis': None}
```



# Filtering with parameters

### Passing parameters to the requests module

We can use parameters to filter the data returned from an API endpoint.  We can either pass a parameters dictionary to the Requests module, or we can use parameters in the request URL itself.  

In this example, we will first filter the devices returned by site, and then further filter that dataset and return only devices with a particular `platform`

First, add this parameters dictionary to your script just below the headers dictionary:

```python
parameters = {
    'site': 'ams01'
}
```

Then add the `params` argument to the line that executes our API call:


Before:

```python
devices = requests.get(devices_url, headers=headers, verify=False).json()
```

After:

```python
devices = requests.get(devices_url, params=parameters, headers=headers, verify=False).json()
```


Now lets re-run our script in interactive mode and explore the results:

**We can see that the devices variable has the same keys as before, but when we look at the `count` key, we can see we only have 11 results instead of 50:**

```
>>> devices.keys()
dict_keys(['count', 'next', 'previous', 'results'])
>>> devices['count']
11
```

Now lets add a second parameter to our API call.  Add the following `'platform': 'arista_eos'` key:value pair to our parameters dictionary:

```python
parameters = {
    'site': 'ams01',
    'platform': 'arista_eos'
}
```

After re-running the script interactively, we can see that the count of devices returned is now down to 10.  This tells us that there are only 10 devices in the Nautobot database that reside within the site `ams01` and also have a platform called `arista_eos`:

```
>>> devices['count']
10
```

### Using parameters in the request URL

As an alternative to passing parameters to the requests module function call, we can simply include parameters in the request URL itself.

In your script, do the following:

1. Comment out your `parameters` dictionary

```python
# parameters = {
#     'site': 'ams01',
#     'platform': 'arista_eos'
# }
```

2. Create a `devices_params` variable that will contain the portion of our URL that includes the query parameters

```python
devices_params = '?site=ams01&platform=arista_eos'
```
> Parameters begin in the url after a question mark.  Key/value mappings are separated by an equal sign.  Each parameter is separated by an ampersand

3. Append the `devices_params` variable to the `devices_url` variable

```python
devices_url = api_url + 'dcim/devices/' + devices_params
```

3. Remove the `params` argument from your API call

```python
devices = requests.get(devices_url, headers=headers, verify=False).json()
```

This new URL will apply the same parameters, but using the url instead.  Since we are using the url, we don't have to include the `params` argument.

After running the script again, we can see that we still have 10 results, indicating the parameters worked as expected.


