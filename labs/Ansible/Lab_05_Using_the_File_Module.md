# Lab 05 - Using the file module

This lab highlights the use of the `file` module.  It allows you to set attributes on or delete files, symbolic links, and directories.

> **Note:** According to the documentation, this module should not be used to make changes on Windows systems.  For Windows, use the `win_file` module instead.

In this lab, you will only be making changes on your local machine, so no inventory will be needed.


**In this lab you will:**
* Create and modify directories and files
* Delete directories and files

**Resources:**
* [Writing Your First Playbook](https://www.ansible.com/blog/getting-started-writing-your-first-playbook)
* [Ansible File Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)


### Creating and modifying directories and files

##### Step 1 - Create the playbook

`cd` into the `ansible_labs/lab05` folder and create `lab05.yml`:

```bash
student@student-training:~$ cd ansible_labs/lab05 && touch lab05.yml
student@student-training:~/ansible_labs/lab05$ 
```

Open the file in your text editor.

The playbook will consist of a single play and a single task.


**On your own, attempt to create *one play* in the playbook that meets the following:**

* It should be called: 'USING THE FILE MODULE'
* It should not gather facts
* It should target the localhost
* The connection type will be local
* Contains a task list with a single task that debugs an arbitrary message of your choice.



**After writing the playbook yourself, compare your playbook with the following:**


```yaml
---
- name: USING THE FILE MODULE
  gather_facts: false
  hosts: localhost
  connection: local

  tasks:
    - name: Debug start message
      debug:
        msg: The playbook is starting...
```


##### Step 3 - Run the playbook

After running the playbook, you should see the following relevent output:

```
student@student-training:~/lab05$ ansible-playbook lab05.yml 
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [USING THE FILE MODULE] *******************************************************************************************************************************************

TASK [Debug start message] *********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "The playbook is starting..."
}

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


First, notice the warning we received at the beginning of the playbook run?  When running a playbook, if no inventory is provided, Ansible assumes we want to run the playbook against the localhost.  This means that the hosts key isn't really needed, but lets keep it just so that we are explicit with our intentions.

You will also notice from the output that our debug message was printed successfully.  The purpose of this task was just to test your playbook syntax.

##### Step 1 - Create a new directory

In your playbook, add the following task:

```yaml
    - name: Create a directory
      file:
        path: "{{ playbook_dir }}/my_new_directory/"
        state: directory
```

We are using the **playbook_dir** special variable.  This special variable is a method of using the playbook's *relative path*.

When using the file module, the **state** is assumed to be *file*, so if we want to create a directory, we must specifiy this in the **state** parameter.


##### Step 2 - Run the playbook

After running you playbook, you should see the following relevent output:

```
TASK [Create a directory] **********************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

If you view the contents of your playbook directory, you can see that the directory was created successfully:

```
student@student-training:~/lab05$ ll
total 16
drwxrwxr-x  3 student student 4096 Apr 20 12:34 ./
drwxr-xr-x 57 student student 4096 Apr 20 11:15 ../
-rw-rw-r--  1 student student  743 Apr 20 12:33 lab05.yml
drwxrwxr-x  2 student student 4096 Apr 20 12:34 my_new_directory/
```

##### Step 3 - Create a new file

In your playbook, add the following task:

```yaml
    - name: Create a file
      file:
        path: "{{ playbook_dir }}/my_new_directory/file_1.cfg"
        mode: '0744'
        state: touch
```

**There are a few things of note here:**
* We are using the **playbook_dir** special variable.  This special variable is a method of using the playbook's *relative path*.
* We are setting the permissions on the new file by passing the **mode** parameter.  The leading 0 in the mode is required so that Ansible's YAML parser knows that the number is an **octal** value.  Not including a leading 0 in this case would cause unintended results.
* If a file does not exist yet, you must set the **state** parameter to 'touch'
* The file module does allow you to pass the **owner** and **group** parameters, but if they are not supplied, Ansible will use the default **umask** of the target device.

##### Step 4 - Run the playbook

After running the playbook, you should see the following relevent output:

```bash

TASK [Create a file] ***************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

>**Note:** If you were to run the playbook a second time, the target device will still say *changed* even though the file already exists.  This has to do with how the file module is written.  In the background, the file is **not** being replaced.  

You should now see the new file in the directory that you just created:

```
student@student-training:~/lab05$ ll my_new_directory
total 8
drwxrwxr-x 2 student student 4096 Apr 20 12:41 ./
drwxrwxr-x 3 student student 4096 Apr 20 12:34 ../
-rwxr--r-- 1 student student    0 Apr 20 12:41 file_1.cfg*
```

##### Step 5 - Change permissions using symbolic modes

As of Ansible version 1.8, you can also use **symbolic modes** to specify permissions for a file.

Add the following task to your playbook:

```yaml
    - name: Change file permissions
      file:
        path: "{{ playbook_dir }}/my_new_directory/file_1.cfg"
        mode: a+x
```

This mode will *add* the executable permission to all parties: the owner, group, and all others.

##### Step 6 - Run the playbook

After running your playbook, you should see the following relevent output:

```
TASK [Create a file] ***************************************************************************************************************************************************
changed: [localhost]

TASK [Change file permissions] *****************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

If you view the contents of the directory, you can see the permissions have been changed:

```
-rwxr-xr-x  1 student student    0 Apr 20 12:12 file_1.cfg*
```

##### Step 7 - Change permissions of the directory recursively

In your playbook, add the following task:

```yaml
    - name: Give directory insecure permissions
      file:
        path: "{{ playbook_dir }}/my_new_directory"
        state: directory
        recurse: true
        mode: '0777'
```

##### Step 8 - Run the playbook

After running your playbook, you should see the following relevent output:

```
TASK [Give directory insecure permissions] *****************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Your directory should have full permissions now:

```
drwxrwxrwx  2 student student 4096 Apr 20 12:41 my_new_directory/
```

You should also see that since we set the **recurse** parameter to true, the file within the directory has full permissions as well:

```
-rwxrwxrwx 1 student student    0 Apr 20 12:48 file_1.cfg*
```

##### Step 9 - Change the owner and group

In your playbook, add a task that changes the owner and group of the file you created:

```yaml
    - name: Change owner and group to root
      file:
        path: "{{ playbook_dir }}/my_new_directory/file_1.cfg"
        owner: root
        group: root
```

Here we will attempt to set both the owner and group to the root user.  We did not have to pass the **state** because the default value of the parameter is *file*.  *file* is the state you'd want to use to modify an ***existing*** file.

##### Step 10 - Run the playbook

After running the playbook, you should see the following relevent output:

```
TASK [Create a file] ***************************************************************************************************************************************************
changed: [localhost]

TASK [Change owner and group to root] **********************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "gid": 1000, "group": "student", "mode": "0755", "msg": "chown failed: [Errno 1] Operation not permitted: b'/home/student/lab05/my_new_directory/file_1.cfg'", "owner": "student", "path": "/home/student/lab05/my_new_directory/file_1.cfg", "size": 0, "state": "file", "uid": 1000}

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0 
```

In this case, our new task failed because Ansible did not have permissions to make this change.  

##### Step 11 - Use root permissions

The method that Ansible uses to run operations with root privileges is called **become**.  When we tell Ansible to become, it assumes we want to *become* the **root** user, but you can actually become any other user.

There are several places to actually tell Ansible to become another user, but in our case we will do it at the *task level*.

Change the last task in the playbook to include the **become** task-level option:

```yaml
    - name: Change owner and group to root
      file:
        path: "{{ playbook_dir }}/my_new_directory/file_1.cfg"
        owner: root
        group: root
      become: true
