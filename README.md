## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![draw.io network diagram](https://github.com/Caroline-Coker/Project-13/blob/main/Images/Screen%20Shot%202022-04-30%20at%209.18.29%20AM.png)  

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _yml_ and _config_ file may be used to install only certain pieces of it, such as Filebeat.

**For ansible config edit**: 
_cd /etc/ansible_
_nano ansible.cfg_
change 'remote_user' to 'carolinecoker' 

**assigning a username and ssh public key in azure GUI: **
Web-1/Web-3/Elk server > reset password > reset ssh public key
username: carolinecoker
SSH key: copy id_rsa.pub from ansible control node in .ssh/ directory 
**To get the ssh key run:**
ssh-keygen
cat id_rsa.pub

  - _TODO: Enter the playbook file._

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly _available_ in addition to restricting _traffic_ to the network.
- _What aspect of security do load balancers protect?_ ``Availability, web traffic, and web security 
- What is the advantage of a jump box?_ Automation, Security, Netwrok segmentation, access control

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _data_ and system _logs_.
- _What does Filebeat watch for?_ monitors logfiles or locations that are specified, collects log events, and forwards them to Elasticsearch or Logstash
- _What does Metricbeat record?_ takes metrics and stats that it collects and sends them to the output that is specified, such as Elasticsearch or Logstash

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.1   | Linux            |
| Web-1    | Web server |10.0.0.5  | Linux
| Web-3    | Web server |10.0.0.6  | Linux 
| ELK      | Elk server |10.1.0.4/ 20.213.240.70 |Linux

**This step tests for redudancy on Web-1 and Web-3 VMs: **
go to chrome browser and type in http://[load-balancer-external-IP]/setup.php

see this image if successful: 

turn off one of the VMs and see if you can still access the DVWA file 
then turn off both machines and see if access to the site is denied 

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _ELk Server_ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _workstation Public IP address through TCP 5601_

Machines within the network can only be accessed by _workstation and JumpBoxProvisioner_.

- _Which machine did you allow to access your ELK VM? What was its IP address?_ JumpBoxProvisioner public IP via ssh port 22

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes | Public IP on ssh port 22
| Web-1    | No  | 10.0.0.4 on ssh port 22                     
| Web-3    | No  | 10.0.0.4 on ssh port 22                     
| Elk server| No | Public IP using TCP 5601
| Load balancer | No | Public IP on HTTP 80

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _What is the main advantage of automating configuration with Ansible?_ lets you quickly and easily deploy multitier applications 

The playbook implements the following tasks:
- _In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- Specify different groups of machines as well as a different remote user: e.g. Config elk VM with Docker
- Increase system memory: e.g. Use more memory 
- Install the following services: 'docker.io' 'python3-pip' 'docker' (docker python pip module)
- Launching and exposing the container with published ports: '5601:5601' '9200:9200' '5044:5044'

The following screenshot displays the result of running `sudo docker ps` after successfully configuring the ELK instance.

**how to access Elk VM**
ssh into JumpBoxProvisioner using ssh remote_user@public-IP
start and attach docker container using these commands:
 sudo docker start [container name]
 sudo docker attach [container name]
 ssh into Elk VM with private IP address using this command:
 ssh [remote_user]@10.1.0.4

![docker_ps.png](https://github.com/Caroline-Coker/Project-13/issues/1#issue-1221838359)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _List the IP addresses of the machines you are monitoring_ Web-1: 10.0.0.5 Web-3: 10.0.0.6

We have installed the following Beats on these machines:
- _Specify which Beats you successfully installed_ 
Filebeat and Metricbeat

These Beats allow us to collect the following information from each machine:
- _In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
* Filebeat: monitors the log files or locations that you specify, collects log events, and forwards them
* Metricbeat: helps monitor servers by collecting metrics from the system and services running on the server

**The playbook implements the following tasks: **

---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: carolinecoker
  become: true
  tasks:
  
       Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes

**run the playbook using this command:** 
ansible-playbook install-elk.yml

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

**Download Filebeat playbook using this command:**
_curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/filebeat-config.yml_

SSH into the control node and follow the steps below:
- Copy the _'/etc/ansible/filebeat-config.yml'_ file to _/etc/ansible/filebeat-playbook.yml__.
- Update the filebeat.yml_ file to  'curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb'

- use the command to change the file _nano filebeat-config.yml_ and changed to the following information

hosts: ["10.1.0.4:9200"]
  username: "elastic"
  password: "changeme" 
setup.kibana:
  host: "10.1.0.4:5601"  
  
- Run the playbook using this command:
 _ansible-playbook filebeat-playbook.yml_ and 
- navigate to Kibana > Logs : add log data > system logs > 5:module status > check data to check that the installation worked as expected.

- _file is the playbook:_ 
Filebeat: filebeat-playbook.yml
Metricbeat: metricbeat-playbook.yml

- _command and file used to update to make Ansible run the playbook on a specific machine and specify what file the information goes to:_ 
curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/filebeat-config.yml 

- _URL used to navigate to in order to check that the ELK server is running_ 
![Insallation successful.png] (http://[your.VM.IP]:5601/app/kibana) 

**For Metriccbeat:** 
download metricbeat playbook using this command: 
_curl -L -O https://gist.githubusercontent.com/slape/58541585cc1886d2e26cd8be557ce04c/raw/0ce2c7e744c54513616966affb5e9d96f5e12f73/metricbeat > /etc/metricbeat/metricbeat-config.yml_

copy _/etc/metricbeat/metricbeat-config.yml_ to _/etc/metricbeat/metricbeat-playbook.yml_

_nano metricbeat-config.yml_ and changed to the following:
setup.kibana:
  host: "10.1.0.4:5601"
hosts: ["10.1.0.4:9200"]
  username: "elastic"
  password: "changeme"
  
run the playbook using this command: _ansible-playbook metricbeat-playbook.yml_ and navigate to kibana > add metric data > docker metrics > module status to check data and installation success 

**successful installation should look like:**
[] (https://github.com/Caroline-Coker/Project-13/issues/2#issue-1221851296)

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._