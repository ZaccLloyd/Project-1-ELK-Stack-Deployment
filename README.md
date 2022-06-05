## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/ZaccLloyd/Project-1-ELK-Stack-Deployment/blob/main/Images/Project-1-Network-Diagram.PNG)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Playbook file may be used to install only certain pieces of it, such as Filebeat.

#### Playbook 1: pentest.yml
```
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
```

#### Playbook 2: install-elk.yml
```
---
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
  become: true
  tasks:
    # Use apt module
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
        value: 262144
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
```

#### Playbook 3: filebeat-playbook.yml
```
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:
  
  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb 
 
  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb 
  
  - name: drop in filebeat.yml 
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
  
  - name: enable and configure system module
    command: filebeat modules enable system
  
  - name: setup filebeat
    command: filebeat setup
  
  - name: Start filebeat service
    command: service filebeat start
```
#### Playbook 4: Metricbeat-playbook.yml
```
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start
```


This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build



### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly Available, in addition to restricting access to the network.
- Load balancers protect server availability, balancing requests across multiple servers while simultaneously working as a health probe continually tests availability. 
- The jump box allows remote connections into the cloud network, while also minimise the attack surface through easy monitoring of all remote connections. The use of an SSH key means that only trusted IPs can gain access to the Jumpbox.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the systems configuration and system files.
- Filebeat is used to monitor log files
- Metricbeat is used to collect OS & service statistics from all Vms that are monitored on the network.

The configuration details of each machine may be found below.

| Name     | Function      | IP address | Loaction    |
|----------|---------------|------------|-------------|
| Jump-Box | Admin Gateway | 10.10.0.4  | Aus Central |
| Web-1    | DVWA          | 10.10.0.5  | Aus Central |
| Web-2    | DVWA          | 10.10.0.6  | Aus Central |
| Elk-VM   | ELK Analysis  | 10.0.0.4   | Aus East    |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept ssh connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Admin Ip Address specified via ssh key generation.

Machines within the network can only be accessed by Jump-box. The Jump Box IP address is 10.10.0.4.
- The jump box can access ELK-VM using an SSH via the Ansible Docker container 'Viginant_Fermi'

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP addresses            | Allowed ports |
|----------|---------------------|---------------------------------|---------------|
| Jump Box | No                  | Admin IP (via predefined key)   | SSH:22        |
| Web-1    | Yes:HTTP            | Public                          | HTTP:80       |
| Web-2    | Yes:HTTP            | Public                          | HTTP:80       |
| ELK-VM   | No                  | Jumpbox IP (via predefined key) | HTTP:5601     |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- It ensures that machines are built and deployed in a consistent and timely manner. Consistent and timely system configuration and deployment ensures that the system exposure to human error is minimized, and the attack surface is reduced.
- Automatic configuration emphasises the ability for the system capacity for horizontal scaling dependent on system requirements.    

### Playbooks
The four playbooks above implement the following tasks:

#### Playbook 1: pentest.yml
pentest.yml is used to set up DMWA servers running in a Docker container on each of the web services show in the diagram above.  It implements the following tasks:

* Installs Docker 
* Installs Python 3 
* Installs Docker’s python module 
* Downloads and launch the DVWA Docker container 
* Enables the Docker service

#### Playbook 2: ELK-Install.yml
This playbook is used to set up the ELK server within the docker container on the ELK-VM. ELK-Install.yml implements the following:

* Installs Docker 
* Installs Python 3 
* Installs Docker’s python module 
* Increase memory to support the ELK stack
* Download and launch the Docker ELK container

#### Playbook 3: Filebeat-playbook.yml
This playbook is used to deploy Filebeat on each of the webservers to allow central monitoring of the ELK services. Filebeat specifically collects log files on a given server. Filebeat-playbook.yml implements the following:

* Downloads and installs Filebeat 
* Enables and configures the docker module to support Filebeat. 
* Configures and starts filebeat

#### Playbook 4: Metricbeat-playbook-yml
The following playbook is used to deploy Metricbeat on the webservers to allow central monitoring of the ELK services. Metricbreat specifically collects OS and Service metrics on a given server. Metricbeat-playbook-yml implements the following:

* Downloads and installs Metricbeat
* Enables and configures the docker module to support Metricbeat
* Configures and starts Metricbeat


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![](https://github.com/ZaccLloyd/Project-1-ELK-Stack-Deployment/blob/main/Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1: 10.10.0.5
- Web-2: 10.10.0.6 

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat collectsand ships log files and events then forward them to Elasticsearch for indexing. e.g. Filebeat collects system.log files to monitor what commands are executed on a given machine. 
- Metricbeat collects and ships server metrics regarding the operating system and other running services and ships them to Elaticsearch for indexing. i.e. metricbeat collects metrics on MySQL regarding service details.   

### Using the Playbook
To use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook files to the Ansible Docker Container Located on the Jump Box.
- Update the Ansible hosts file `/etc/ansible/hosts` file to include:

```
[webservers]
10.10.0.5	 ansible_python_interpreter=/usr/bin/python3
10.10.0.6	 ansible_python_interpreter=/usr/bin/python3

[elkservers]
10.0.0.4	 ansible_python_interpreter=/usr/bin/python3
```

- Run the playbook and navigate to the Filebeat installation page on the ELK server GUI via the http://<elk-server-ip>:5601/app/kibana to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:
- All Ansible files are located within the /etc/ansible directory within the Ansible Docker Container. 
Imputing the Ip address of the target machine within the /etc/ansible/hosts file under 'webservers' or 'elkservers' will ensure that ansible will run the playbook on the given machine.
-  navigate to http://<elk-server-ip>:5601/app/kibana to see if the ELK server is running. 


1. ssh into the Jump Box VM via your local machine `~$ ssh sysadmin@<Jump Box Public IP>`
2. Start the Ansible Docker container `~$ sudo docker start <Ansible Container>`
3. Attach to the container `~$ sudo docker attach <Ansible Container Name>`
4. Use `nano [name-of-play].yml` to open a YML file and begin configuring. 
5. Once the playbooks have been configured to your requirement run the playbooks with the following commands:
	* `ansible-playbook /etc/ansible/pentest.yml`
	* `ansible-playbook /etc/ansible/install-elk.yml`
	* `ansible-playbook /etc/ansible/roles/filebeat-playbook.yml`
	* `ansible-playbook /etc/ansible/roles/metricbeat-playbook.yml`

Note that  'filebeat-playbook.yml & metricbeat-playbook.yml' will only configue the servers whose IPs' are listed under [webservers] in the '/etc/ansible/hosts' file.
Also note that the `install_elk.yml` play will only configure the servers whose IPs' are listed under [elkservers] in the '/etc/ansible/hosts' file.

After configuring the play file of a given playbook, run: 'ansible-playbook [name-of-play].yml' to ensure a successful output. You may need to troubleshoot for errors. 
	