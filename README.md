# Azure-ELK-Stack
An in-depth walkthrough on how I configured an ELK stack server in order to set up a cloud monitoring system
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Network Diagram](https://github.com/saifpatell/Azure-ELK-Stack/blob/main/Diagrams/Diagram.io.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the filebeat-playbook.yml file may be used to install only certain pieces of it, such as Filebeat.

 - [Ansible ELK Installation and VM configuration](https://github.com/saifpatell/Azure-ELK-Stack/blob/453dc378385a6197ed155b038096677207b2ac5d/Ansible/install-elk.yml)
 - [Ansible Web VM configuration](https://github.com/saifpatell/Azure-ELK-Stack/blob/main/Ansible/Pentest.yml)
 - [Ansible Filebeat Playbook](https://github.com/saifpatell/Azure-ELK-Stack/blob/453dc378385a6197ed155b038096677207b2ac5d/Ansible/filebeat-playbook.yml)
 - [Ansible Metricbeat Playbook](https://github.com/saifpatell/Azure-ELK-Stack/blob/main/Ansible/metricbeat-playbook.yml)

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly responsive, in addition to restricting incoming traffic to the network.

Additionally, load balancers:
 - Distribute client requests or network load efficiently across multiple servers
 - Ensure high availability and reliability by sending requests only to servers that are online
 - Provide the flexibility to add or subtract servers as demand dictates

A jump box is a secure computer that all admins first connect to before launching any administrative task or use as an origination point to connect to other servers or untrusted environments. The purpose of having a jump box is to reduce the attack surface of your network. The majority of your admin work is performed from a highly secured computer that is configured in a way that makes it less likely to be exploited and is restricted in access. The primary benefits of "provisioners" are that they:

a.) Drastically reduce the potential for human error and b.) Make it easy to configure potentially thousands of identical machines all at once.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the files and system metrics.
Within an ELK stack, there are features such as:
- Filebeat: A lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- Metricbeat: Installed on the different servers in your environment and used for monitoring their performance, as well as that of different external services running on them. For example, you can use Metricbeat to monitor and analyze system CPU, memory and load.

The configuration details of each machine can be found below.


| Name                 | Function   | IP Address | Operating System |
|----------------------|------------|------------|------------------|
| Jump-Box-Provisioner | Gateway    | 10.0.0.4   | Linux            |
| Web-1                | Web-Server | 10.0.0.5   | Linux            |
| Web-2                | Web-Server | 10.0.0.6   | Linux            |
| Web-2                | Web-Server | 10.0.0.7   | Linux            |
| ELK-VM               | ELK-Stack  | 10.1.0.4   | Linux            |
| Load Balancer        |Load Balancer | Static External IP       | Linux            |
| Workstation          |Access Control| External IP or PublicIP  | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. Only the ELK-VM can accept connections from the Internet. Access to this machine is only allowed from my public IP address through HTTP via Port 5601. 

Machines within the network can only be accessed by the Jump-Box-Provisioner by SSH via port 22. the ELK server VM can also be accessed by my worstation's public IP address  .

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box    |      No              | My Public IP via SSH  22      |
|   Web1      |      No              | 10.0.0.4 on SSH  22   |
|   Web2      |      No              | 10.0.0.4 on SSH  22  |
|   Web2      |      No              | 10.0.0.4 on SSH  22  |
|ELK Server   |      No              | My Public IP using HTTP via port 5601 |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because Ansible lets you quickly and easily deploy multitier apps. You won't need to write custom code to automate your systems; you list the tasks required to be done by writing a playbook, and Ansible will figure out how to get your systems to the state you want them to be in

The playbook implements the following tasks:

```
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: RedAdmin
  become: true
  tasks:
``` 
- Increase System Memory :
```
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
  ```
- Install the following services:
    ```
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Docker module
      pip:
        name: docker
        state: present
    ``` 
- Launch the container with through published ports:
    ```
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
    ```

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![ELK instance](https://github.com/saifpatell/Azure-ELK-Stack/blob/main/Diagrams/Docker%20PS%20(1).PNG)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 : 10.0.0.5
- Web-2 : 10.0.0.6
- Web-3 : 10.0.0.7


I have installed the following Beats on these machines:
- Web-1
- Web-2
- Web-3
- ELK-VM

These Beats allow us to collect the following information from each machine:
 - Filebeat: log events
 - Metricbeat: metrics and system statistics

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook files to Ansible Docker Container.
- Update the Ansible hosts file /etc/ansible/hosts to include the following:
 ```
[webservers]
10.0.0.4 ansible_python_interpreter=/usr/bin/python3
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3

[elkservers]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3

 ```

Update the Ansible configuration file /etc/ansible/ansible.cfg and set the remote_user parameter to the admin user of the web servers.

### Running the Playbooks

1.Start an ssh session with the Jump Box 
 ```
~$ ssh sysadmin@[Jump Box Public IP]
 ```
2.Start the Ansible Docker container 
 ```
~$ sudo docker start [Ansible Container]
 ```
3.Attach a shell to the Ansible Docker container with the command 
 ```
~$ sudo docker attach [Ansible Container Name]
 ```
4.Run the playbooks with the following commands:
 ```
 ansible-playbook /etc/ansible/Pentest.yml
 ansible-playbook /etc/ansible/install-elk.yml
 ansible-playbook /etc/ansible/roles/filebeat-playbook.yml
 ```
 - Note that the install-elk.yml playbook configures only the server(s) listed as [elkservers] in /etc/ansible/hosts

 - Similarly the Pentest.yml & filebeat-playbook.yml playbook configures the servers listed as [webservers] in /etc/ansible/hosts

 - After running the playbooks and observing no errors in the output, navigate to Kibana to check that the installation worked as expected by viewing Filebeat and Metricbeat data and reports in the Kibana Dashboard

 - Kibana can be accessed at http://[elk-server-ip]:5601/app/kibana
