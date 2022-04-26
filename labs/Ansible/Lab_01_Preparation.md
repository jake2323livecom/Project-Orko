# Lab 01 - Preparation

In this lab you will complete the prep necessary for future Ansible Labs.  

**In this lab you will:**
* Install Ansible
* Create all necessary directories

**Additional Resources:**
* [Bash for loop examples](https://www.cyberciti.biz/faq/bash-for-loop/)
* [Iterate over range of numbers in bash](https://www.cyberciti.biz/faq/unix-linux-iterate-over-a-variable-range-of-numbers-in-bash/)
* [Bash if statements](https://ryanstutorials.net/bash-scripting-tutorial/bash-if-statements.php)

### Task 1 - Install Ansible

PUT CONTENT HERE

### Create a necessary directories

##### Step 1 - Create parent directory

Create a directory that will serve as the parent directory for all future labs and `cd` into it:

```bash
student@student-training:~$ mkdir ansible_labs && cd ansible_labs
```

##### Step 2 - Create directories for each lab

Create a bash script that will create a directory for labs 1-15.

Your script might look something like this:

```bash
for num in {1..15}
do
        if [ $num -lt 10 ]
        then
                mkdir lab0$num
        else
                mkdir lab$num
        fi
done
```

After running your script, your parent directory should have the following:

```
student@student-training:~/ansible_labs$ tree
.
├── ansible_prep.sh
├── lab01
├── lab02
├── lab03
├── lab04
├── lab05
├── lab06
├── lab07
├── lab08
├── lab09
├── lab10
├── lab11
├── lab12
├── lab13
├── lab14
└── lab15

15 directories, 1 file
```

