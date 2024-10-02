
## IAM: Identity access management

### Authentication & Authorization

- With Username and password -> authentication
- As AWS has 1000's of services, if a new user is created he will not have admin access rather he has limited access i.e based on the permissions assigned to the user
- Each user can have certain permissions
- In every software system, each user has the following:
  - Username and Password
  - User Group
  - User Permissions
- AWS services also should have certain permissions, so that they can't create resources automatically
- Using AWS Command line utlility, we can create and manage resources on our AWS account
- Roles: Its for non humans, which should be attached to the AWS services so that they can create resources based on our request.
- When creating role, we need to choose the following:
  1. For which service --> EC2
  2. What are its permissions --> AdministratorAccess

## Scripts for creating instances automatically and updating Route53 records

- We can create EC2 instances using AWS CLI: `aws ec2 run-instances --image-id ami-03265a0778a880afb --instance-type t2.micro --security-group-ids sg-039d04b205a0ea7db`
- We can perform query on this using JSON Query (jq) to fetch the Private IP address of the instances: `aws ec2 run-instances --image-id ami-03265a0778a880afb --instance-type t2.micro --security-group-ids sg-039d04b205a0ea7db | jq -r '.Instances[0].PrivateIpAddress'`

## Disadvantanges with Shell Scripting

1. Homogenous i.e. script is dependent on distribution of OS
2. Validation is necessary at each step
3. If we have 1k servers, we need to login and run the script manually
4. Imperative approach

### Imperative vs Declarative

- Imperative code focuses on writing an explicit sequence of commands to describe how you want the computer to do things
  - E.g. Shell Scripting
- Declarative code focuses on specifying the result of what you want, simpler syntax
  - E.g. Ansible, Terraform

## Push vs Pull Mechanism

- With Push mechanism, server pushes the configuration to the nodes
- With Pull mechanism
  - Nodes connect with the server (called as Configuration server) and pull the configuration from the server
  - Schedule how frequent the nodes should connect to the server for which we need to have an **agent** software installed on the nodes
  - Even though change in configuration is very rare, node needs to stay in connected with the server due to which the resources are wasted

## Ansible

- With Ansible, we can connect to any number of servers without explicit login
- We have one main server (called as Configuration server) and all the other servers we call them as Nodes on which we can execute our code
- Ansible connects to the nodes remotely using SSH mechanism
- We can also connect with SSH to the remote server using: `sshpass -p 'DevOps321' ssh centos@<IP address>`
  - In addition to that, we can also create files on the remote server without logging in: `sshpass -p 'DevOps321' ssh centos@<IP address> touch /tmp/test`
- Ansible works on Push Mechanism where as Chef, Puppet works on Pull Mechanism
- We can install Ansible using: `sudo yum install ansible -y`
- The instance on which Ansible is installed is called **Ansible server**
- We can use command line utility to execute commands on our nodes for e.g. `ansible -i inv all -e ansible_user=centos -e ansible_password=DevOps321 -b -m ansible.builtin.yum -a "name=nginx"` can be used to install NGiNX on remote node present inside inv file
- Similar to Shell Script, if we have all the ansible related commands inside a file, it is called ansible playbook
- Ansible playbooks are written using YAML (Yet Another Markup language)



## Inventory

- List of servers that Ansible is managing
- It is always recommended to logically group servers based on:
  - Geography -> IN, US, UK, AU, etc
  - Environment -> DEV, QA, PRE-PROD, UAT, PROD
  - Component -> web, app, db
  - Server -> mongodb, cart, catalogue
- For e.g.
  - roboshop-us-dev-db-mongodb-01
  - roboshop-us-dev-db-mongodb-02 (For high availability)
- To check if a DNS record is updated or not, we can use: `nslookup <domain-address>`
  - For e.g. `nslookup mongodb.learninguser.online`


# Sample Inventory File
web1 ansible_host=server1.company.com
web2 ansible_host=server2.company.com
web3 ansible_host=server3.company.com

# Sample Inventory File
  
# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# DB Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Dbp@ss123!

