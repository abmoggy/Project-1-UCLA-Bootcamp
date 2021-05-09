## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.
![AzureCloudDiagram](https://user-images.githubusercontent.com/29813114/117566098-d7855580-b0de-11eb-830a-b3ae0a3f280b.jpg)
https://github.com/abmoggy/Project-1-UCLA-Bootcamp/blob/main/AzureCloudDiagram.pdf

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the install-elk.yml file may be used to install only certain pieces of it, such as Filebeat.

  install-elk.yml
  filebeat-playbook.yml
  metricbeat-playbook.yml

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly avaiable, in addition to restricting inbound TCP and HTTP traffic to the network.
-  The load balancers protect against Distributed Denial of Service attacks (DDoS) to the webserver.  Traffice is redirected to two other servers, reducing the risk of webserver being "unavaialable" to server web traffic.

- The advantage of a jump box is a single secure connection node to configure multiple docker containers within the network.  

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems and system services and metrics.
- Filebeat watch for changes in the file system.
- Metricbeat record predefined services and metrics.

The configuration details of each machine may be found below.

| Name                  | Function   | IP Address | Operating System |
|-----------------------|------------|------------|------------------|
| jumpbox-provisioner   | Gateway    | 10.0.0.6   | Linux            |
| web-1                 | Web Server | 10.0.0.4   | Linux            |
| web-2                 | Web Server | 10.0.0.5   | Linux            |
| web-3                 | Web Server | 10.0.0.7   | Linux            |
| elk-server            | Monitoring | 10.1.0.4   | Linux            |
### Access Policies

The machines on the internal network are not exposed to the public Internet. Those that have IP Address of 10.*.0.* are internal private IP networks.

Only the jumpbox-provisoner and the load balancer machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 
-47.39.77.13     (home ip)
-184.22.199.105  (allowed user public ip address)

jumpbox-provisioner public IP Address is: 52.234.218.240
load-balancer       public IP Address is: 20.94.200.161

Machines within the network can only be accessed by using Docker containers.
Only the machines listed below are allowed to access the ELK VM.


A summary of the access policies in place can be found in the table below.

| Name                | Publicly Accessible | Allowed IP Addresses |
|---------------------|---------------------|----------------------|
| jumpbox-provisioner | Yes/No              | 10.0.0.6             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because any configuration or changes are done via a predefined script(*.yml file).  This process can be repeated and gurantees that the configured machine will be EXACTLY the same as other using the same configuration file.

The playbook implements the following tasks:
-Install the docker.io on host elk
- Install python3-pip
- Install Docker module
- Increase virtual memory for the container
- Download (pull) and install the elk server container
- Configuring the elk-server to start every time the virtual machine is started
- Mapping the required ports (published) for elk to operate

File location for the ansible playbook file to install elk server:
root@a61338b7ce1f:/etc/ansible/install-elk.yml

Below is the install-elk.yml file:

 1 ---
 2 - name: Configure Elk VM with Docker
 3   hosts: elk
 4   remote_user: sysadmin
 5   become: true
 6   tasks:
 7     # Use apt module
 8     - name: Install docker.io
 9       apt:
10         update_cache: yes
11         force_apt_get: yes
12         name: docker.io
13         state: present
14
15       # Use apt module
16     - name: Install python3-pip
17       apt:
18         force_apt_get: yes
19         name: python3-pip
20         state: present
21
22       # Use pip module (It will default to pip3)
23     - name: Install Docker module
24       pip:
25         name: docker
26         state: present
27
28       # Use command module
29     - name: Increase virtual memory
30       command: "sysctl -w vm.max_map_count=262144"
31
32       # Use sysctl module
33     - name: Use more memory
34       sysctl:
35         name: vm.max_map_count
36         value: '262144'
37         state: present
38         reload: yes
39
40       # Use docker_container module
41     - name: download and launch a docker elk container
42       docker_container:
43         name: elk
44         image: sebp/elk:761
45         state: started
46         restart_policy: always
47         # Please list the ports that ELK runs on
48         published_ports:
49         - 5601:5601
50         - 9200:9200
51         - 5044:5044
52
53       # Use systemd module
54     - name: Enable service docker on boot
55       systemd:
56         name: docker
57         enabled: yes

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

https://github.com/abmoggy/Project-1-UCLA-Bootcamp/blob/main/docker-ps-in-elk-server_SAVE.JPG

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
| web-1   | 10.0.0.4   | DVWA1 
| web-2   | 10.0.0.5   | DVWA2
| web-3   | 10.0.0.7   | DVWA3

We have installed the following Beats on these machines:
web-1
web-2
web-3

The elk server successfully received from this filebeat module
https://github.com/abmoggy/Project-1-UCLA-Bootcamp/blob/main/successful-elk-data-verification.JPG

These Beats allow us to collect the following information from each machine:
- FILEBEAT: Detects and logs any changes made to the file system, (in this instance Apache logs)on that machine and send those logs data to ELK stak server for analysis.

Filebeat playbook file:
root@a61338b7ce1f:/etc/ansible/files/filebeat-playbook.yml

 1 ---
 2 - name: Installing and Launch Filebeat
 3   hosts: webservers
 4   become: yes
 5   tasks:
 6     # Use command module
 7   - name: Download filebeat .deb file
 8     command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
 9
10     # Use command module
11   - name: Install filebeat .deb
12     command: dpkg -i filebeat-7.4.0-amd64.deb
13
14     # Use copy module
15   - name: Drop in filebeat.yml
16     copy:
17       src: /etc/ansible/files/filebeat-config.yml
18       dest: /etc/filebeat/filebeat.yml
19
20     # Use command module
21   - name: Enable and Configure System Module
22     command: filebeat modules enable system
23
24     # Use command module
25   - name: Setup filebeat
26     command: filebeat setup
27
28     # Use command module
29   - name: Start filebeat service
30     command: service filebeat start
31
32     # Use systemd module
33   - name: Enable service filebeat on boot
34     systemd:
35       name: filebeat
36       enabled: yes

- METRICBEAT: Detects and logs changes made the the system or services.  We can then use these metrics to monitor different parameters such as CPU, memory, IO loads, SSH login attemps, SUDO requests.  

Metricbeat playbook file:
 root@a61338b7ce1f:/etc/ansible/files/metricbeat-playbook.yml
 
 1 ---
 2 - name: Install metric beat
 3   hosts: webservers
 4   become: true
 5   tasks:
 6     # Use command module
 7   - name: Download metricbeat
 8     command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
 9
10     # Use command module
11   - name: install metricbeat
12     command: dpkg -i metricbeat-7.4.0-amd64.deb
13
14     # Use copy module
15   - name: drop in metricbeat config
16     copy:
17       src: /etc/ansible/files/metricbeat-configuration.yml
18       dest: /etc/metricbeat/metricbeat.yml
19
20     # Use command module
21   - name: enable and configure docker module for metric beat
22     command: metricbeat modules enable docker
23
24     # Use command module
25   - name: setup metric beat
26     command: metricbeat setup
27
28     # Use command module
29   - name: start metric beat
30     command: service metricbeat start
31
32     # Use systemd module
33   - name: Enable service metricbeat on boot
34     systemd:
35       name: metricbeat
36       enabled: yes

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the install-elk.yml file to include 
- Update the hosts file to include the elk server IP Address (10.1.0.4) into a new category [Elk]
- Run the playbook, and navigate to curl http://localhost:5601/app/kibana (this is if you are ssh to the elk server) if from jumpbox-provisioner (curl http://10.1.0.4:5601/app/kibana) to check that the installation worked as expected.

- Copy the install-elk.yml is the playbook file? to /etc/ansible/install-elk.yml 
- filebeat-config.yml is the configuration file do you update to make Ansible run the playbook on a specific machine? 
One can specify which machine to install the ELK server on versus which to install Filebeat on by specifying what machine group it is in the /etc/ansible/hosts file under [elk] group

- http://10.1.0.4:5601/app/kibana is the url to check that the ELK server is running.

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc.

# SSH to Jumpbox-provisioner from home pc
$ ssh azureuser@52.234.218.240
jumpbox-provisioner public IP Address is: 52.234.218.240
load-balancer       public IP Address is: 20.94.200.161
# check docker container
azureuser@jumpbox-provisioner:~$ sudo docker ps
CONTAINER ID   IMAGE                           COMMAND   CREATED       STATUS         PORTS     NAMES
a61338b7ce1f   cyberxsecurity/ansible:latest   "bash"    13 days ago   Up 3 seconds             epic_ptolemy
# connecting (attach) to docker container
azureuser@jumpbox-provisioner:~$ sudo docker attach epic_ptolemy
root@a61338b7ce1f:~#

# We will want to locate where all our files are
root@a61338b7ce1f:~# cd /etc/ansible
root@a61338b7ce1f:/etc/ansible# ls -al
total 35724
drwxr-xr-x 1 root root     4096 May  9 06:49 .
drwxr-xr-x 1 root root     4096 Apr 25 18:13 ..
-rw-r--r-- 1 root root    19988 Apr 25 18:47 ansible.cfg
-rw-r--r-- 1 root root    73112 May  2 03:44 filebeat-config.yml
drwxr-xr-x 2 root root     4096 May  9 06:50 files
-rw-r--r-- 1 root root     1235 May  9 03:04 hosts
-rw-r--r-- 1 root root     1306 May  1 05:00 install-elk.yml
-rw-r--r-- 1 root root 36447374 May  2 04:15 metricbeat-7.6.1-amd64.deb
-rw-r--r-- 1 root root      685 Apr 30 03:09 pentest.yml
drwxr-xr-x 2 root root     4096 Dec  4  2019 roles

root@a61338b7ce1f:/etc/ansible# ls -al /etc/ansible/files
total 100
drwxr-xr-x 2 root root  4096 May  9 06:50 .
drwxr-xr-x 1 root root  4096 May  9 06:49 ..
-rw-r--r-- 1 root root 73067 May  2 03:03 filebeat-config.yml
-rw-r--r-- 1 root root   920 May  2 03:50 filebeat-playbook.yml
-rw-r--r-- 1 root root  6188 May  2 04:17 metricbeat-configuration.yml
-rw-r--r-- 1 root root   947 May  2 04:20 metricbeat-playbook.yml

# Now that we know the locations of our files within the docker container we can exit and begin to download the files and upload them to Github repository.

# Copy the config, playbook and hosts files from the docker containter into the git repository
$ sudo docker cp epic_ptolemy:/etc/ansible/hosts /home/RedAdmin/Elk-Project/Ansible
$ sudo docker cp epic_ptolemy:/etc/ansible/files/filebeat-config.yml /home/RedAdmin/Elk-Project/Ansible
$ sudo docker cp epic_ptolemy:/etc/ansible/files/filebeat-playbook.yml /home/RedAdmin/Elk-Project/Ansible
$ sudo docker cp epic_ptolemy:/etc/ansible/files/metricbeat-config.yml /home/RedAdmin/Elk-Project/Ansible
$ sudo docker cp epic_ptolemy:/etc/ansible/files/metricbeat-playbook.yml /home/RedAdmin/Elk-Project/Ansible

# The terminal will request your github username and password to push the files.
# Next we need to ensure that the hosts file contains the proper IP Addresses to ensure the playbooks will run properly. You'll want to restart and attach to the docker container.
$ cd /etc/ansible
root@a61338b7ce1f:/etc/ansible# nano hosts

# Edit the hosts file.  Remove the # next to [webservers] and add your ips for your vms under this section. Once this is done you'll need to go create a new group [elk]. Under the [elk] line you'll add the up for your elk machine. 10.1.0.4 Be sure to add the python3-pip reference. 

[webservers]
10.0.0.4 ansible_python_interpreter=/usr/bin/python3
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.7 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3

# Once the ips are added to the proper sections you can run the playbooks as followed:
root@a61338b7ce1f:/etc/ansible# ansible-playbook install-elk.yml 
$ cd /etc/ansible/files
$ ansible-playbook filebeat-playbook.yml
$ ansible-playbook metricbeat-playbook.yml

# If succesfully done run the following command:
$ curl http://10.1.0.0.4:5601/app/kibana

# This should print a HTML onto the console showing that it is up and running. You can then visit the site at http://:20.94.200.1615601/app/kibana to check if the elk stack is running correctly.

