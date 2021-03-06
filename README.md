## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Unit 13 Diagram](Diagrams/Unit-13-Diagram.png "Unit 13 Diagram")

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the yaml file may be used to install only certain pieces of it, such as Filebeat.

  - [My playbook](/Ansible/my-playbook.yml)
    - Sets up a set of highly available web servers and prepares DVWA.
  - [Filebeat playbook](/Ansible/filebeat-playbook.yml)
    - Sets up Filebeat on the web servers to push logs to the ELK server.
  - [Metricbeat playbook](/Ansible/metricbeat-playbook.yml)
    - Sets up Metricbeat on the web servers to push metrics to the ELK server.
  - [ELK playbook](/Ansible/elk-playbook.yml)
    - Sets up ELK on a server that has a public IP.


### This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- The load balancer ensures that each host is not overloaded with requests.
- The load balancer allows you to scale the number of servers hosting the site on the backend without affecting the site's availability. 
- The load balancer can also handle SSL requests. Our DVWA website is only doing HTTP in this scenario.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the files and system monitoring.
- Filebeat handles syslog monitoring.
  - Any changes to the system that are recorded by syslogs will show up in Filebeat.
- Metricbeat handles the monitoring of system resources.
  - System usage like CPU, memory, disk, and network statics show up in Metricbeat.

The configuration details of each machine may be found below.

| Name                | Function | IP Address | Operating System |
|---------------------|----------|------------|------------------|
| ELK-01              | ELK stack | 10.1.0.4  | Ubuntu LTS 20.04 |
| Jumpbox-Provisioner | Gateway  | 10.0.0.4   | Ubuntu LTS 20.04 |
| Web-1               | Server   | 10.0.0.5   | Ubuntu LTS 20.04 |
| Web-2               | Server   | 10.0.0.6   | Ubuntu LTS 20.04 |
| Web-3               | Server   | 10.0.0.7   | Ubuntu LTS 20.04 |


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox-provisioner and ELK machines can accept connections from the Internet. Access to these machines are only allowed from the following IP addresses:
- My public IP is the only IP whitelisted to allow access via port 22 (SSH) and the ELK server is allowing ports 5044, 5601, and 9200 so that I can view Kibana's web page.
- The internal web servers are behind a load balancer that are only accepting port 80 (HTTP) access from my public IP.

Machines within the network can only be accessed by Jumpbox-provisioner via SSH.
- Which machine did you allow to access your ELK VM?
  - My home PC has it's public IP whitelisted so that I can view the Kibana web page.
- What was its IP address?
  - 20.114.213.34

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| ELK-01 | Yes, via 20.114.213.34 | vNet, 10.0.0.4, My Public IP |
| Jumpbox-Provisioner | Yes,  via 20.106.161.137 | vNet, My Public IP |
| Web-1 | No | vNet, 10.0.0.4 |
| Web-2 | No | vNet, 10.0.0.4 |
| Web-3 | No | vNet, 10.0.0.4 |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- It is scalable. No matter how many servers need to be deployed, you can configure each to be identical.
- Once a playbook is configured, you remove the human element from making a mistake while trying to configure large numbers of servers at one.
  - One distraction and you forget to modify a variable in the configuration file. Now that server isn't doing what was intended or pointing at the correct IP.
- It is a massive time saver. Traditional hands on approach would take quite a while to configure even a few servers.

The playbook implements the following tasks:
- Updates the server's repo cache so it knows about docker.io and pip.
- It sets system variables to defined how much virtual memory can be allocated to ELK.
- Installs docker.io using apt.
- Installs python3-pip using apt.
- Installs docker using pip.
- Enables docker to auto start on boot.
- Reaches out to docker and gets ELK version 761
  - Starts the service.
  - Maps port bindings to 5044, 5601, and 9200.

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![docker ps](/Images/docker_ps.jpg)


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Webservers
  - Web-1 - 10.0.0.5
  - Web-2 - 10.0.0.6
  - Web-3 - 10.0.0.7

We have installed the following Beats on these machines:
- Filebeats
- Metricbeats

These Beats allow us to collect the following information from each machine:
- Filebeats is aggregating all the syslogs from the web servers so ELK can analyze and display graphical representations of what has been monitored.
  - ![Filebeat Example](/Images/filebeats-example.png)
- Metricbeats is aggregating all the metrics from the web servers so ELK can visualize system resources like CPU, memory, network, and disk usage.
  - ![Metricbeat Example](/Images/metricbeats-example.png)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the Ansible files from my GitHub directory to "/etc/ansible/" location on your Ansible control node.
- Update the ansible.cfg file to include your remote user name:
  ```
  remote_user=YOURUSERNAME
  ```
- Update the hosts file to ensure all of your servers are grouped according to their group and add the python interpreter info:
  ```
  [webservers]
  10.0.0.5 ansible_python_interpreter=/usr/bin/python3
  10.0.0.6 ansible_python_interpreter=/usr/bin/python3
  10.0.0.7 ansible_python_interpreter=/usr/bin/python3
  ```
  ```
  [elkserver]
  10.1.0.4 ansible_python_interpreter=/usr/bin/python3
  ```
- Ensure the [Filebeat Config](/Ansible/filebeat-config.yml) and [Metricbeat Config](/Ansible/metricbeat-config.yml) files are updated to reflect the IP of your Kibana server.
  - Filebeat config line numbers:
    - 1105 ```hosts: ["10.1.0.4:9200"]```
    - 1106 ```username: "elastic"```
    - 1107 ```password: "changeme"```
    - 1805 ```host: "10.1.0.4:5601"```
  - Metricbeat config line numbers:
    - 62 ```host: "10.1.0.4:5601"```
    - 95 ```hosts: ["10.1.0.4:9200"]```
    - 96 ```username: "elastic"```
    - 97 ```password: "changeme"```

- Run the playbook, and navigate to ```http://ELK-Public-IP/app/kibana``` to check that the installation worked as expected.
  - ```ansible-playbook /etc/ansible/elk-playbook.yml```
  - ![ELK successful](/Images/kibana.jpg "ELK successful")

- Once ELK is successfully installed and running, you can now run the [Filebeat playbook](/Ansible/filebeat-playbook.yml).
  - ```ansible-playbook /etc/ansible/filebeat-playbook.yml```
  - You can click Add Log Data > System Logs > DEB for how to install Filebeat but my playbook does these steps for you.
  - When the playbook finishes, you can click check data on Kibana to verify ELK is receiving data from the web servers.
  - ![Filebeat successful](/Images/filebeat_success.jpg "Filebeat successful")

- You can now run the [Metricbeat playbook](/Ansible/metricbeat-playbook.yml).
  - ``` ansible-playbook /etc/ansible/metricbeat-playbook.yml```
  - Back on the Kibana homepage, click Add Metric Data > Docker Metrics > DEB. Like Filebeat, scroll down to check data.
  - ![Metricbeat successful](/Images/metricbeat_success.jpg "Metricbeat successful")

- You now have an ELK server with Filebeat and Metricbeat data collection that you can filter and analyze.
 
### Some useful commands to help use Docker:

| Command | What the command does |
|---------|-----------------------|
| sudo docker container list -all | lists all containers installed |
| sdo docker start CONTAINERNAME | initialize the container by name |
| sudo docker attach CONTAINERNAME | gives you CLI access into the container | 
| sudo docker cp CONTAINER/SOURCE/PATH LOCAL/DESTINATION/PATH | copies files from your container to host machine |