```

- To fetch the hosts that are under a group inside the inventory: `ansible -i inventory mongodb --list-hosts`
- To fetch the hosts that are ungrouped inside the inventory: `ansible -i inventory ungrouped --list-hosts`
- To fetch all the hosts (grouped + ungrouped) inside the inventory: `ansible -i inventory all --list-hosts`
- To prompt for a password when running an ansible adhoc command: `ansible -i inventory mongodb -m ping -u centos --ask-pass`
- Task: Installing NGiNX on the remote server without logging in
  - yum install nginx -y --> command
  - ansible --> module/collection
  - `module/collection name <parameters>`
- To install: `ansible -i inventory mongodb --become -e ansible_user=centos -e ansible_password=DevOps321 -m ansible.builtin.yum -a "name=nginx state=installed"`
  - `--become` or `-b`: To use sudo previliges
  - `-a`: arguments
  - `-e`: extra arguments
- To start the service: `ansible -i inventory mongodb -b -e ansible_user=centos -e ansible_password=DevOps321 -m ansible.builtin.service -a "name=nginx state=started"`
  - yellow color: It successfully changed the configuration on the server
  - Green color: The changes were already made on the server
- Adhoc: Something emergency i.e. to perform a quick check
- Ansible playbooks consists of list of ansible adhoc commands that are written and managed using YAML syntax

## YAML

- Yet Another Markup Language
- Some other Markup languages are:
  - XML: Extensible Markup Language
  - HTML: Hypertext Markup language
- Indentation needs to be proper else it will throw errors
- YAML consists of 3 data types majorly:
  1. Single Key-value pair: E.g. `name: Pavan Kumar`
  2. Map:

      ```yaml
        address:
          city: hyd
          country: india
          dno: 102
          street: ganghi nagar
      ```

  3. List

      ```yaml
        addresses:
        - type: perm
          city: hyd
          country: india
        - type: current
          city: california
          country: US
      ```



```yaml
# Playbook is list of plays, always it should start with -
- name: ping the node
  hosts: mongodb
  # this is list of tasks/modules/collections
  tasks:
  - name: pinging the server
    # this is the map
    ansible.builtin.ping:
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 01-playbook.yaml`

## Variables

- Play level variables
- Task level variables
- From files
- From Command prompt
- From inventory file



```yaml
- name: variables in ansible
  hosts: mongodb
  # This is Play level variables, map
  vars:
    COURSE: DevOps with AWS
    TRAINER: 
    DURATION: 
  tasks:
  - name: print hello world
    ansible.builtin.debug:
      msg: "Hello, I am learning Ansible"
  - name: print variables
    ansible.builtin.debug:
      msg: "Hello, I am learning {{COURSE}}, Trainer is {{TRAINER}}, Duration is {{DURATION}}"

```
# Ansible: Variables, Datatypes and Conditions



### Vars from files

`ansible/04-vars-files.yaml`

```yaml
- name: variables from files
  hosts: localhost #managing the ansible server itself
  vars_files:
  - variables.yaml
  tasks:
  - name: printing variables
    ansible.builtin.debug:
      msg: "We are learning {{NAME}}, Duration is: {{DURATION}}, Trainer is: {{TRAINER}}"
```

`ansible/variables.yaml`

```yaml
NAME: DevOps with AWS
DURATION: 
TRAINER:
```

### Values to Vars during runtime

`ansible/05-vars-prompt.yaml`

```yaml
- name: variables from prompt
  hosts: localhost
  vars_prompt:
  - name: USERNAME
    prompt: Please enter your username
    private: false # you can see the value entered on screen
  - name: PASSWORD
    prompt: Please enter your password
    private: true # you can't see the value entered on screen
  tasks:
  - name: print variable values
    ansible.builtin.debug:
      msg: "username: {{USERNAME}}, password: {{PASSWORD}}"
```

### Task level variables

- Can inherit the variables from play level
- Task level can extend and override these variables

`ansible/06-task-level.yaml`

```yaml
- name: variables at task level
  hosts: localhost
  # these variables of parent or play level
  vars:
  - money: "10000 RS"
    land: "100 hectars"
  tasks:
  - name: inherit values from play
    ansible.builtin.debug:
      msg: "MONEY: {{money}}, LAND: {{land}}"
  - name: inherit values from play and add and override
    vars:
    - money: "200000 RS"
      houses: "3 houses"
    ansible.builtin.debug:
      msg: "MONEY: {{money}}, LAND: {{land}}, houses: {{houses}}"
```

- To run the playbook: `ansible-playbook 06-task-level-yaml`

### From inventory file

`ansible/07-inventory.yaml`

```yaml
- name: variables from inventory
  hosts: mongodb
  tasks:
  - name: print mongodb username
    ansible.builtin.debug:
     msg: "username is: {{MONGO_USERNAME}}"
```

`ansible/inventory`

```ìnventory
[mongodb]
172.31.42.158

