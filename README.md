# Ansible_Dynamic_Inventory_Task

Task Description..!
🔰 Provision EC2 instances through ansible.

🔰 Retrieve the IP Address of instances using the dynamic inventory concept.

🔰 Configure the web servers through the ansible role.

🔰 Configure the load balancer through the ansible role.

🔰 The target nodes of the load balancer should auto-update as per the status of web servers.

1.Ec2 Instances provision..

Firstly, inorder to make ansible to contact AWS we need to install boto libraries of python.

To install : *pip3 install boto3*

Playbook for provisioning 3 ec2 instances(later we use 2 for webservers and one for loadbalancer) :

ec2p.yml -> Have to run on localhost / Master Node itself

- hosts: localhost
  vars_files:
          - awskeys.yml
          - var.yml
  tasks:
          - name: Provisioning 3 instances on AWS..!
            ec2:
                   key_name: "{{key_pair}}"
                   instance_type: "{{instance_type}}"
                   image: "{{ami_image}}"
                   wait: yes
                   vpc_subnet_id: "{{subnet}}"
                   assign_public_ip: yes
                   region: "{{region}}"
                   state: present
                   aws_access_key: "{{aws_access_key}}"
                   aws_secret_key: "{{aws_secret_key}}"
                   instance_tags:
                           Name: "{{item}}"
            loop: "{{instances}}"
            register: total
            
Here Used to var files to keep my aws credentials and we can secure that file using the ansible vault: ansible-vault encrypt awskeys.yml    
RUN the Above Playbook: ansible-playbook --ask-vault-pass ec2p.yml


2.Retrieving the Ip adresses dynamically:

In order to work on dynamic inventory , ansible has precreated scripts for almost all platforms.. here we apply the same.

👉 Download Inventory scripts to inventory directory using wget command:

$wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.py

$wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.ini

👉 Make the scripts executable..!

$ chmod +x ec2.py 
$ chmod +x ec2.ini

👉 Export your credentials by using the export command:

$ export AWS_REGION='ap-south-1'
$ export AWS_ACCESS_KEY_ID='ADB4XXXXXXXXXXXXXXX'
$ export AWS_SECRET_ACCESS_KEY='EF2@RTXXXXXXXXXXXXXXXX'

Now, we are set to retrieve the ip's of our instances from aws..

Run : ansible all --list-hosts


**Ansible role:** Roles let you automatically load related vars_files, tasks, handlers, and other Ansible artifacts based on a known file structure. Once you group your content in roles, you can easily reuse them and share them with other users. An Ansible role has a defined directory structure with seven main standard directories.

Now, let's create roles for webserver and loadbalancer...use the commands :

$ ansible-galaxy init loadbalancer
$ ansible-galaxy init webserver

Note: Run the above two commands from the path /etc/ansible/roles/ ...which is the path from where ansible search for roles by default.

Now After initializing the roles..open the Webserver Role and  add the webserver configuration play in it.
Tasks -> main.yml
- name: Installation web server
  package: 
    name : "httpd"
    state : "present"
- name: Copying Web content to server
  copy: "Hello ..This is Webserver {{ansible_facts ['default_ipv4']['address']}}"
  dest: /var/www/html/index.html
- name: Start web service
  service:
  name: "httpd"
  state: started
  
 3.Configure Haproxy-loadbalancer using ansible role.
 
Tasks -> main.yml:
 - name: Install haproxy s/w
   package: "haproxy"
   state: present
 - name: Copying haproxy config file
   template:
       src: "haproxy.cfg"  #this file we have to get and do necessary chganges to it and upload to managed node.
       dest: "/etc/haproxy/haproxy.cfg"
    notify: restart loadbalancer
- name: start load balancer
  service:
    name: "haproxy"
    state: started
   
Handlers -> main.yml:
- name: restart loadbalancer
  service:
    name: "haproxy"
    state: restarted
#In templates directory of the role have to place that haproxy config file by making some changes so that it can register newly launched webservers to the load balancer.

Final Playbook to configure both webservers and loadbalancer..using roles:

infra.yml :
- hostsL: loadbalancer
  roles:
        - loadbalancer
- hosts: webserver
  roles:
        - webserver

**BY RUNNING THIS FINAL PLAYBOOK WE WILL GET OUR DESIRED INFRASTRUCTURE WITH WEBSERVERS AND HAPROXY LOADBALANCER...Thanks For Reading**




