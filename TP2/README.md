Project Documentation for Ansible Deployment
Project Directory Structure

First, we set up the project directory structure:

bash

mkdir -p TP3/ansible/inventories

Inventory File

We create an inventory file to define our hosts and connection details:

File: TP3/ansible/inventories/setup.yml

yaml

all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts: remy.francis.takima.cloud

Summary of Base Commands
Ping Test

Verifies connectivity and correct setup of the inventory:

bash

ansible all -i inventories/setup.yml -m ping

Gather Facts

Retrieves detailed information about the hosts:

bash

ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

Manage Packages

Ensures the desired state of packages (e.g., installing or removing packages):

bash

ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become

Roles Setup
Docker Role

Initialize the Docker role:

bash

ansible-galaxy init roles/docker

In roles/docker/tasks/main.yml, add the tasks to install Docker:

yaml

---
- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repository
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install python3
  yum:
    name: python3
    state: present

- name: Install Docker Python module with pip3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Ensure Docker is running
  service:
    name: docker
    state: started
  tags: docker

Add the Docker role to the playbook:

yaml

- hosts: all
  gather_facts: false
  become: true

  roles:
    - docker

Run the playbook:

bash

ansible-playbook -i inventories/setup.yml playbook.yml

Documenting docker_container Tasks Configuration

By using Ansible roles to manage Docker containers, I was able to modularize the deployment of each component of my application. This approach makes it easy to manage and scale the application, ensuring that each part is correctly configured and can communicate with the others. The use of environment variables and Docker networks ensures seamless integration and connectivity between the containers.
HTTP Role

File: roles/http/tasks/main.yml

yaml

---
# tasks file for roles/http
- name: Run httpd
  community.docker.docker_container:
    name: httpd
    image: remyfrancis510/serveur:1.0
    state: started
    networks:
      - name: app-network
    env:
      BACKEND_host: api
    ports:
      - "80:80"

Database Role

File: roles/database/tasks/main.yml

yaml

---
# tasks file for roles/database
- name: Run database
  community.docker.docker_container:
    name: database
    image: remyfrancis510/my_postgres:v1
    state: started
    networks:
      - name: app-network
    env:
      POSTGRES_PASSWORD: pwd
      POSTGRES_DB: db
      POSTGRES_USER: usr

API Role

File: roles/api/tasks/main.yml

yaml

---
# tasks file for roles/api
- name: Run API
  community.docker.docker_container:
    name: api
    image: remyfrancis510/backend:1.0
    state: started
    networks:
      - name: app-network
    env:
      DB_host: database
      DATABASE_USER: usr
      DATABASE_PASSWORD: pwd
      DATABASE_NAME: db
      DATABASE_PORT: "5432"

Ensuring Network Connectivity

To ensure that all containers can communicate with each other, they are all connected to the app-network Docker network. This network configuration allows containers to refer to each other by their names (e.g., the HTTP container can reach the API container using the name api).
Conclusion

By using Ansible roles to manage Docker containers, I was able to modularize the deployment of each component of my application. This approach makes it easy to manage and scale the application, ensuring that each part is correctly configured and can communicate with the others. The use of environment variables and Docker networks ensures seamless integration and connectivity between the containers.