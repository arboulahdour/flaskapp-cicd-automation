# Flaskapp CICD Automation

This Lab demonstrate the implementation of a DevOps CI/CD Pipeline using Git, Jenkins, Ansible and Docker on a virtual envirement created by Vagrant to deploy a python-flask application in a Docker container.
#

Once the developer issues the commit and pushes the code to a specific branch on GitHub, a Jenkins job lanches to build the image, push it on Docker Hub then used in Dev or Test envirement using Ansible. 

## Prerequisites
02 Centos 8 servers
Must have Ansible and Jenkins installed on build server 

## The steps of this tutorial

1. Build Flask Docker image (build server) && Push it to Docker Hub
2. Deploy the application on dev/test server using the created image

### 1. Build Flask Docker image (build server) && Push it to Docker Hub

##### This is the playbook used for building and pushing the image (buildserver.yml)
~~~
---
- name: "Docker Image/Build and Image/Push"
  hosts: build
  become: true
  gather_facts: no
  vars_files:
    - docker.vars

  tasks:

    - name: "Build Step: Cloning Repository"
      git:
        repo: "{{ repo_url }}"
        dest: "{{ repo_dest }}"
      register: repo_status

    - name: "Build Step: Login to remote Repository"
      when: repo_status.changed == true
      docker_login:
        username: "{{ docker_username }}"
        password: "{{ docker_password }}"

    - name: "Build Step: Building image"
      docker_image:
        source: build
        build:
          path: "{{ repo_dest }}"
          pull: yes
        name: "{{ image_name }}"
        tag: "{{ item }}"
        push: true
        force_tag: yes
        force_source: yes
      with_items:
        - 1
~~~

### 2. Deploy the application on dev/test server using the created image

##### This is the playbook used for deploying the application (devserver.yml)
~~~
- name: "Run Application using Docker On Dev Server"
  hosts: dev
  become: true
  gather_facts: no
  vars_files:
    - docker.vars

  tasks:

    - name: "Deploy Step: List of additional package Installation"
      yum:
        name:
        - git
        - python3-pip
        state: present

    - name: "Deploy Step: Python docker extension installation"
      pip:
        name: docker-py


    - name: "Deploy Step: Docker service restart & enable"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "Deploy Step: Run the container"
      docker_container:
        name: flaskapp
        image: "{{ image_name }}:1"
        recreate: yes
        # pull: yes
        published_ports:
          - "8888:5000"
~~~

### Hosts file (hosts)
~~~
[build]
192.168.231.136

[dev]
192.168.231.137
~~~
- The IP address of the Build Server and Dev Server should be updated

### Ansible variable file (docker.vars)
~~~
repo_url: "https://github.com/Arboulahdour/flaskapp-devops.git"
repo_dest: "/var/cicdLab/flaskapp-devops"
image_name: "arboulahdour/flaskapp"
docker_username: "arboulahdour"
docker_password: ""
~~~
- Some of these variables defined in the Ansible variable file (docker.vars) should be updated.

### Author
Created by @Arboulahdour

<a href="mailto:ar.boulahdour@outlook.com">E-mail me !</a>
