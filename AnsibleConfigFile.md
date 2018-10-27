### Ansible Config File

1. Changes can be made and used in a configuration file which will be processed in the following order:

```
* ANSIBLE_CONFIG (an environment variable)
* ansible.cfg (in the current directory)
* .ansible.cfg (in the home directory)
* /etc/ansible/ansible.cfg

![image](../media/ansible.cfg.png)

2. Config File Keys

## ANSIBLE_HOST_KEY_CHECKING

It bypass the yes/no verfication while adding the remote host in known_hosts during SSH.

```
	host_key_checking = False

```
![host_key_checking](../media/host_key_checking.png)

## Become, Become_pass

hosts inventory file

centos1 ansible_user=root -> sshing in remote host as root user (copying the root password in authorized keys)

ubuntu1 ansible_become=true ansible_become_pass=password-> means sshing as user then becoming a root user.

![image](../media/become%2Cbecome_pass.png)

## ansible_port

used to change the ssh default port to run on other port.

``` 
	centos1 ansible_user=root

               OR

	centos1:2222

```
## Grouping 

![group-range](../media/group-range.png)

## Grouping & Vars & Children

[centos]

centos1 ansible_port=2222 
centos[2:3] -> host-range

[centos:vars]
ansibel_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true 
ansible_become_pass=password

SO, [ubuntu:vars] will apply to all ubuntu group. Similary for centos

![children](../media/children.png)

## Precedence

ansible_port=2222 will have higher precedence than the ansible_port=1234 because specific variable takes precedence over others.


![precedence](../media/precednce-result.png)

So, if we ping all it should ping only centos1 and failed others(port=1234 is not a valid port number).

## Another way to run yaml.

![yml](../media/yml.png)


![json-yaml](../media/json-yaml.png)

### FILE MODULE

Sets attributes of files, symlinks, and directories, or removes files/symlinks/directories.Many other modules support the same options as the file module - including copy, template, and assemble.

* Parameters

 - path : Path of the file being managed

 - state: 
	Choices:

	        * absent
		* directory
		* hard
		* link
		* touch
		* file

* touch 

```
          $ ansible aws -m file -a 'path=/tmp/sample state=touch'

          This create a file to the defined location mentioned in path attribute . If the sample file is alredy present
          then also it will not give error like file is already present. 
	
	  If the file in this case sample is edited and then again touch command is executed the new file(sample) is  created 
	  with the previous content. (Note : See the TimeStamp)
	
       
     ![touch](../)

     ![TS](../)

```

* file ←

```
	$ ansible aws -m file -a 'path=/home/sample state=file mode=600'

        Above the file permission will not be changed if the file does not exist in the given location.

![file](../)

```
# Interesting facts:- 

Idempotency:- An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly 
	      without any intervning actions.

GREEN = Success

YELLOW = Success with Changes 

RED = Failure

![color](../)


### FETCH MODULE

It copy the file from remote to localhost. While fetching a file it creates a directory with the name of file.

$ ansible aws -m fetch -a "src=/tmp/sample dest=/tmp/sample"

OUTPUT Directory will be :-

![fetch](../)

![fetch-loc](../)


### Ansible Playbooks, Breakdown of Section

```
---
 # YAML documents begin with the document separator ---

 # The minus in YAML this indicates a list item.  The playbook contains a list 
 # of plays, with each play being a dictionary
 -

  # Target: where our play will run and options it will run with

	hosts: aws
        user:  root

  # Variable: variables that will apply to the play, on all target systems

    vars:
      motd: "Welcome to CentOS Linux - Ansible Rocks\n"

  # Task: the list of tasks that will be executed within the play, this section
  # can also be used for pre and post tasks
	
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        src: centos_motd
        dest: /etc/motd

      copy:
        content: "Hi Ansible"                                               // This content will get copied to the defined location
        dest:     /etc/motd

   - name: Configure a MOTD (message of the day)
      copy:
        content: "{{ motd }}"
        dest: /etc/motd

  # Handlers: handlers that are executed as a notify key from a task

  # Roles: list of roles to be imported into the play

 # Three dots indicate the end of a YAML document
...

```

# HANDLERS

```
---
 # YAML documents begin with the document separator ---

 # The minus in YAML this indicates a list item.  The playbook contains a list 
 # of plays, with each play being a dictionary
-

  # Target: where our play will run and options it will run with
  hosts: centos
  user: root
  gather_facts: false

  # Variable: vvariables that will apply to the play, on all target systems
  vars:
    motd: "Welcome to CentOS Linux - Ansible Rocks\n"

  # Task: the list of tasks that will be executed within the playbook
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "{{ motd }}"
        dest: /etc/motd
      notify: MOTD changed

  # Handlers: handlers that are executed as a notify key from a task
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed

  # Roles: list of roles to be imported into the play

 # Three dots indicate the end of a YAML document
...

```
# Execute task based on Platform

```

---
 # YAML documents begin with the document separator ---
 
 # The minus in YAML this indicates a list item.  The playbook contains a list
 # of plays, with each play being a dictionary
 -
 
  # Target: where our play will run and options it will run with
  hosts: linux
 
  # Variable: variables that will apply to the play, on all target systems
  vars:
    motd_centos: "Welcome to CentOS Linux - Ansible Rocks\n"
    motd_ubuntu: "Welcome to Ubuntu Linux - Ansible Rocks\n"
 
  # Task: the list of tasks that will be executed within the playbook
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "{{ motd_centos }}"
        dest: /etc/motd
      notify: MOTD changed
      when: ansible_distribution == "CentOS"

    - name: Configure a MOTD (message of the day)
      copy:
        content: "{{ motd_ubuntu }}"
        dest: /etc/motd
      notify: MOTD changed
      when: ansible_distribution == "Ubuntu"
 
  # Handlers: handlers that are executed as a notify key from a task
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
 
  # Roles: list of roles to be imported into the play
 
 # Three dots indicate the end of a YAML document
...

```

## ANSIBLE Playbooks,Variables

* vars_prompt:- This will prompt to ask the username while executing the playbook.
		Prompts are executed in the beginning of every play, it doesn't matter where you place vars_prompt block - before or after tasks section.

```
---
 # YAML documents begin with the document separator ---
 
 # The minus in YAML this indicates a list item.  The playbook contains a list
 # of plays, with each play being a dictionary
 -
 
  # Target: where our play will run and options it will run with
  hosts: centos1
  gather_facts: false
 
  # Variable: variables that will apply to the play, on all target systems
  vars_prompt:
    - name: username
 
  # Task: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test vars_prompt
      debug:
        msg: "{{ username }}"
 
 # Three dots indicate the end of a YAML document
...
	
```

![vars_prompt]()

when used with private attribute equal to false as:-
 vars_prompt:
       name: username
       private: false

![private]()


* Magic Variables, and How To Access Information About Other Hosts

Even if you didn’t define them yourself, Ansible provides a few variables for you automatically. The most important of these are 
 > hostvars, group_names,groups, environment.
 
  Users should not use these names themselves as they are reserved.hostvars lets you ask about the variables of another host, including facts that have been gathered about that host. 


### ANSIBLE Playbook Module

Register and when 









  

     




















