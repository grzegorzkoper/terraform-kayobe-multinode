==========================
Terraform Kayobe Multinode
==========================

This Terraform configuration deploys a requested amount of Instances on an OpenStack cloud, to be
used as a Multinode Kayobe test environment.

Usage
=====

These instructions show how to use this Terraform configuration manually. They
assume you are running an Ubuntu host that will be used to run Terraform. The
machine should have network access to the environment that will be created by this
configuration.

Install Terraform:

.. code-block:: console

   wget -qO - terraform.gpg https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/terraform-archive-keyring.gpg
   sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/terraform-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/terraform.list
   sudo apt update
   sudo apt install terraform

Clone and initialise the Kayobe config:

.. code-block:: console

   git clone https://github.com/stackhpc/terraform-kayobe-multinode
   cd terraform-kayobe-multinode


Initialise Terraform:

.. code-block:: console

   terraform init

Generate an SSH keypair:

.. code-block:: console

   ssh-keygen -f id_rsa -N ''

Create an OpenStack clouds.yaml file with your credentials to access an
OpenStack cloud. Alternatively, download one from Horizon.

.. code-block:: console

   cat << EOF > clouds.yaml
   ---
   clouds:
     sms-lab:
       auth:
         auth_url: https://api.sms-lab.cloud:5000
         username: <username>
         project_name: <project>
         domain_name: default
       interface: public
   EOF

Export environment variables to use the correct cloud and provide a password:

.. code-block:: console

   export OS_CLOUD=sms-lab
   read -p OS_PASSWORD -s OS_PASSWORD
   export OS_PASSWORD

Generate Terraform variables:

.. code-block:: console

   cat << EOF > terraform.tfvars
   ssh_private_key = "id_rsa"
   ssh_public_key = "id_rsa.pub"
   seed_vm_name = "kayobe-seed"
   seed_vm_image = "CentOS-stream8"
   multinode_keypair = "multinode_keypair"
   seed_vm_flavor = "general.v1.tiny"
   multinode_vm_network = "stackhpc-ipv4-geneve"
   multinode_vm_subnet = "stackhpc-ipv4-geneve-subnet"
   multinode_image = "CentOS-stream8"
   multinode_flavor = "general.v1.tiny"
   EOF

Generate a plan:

.. code-block:: console

   terraform plan

Apply the changes:

.. code-block:: console

   terraform apply -auto-approve

You should have requested number of resources spawned on Openstack, and ansible_inventory file produced as output for Kayobe.