[mongodb:vars]
MONGO_USERNAME=mongodbadmin
MONGO_DB=categories
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 01-playbook.yaml`

### From command line

`ansible/08-command_line.yaml`

```yaml
- name: variables from command line
  hosts: localhost
  tasks:
  - name: print variable from command line
    ansible.builtin.debug:
    msg: "The value of variable course is {{COURSE}}"
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 -e COURSE="DevOps with AWS" 08-command_line.yaml`

### Variable Precedence

- 1st preference is command line args
- 2nd preference is task level
- 3rd preference is from vars_files
- 4th preference is from prompt
- 5th preference is from play level
- 6th preference is from inventory

## Data Types

1. Number
2. String
3. List
4. Map
5. Boolean

`ansible/data_types.yaml`

```yaml
- name: ansible variable data types 
  hosts: localhost
  vars:
    - AGE: 30 # Number
    - NAME: "" # String
    - isDevOps: true # Boolean
    - Skills: # List
      - DevOps
      - AWS
      - Docker
    - EXPERIENCE: # Map
        DevOps: 7
        AWS: 5
        Docker: 4
  tasks:
    - name: print number variable
      ansible.builtin.debug:
        msg: "{{AGE}}"
    - name: print String variable
      ansible.builtin.debug:
        msg: "{{NAME}}"
    - name: print Boolean variable
      ansible.builtin.debug:
        msg: "{{isDevOps}}"
    - name: print List variable
      ansible.builtin.debug:
        msg: "{{Skills}}"
    - name: print Map variable
      ansible.builtin.debug:
        msg: "{{EXPERIENCE}}"
```

## Conditions

`ansible/simple_condition.yaml`

```yaml
- name: simple condition
  hosts: localhost
  vars:
    NAME: DevOps1
  tasks:
  - name: run this if name is DevOps
    ansible.builtin.debug:
      msg: "Hello {{NAME}}"
    when: NAME == "DevOps"
```

- We can register variables to store the output of a command
- when we dont have an equivalent module for some linux commands, we use `ansible.builtin.shell` or `ansible.builtin.command`
- when there is a failure at the time of command execution, ansible stops the execution of playbook at that task itself
- We can also ignore the erros by setting: `ignore_errors: true`

```yaml
- name: create user
  hosts: localhost
  tasks:
    - name: check roboshop user exist or not
      ansible.builtin.command: id roboshop
      register: out # out is variable name
      ignore_errors: true
      
    - name: print exit status
      ansible.builtin.debug:
        msg: "{{out.rc}}"
    
    - name: create user roboshop
      become: yes # sudo access for this task only
      ansible.builtin.user:
        name: roboshop
      when: out.rc != 0
```
# Ansible: Loops

Date: 30-08-2023

## Loops

- We use `item` as our iterable to access elements inside a loop

`ansible/loops.yaml`

```yaml
- name: loops example
  hosts: localhost # you no need to give user name and password through ansible command line
  tasks:
  - name: print the names
    ansible.builtin.debug:
      msg: "Hello {{item}}"
    loop:
    - Raheem
    - John
