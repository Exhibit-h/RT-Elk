# RT-Elk
Monitor applications using a machine with ELK installed

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[F:\Studies\Project_1\Diagram_RT_Elk]

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the .yml file may be used to install only certain pieces of it, such as Filebeat.


This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. The load balancer ensures that work to process incoming traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorized users — namely, ourselves — will be able to connect in the first place.


Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to their systems file resources, and metrics like login attempts, commands run, etc.


The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name                | Function | IP Address | Operating System |
|---------------------|----------|------------|------------------|
| JumpBox-Provisioner | Gateway  | 10.0.0.4   | Linux            |
| Web-1               | DVWA     | 10.0.0.5   | Linux            |
| Web-2               | DVWA     | 10.0.0.6   | Linux            |
| RT-Elk              | MONITOR  | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:
Availability Zone 1: DVWA 1 + DVWA 2
Availability Zone 2: ELK

Machines within the network can only be accessed by each other. The Web-1 and Web-2 VMs send traffic to the Elk server. 

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | No                  | 10.0.0.4             |
| Web-1    | Yes                 | Any                  |
| Web-2    | Yes                 | Any                  |
| RT-Elk   | No                  | 10.1.0.4             |

### Elk Configuration

The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage the ELK container. Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because using the same ansible playbook, changes later on can be easily applied to all necessary devices within the network. Also, by configuring these machines internally we are able to make changes to them without exposing their IPs to the public internet. Access to the jump box is only allowed from the IP address 71.238.79.179.

The playbook implements the following tasks:

- Configures the VM for Elk stack
- Installs all necessary components needed to automate the system monitoring processes (docker.io, python3-pip and docker)
- Increases memory to run smoothly
- After docker is installed, it downloads the Elk container and enables it

With a duplicate of the playbook below:
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

F:\Studies\docker_ps

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1:10.0.0.5
- Web-2:10.0.0.6


We have installed the following Beats on these machines:
- Filebeat
- Metricbeat


These Beats allow us to collect the following information from each machine:
- Filebeat scans for changes to files within the system. Specifically, it is scanning Apache logs which document requests, the requested resource, repsonse code, time to respond, and source IP address. 
- Metricbeat searches for changes in system metrics like CPU or memory usage. It can be used to detect alterations in CPU/RAM usage, SSH login attempts, and failed sudo escalations.

The playbook below installs Metricbeat on the target hosts. The playbook for installing Filebeat is not included, but looks essentially identical — simply replace metricbeat with filebeat, and it will work as expected.

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
      src: /etc/ansible/metricbeat-config.yml
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

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the .yml playbook file to the Ansible machine.
- Update the ~/etc/ansible/hosts file to designate the private IP(s) of the machine(s) you are monitoring to the 'webservers' host group.
- Run the playbook, and navigate to http://[RT-Elk.VM.IP]:5601/app/kibana to check that the installation worked as expected.