```

>**Note:** The become option is at the task level.  It is **not** an parameter passed to the file module.

##### Step 7 - Run the playbook

In addition to adding the **become** option to our task, we will need to tell Ansible what password to use in order to become the new user.

We will specify the password by using the `--ask-become-pass` option in our `ansible-playbook` command.

This will give you an interactive prompt so that you can enter the required password for escalated privilages.  

>**Remember:** You can 'become' any user.  Ansible just assumes the root user as default.  You will have to enter the *specific password* for the user you wish to become.

```bash
student@student-training:~/lab05$ ansible-playbook lab05.yml --ask-become-pass
BECOME password: 
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [USING THE FILE MODULE] *******************************************************************************************************************************************

TASK [Debug start message] *********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "The playbook is starting..."
}

TASK [Create a file] ***************************************************************************************************************************************************
changed: [localhost]

TASK [Change owner and group to root] **********************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

If you view the contents of your directory, you can see the owner and group were changed to root:

```
-rwxrwxrwx 1 root root    0 Apr 20 12:57 file_1.cfg*
```

### Deleting directories and files

##### Step 1 - Delete the file

In your playbook, add the following task:

```yaml
    - name: Delete the file
      file:
        path: "{{ playbook_dir }}/my_new_directory/file_1.cfg"
        state: absent
      become: true
```

**IMPORTANT**

**Before running your playbook again, remove your directory completely with the `rm -rf my_new_directory` command otherwise the 'Create a file` task will fail.**

##### Step 2 - Run the playbook

After running your playbook, you should see the following relevent output:

```
TASK [Delete the file] *************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

If you list the contents of your directory, you should see the file is now gone.

##### Step 3 - Delete the directory

In your playbook, add the following task:

```yaml
    - name: Delete the directory
      file:
        path: "{{ playbook_dir }}/my_new_directory"
        state: absent
```

##### Step 4 - Run the playbook

After running your playbook, you should see the following relevent output:

```
TASK [Delete the directory] ********************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

If you list the contents of your playbook directory, you should see that the directory is now gone as well.

>**Note:** By default, directory contents are deleted recursively when deleting a directory.