```
## Difference between Shell and Command module

- With command module, we can only perform simple tasks
- With shell module, we can have access to the environment variables, user variables i.e. as if we're inside the server

## Difference between infrastructure and configuration

- In infrastructure, we create servers, route53 records, VPCs, security groups etc
- In configuration, we install packages and their dependencies
- With configuration management such as Ansible, we can also create EC2 servers but it is not at all recommended.


- In order to maintain DRY principle, we would like to take all the code that is common across modules and create functions from it
- In Ansible, functions can be implemented using Roles
- Ansible Roles consists of certain folder structure in which we can place the files such as `tasks/main.yaml` is responsible for executing the list of tasks that are in that role

  ```yaml
  - name: Installing Roboshop project
    hosts: "{{component}}"
    become: yes
    roles:
      - "{{component}}"
  ```

- All the tasks that are common across all the components are kept inside `common` role
- We can import that role using:

  ```yaml
  - name: Install NodeJS
    ansible.builtin.import_role:
      name: common
      tasks_from: nodejs
  ```

- When we want to replace a text with some other text, we can use placeholders using Jinja 2 syntax
- To place the content in the content-holder, we use templates in Ansible as shown below and use `ansible.builtin.template` module to serve this purpose

  `roles/web/templates/roboshop.conf.j2`

  ```conf
  proxy_http_version 1.1;
  location /images/ {
  expires 5s;
  root   /usr/share/nginx/html;
  try_files $uri /images/placeholder.jpg;
  }

  location /api/catalogue/ { proxy_pass http://{{CATALOGUE_HOST}}:8080/; }
  location /api/user/ { proxy_pass http://{{USER_HOST}}:8080/; }
  location /api/cart/ { proxy_pass http://{{CART_HOST}}:8080/; }
  location /api/shipping/ { proxy_pass http://{{SHIPPING_HOST}}:8080/; }
  location /api/payment/ { proxy_pass http://{{PAYMENT_HOST}}:8080/; }

  location /health {
  stub_status on;
  access_log off;
  }
  ```

  `roles/web/templates/roboshop.conf.j2`

  ```yaml
  - name: copy roboshop configuration
    ansible.builtin.template:
      src: roboshop.conf.j2
      dest: /etc/nginx/default.d/roboshop.conf
  ```

- The values are fetched from `vars/main.yaml` file and used at the time of playbook execution

  `roles/web/vars/main.yaml`

  ```yaml
  CATALOGUE_HOST: catalogue.joindevops.online
  USER_HOST: user.joindevops.online
  SHIPPING_HOST: shipping.joindevops.online
  CART_HOST: cart.joindevops.online
  PAYMENT_HOST: payment.joindevops.online
  ```

- Or we can also define `variables.yaml` in main directory where `main.yaml` file is present and pass it to the `main.yaml` as shown below using `vars_files`

  `variables.yaml`

  ```yaml
  - name: "install {{component}}"
    hosts: "{{component}}"
    vars_files:
      - variables.yaml
    become: yes
    roles:
      - "{{component}}"
  ```

## Handlers

- If we want to trigger a function when ever there is change in a file, we can use **handlers** as shown below

  `roles/web/handlers/main.yaml`

  ```yaml
  - name: restart nginx
    service:
      name: nginx
      state: restarted
      enabled: yes
  ```

  `roles/web/tasks/main.yaml`

  ```yaml
  - name: copy roboshop.conf
    ansible.builtin.template:
      src: roboshop.conf.j2
      dest: /etc/nginx/default.d/roboshop.conf
    notify:
      - restart nginx
  ```
  # Miscelleneous topics in Ansible

Date: 05-09-2023

- If we check `ansible --version`, we can get the path to the Ansible configuration and it is present at `/etc/ansible/ansible.cfg`
- Rather than passing the username and password inorder to connect to the server using command line, we can use `ansible.cfg` configuration file to serve this purpose
- To define an environment variable that is valid for a particular session, we use: `export`
- To remove an environment variable, we use: `unset` command

  ```bash
  export a=10
  unset a
  ```

## Configuration precedence in Ansible

1. ANSIBLE_CONFIG -> ENV variable:
    - `export ANSIBLE_CONFIG=/home/centos/test/ansible.cfg`
    - To unset the `ANSIBLE_CONFIG` variable: `unset ANSIBLE_CONFIG`
2. Current working directory
3. User home directory, it should be `.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

- To generate an example `ansible.cfg` file with all the options, we can run: `ansible-config init --disabled > ansible.cfg`
- For e.g.

  `ansible.cfg`

  ```cfg
  [defaults]
  inventory=inventory.ini
  ansible_user=centos
  ```

- If we specify the password inorder for ansible to connect to the servers inside this file, it will not work
- Rather we should define it inside `inventory.ini`

  `inventory.ini`

  ```ini
  [catalogue]
  catalogue.learninguser.shop
  [all:vars]
  ansible_password=DevOps321
  ```

- Now, when we want to run the playbook for catalogue component, we don't need to specify the username and password as its already present inside ansible.cfg file in the current directory
- `ansible-playbook -e component=catalogue main.yaml`

### Ansible tags

- Tags are useful when re-running the playbook for e.g. when a  new release of the artifact is available
- This way, we can specify which particular plays needs to be executed as show below

  `tags/catalogue.yaml`

  ```yaml
  - name: Install catalogue component
  hosts: catalogue
  become: yes
  tasks:
  - name: setup NPM source
    tags:
      - installation
    ansible.builtin.shell: "curl -sL https://rpm.nodesource.com/setup_lts.x | bash"

  - name: Install NodeJS
    tags:
      - installation
    ansible.builtin.yum:
      name: nodejs
      state: installed

  - name: download catalogue artifact
    tags:
      - deployment
    ansible.builtin.get_url:
      url: https://roboshop-builds.s3.amazonaws.com/catalogue.zip
      dest: /tmp
  ```

  `tags/inventory`

  ```inventory
  [catalogue]
  172.31.42.176
  ```

  `tags/ansible.cfg`

  ```cfg
  [defaults]
  remote_user = centos
  private_key_file = /home/centos/devops.pem
  inventory = inventory
  ```

- To execute this playbook, we run the command: `ansible-playbook catalogue.yaml -t deployment` and the tasks that has the tag `deployment` will only be executed.
- Tags in Ansible can be used at any level i.e. Task level, play level etc

## Inbuilt functions in Ansible

- Functions in Ansible are referred as filters in ansible

  `filters/demo.yaml`

  ```yaml
  - name: DEMO on filters
    hosts: localhost
    vars:
      website: https://www.joindevops.com/batch-74s
    tasks:
    - name: extract hostname
      debug:
        msg: "{{ website | urlsplit('hostname') }}"

    - name: set default value if not defined
      debug:
        msg: "{{ COURSE | default('DevOps') }}"
  ```

## Facts in Ansible

- Facts are the information that ansible collects at the time of execution
- For e.g. using these facts, we can identify in which distribution is ansible executing the commands
- This information is useful, when we want to run the script in heterogeneous environment i.e. different OS flavours
- Even though, we never have different OS, OS version, Packages and its versions, services and their status, directory, permissions, etc. in DEV and PROD environments

  `heterogenous/facts.yaml`

  ```yaml
  - name: understand facts
    hosts: centos:ubuntu #all
    become: yes
    tasks:
    - name: print all the facts
      ansible.builtin.debug:
        msg: "All Facts: {{ansible_facts}}"

    - name: add user ubuntu
      ansible.builtin.command: adduser sivakumar
      when: ansible_facts['distribution'] == "Ubuntu"

    - name: add user centos
      ansible.builtin.command: useradd sivakumar
      when: ansible_facts['distribution'] == "CentOS"
  ```

  `heterogenous/inventory`

  ```inventory
  [centos]
  172.31.47.193 ansible_user=centos

  [ubuntu]
  172.31.46.88 ansible_user=ubuntu
  ```

  `heterogenous/ansible.cfg`

  ```cfg
  [defaults]
  remote_user = centos
  private_key_file = /home/centos/devops.pem
  inventory = inventory
  ```

- `ansible_user=centos` are said to be Host variables and using `hosts: centos:ubuntu`, we can specify more than one group of servers to select and run the playbook

## Dynamic Inventory

- This is necessary to fetch resources information from cloud providers such as AWS
- To work with AWS EC2 plugin on ansible, the files should have `aws_ec2.yaml` extension
- For example to fetch all the EC2 instances with the name `web`, we can use the following:

  `web.aws_ec2.yaml`

  ```yaml
  plugin: amazon.aws.aws_ec2
  regions:
    - us-east-1
  filters:
    tag:Name:
      - web
  ```

- To work with this, we should have `botocore` and `boto3` modules installed
- Then we can run: `ansible-inventory -i web.aws_ec2.yaml --list` to fetch the list of `web` hosts
- Then to run a command on all the hosts, we can run: `ansible aws_ec2 -i web.aws_ec2.yaml -e ansible_user=centos -e ansible_password=DevOps321 -m ping`
- We can use `%d` in vim to clear the file contents

## Ansible Vault

- So far passwords to the servers are being provided using commmand prompt or using `ansible.cfg` file which is insecure, rather we can use Ansible vault for this purpose
- A vault is a storage of secrets
- Encoding: A proper pattern to encode the text
- Encryption: Encrypt a text using an algorithm + input password. For e.g. AES256
- So far we have been passing values to the variables to the components that are part of a group i.e. web group using the inventory file
- Instead we can also create a folder named `group_vars`
- To create Ansible vault: `ansible-vault create </path/somename.yaml>`
  - For e.g. `ansible-vault create group_vars/web.yaml`
  - We should enter a password: For e.g. admin123
- To edit the contents of the vault: `ansible-vault edit group_vars/web.yaml`

  `vault/ansible.cfg`

  ```cfg
  [defaults]
  inventory = inventory
  ask_vault_pass = True
  ```

  `vault/inventory`

  ```ini
  [web]
  web.learninguser.shop
  ```

  `vault/01-playbook.yaml`

  ```yaml
  - name: ping playbook
    hosts: web
    tasks:
    - name: ping the server
      ansible.builtin.ping:
  ```

- To use the same variables with values for all the groups, we use: `group_vars/all.yaml`
- To encrypt an existing file, we use: `ansible-vault encrypt group_vars/all.yaml`
- To view the contents of the encrypted vault file: `ansible-vault view group_vars/all.yaml`