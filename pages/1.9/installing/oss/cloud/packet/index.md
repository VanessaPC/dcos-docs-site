---
layout: layout.pug
excerpt:
title: Running DC/OS on Packet
navigationTitle: Packet
menuWeight: 3
---

You can create a DC/OS cluster on Packet bare metal using Terraform.

The included Terraform templates are configured to run Mesosphere DC/OS on Packet. Depending on the DC/OS services that you install, or the amount of computing power your workload needs, you might have to modify the templates to suit your needs. You can modify the Terraform templates, but Mesosphere cannot assist in troubleshooting. If you require support, please email help@packet.net, visit the Packet IRC channel (#packethost on freenode) or consider [DC/OS Enterprise](https://mesosphere.com/).

## Hardware

- 1, 3 or 5 Mesos master nodes in the admin zone

- 4 Mesos private agent nodes

- 1 Mesos public agent node

- Packet “Type 0” Server instances

# Create a DC/OS cluster

## Prerequisites:

- [Packet API Key](https://help.packet.net/quick-start/api-integrations)

- [Packet Project ID](https://help.packet.net/quick-start/api-integrations)

- [Terraform by Hashicorp](https://www.terraform.io/intro/getting-started/install.html)

## Installing DC/OS

#### With this method, the network is open by default. Because of this, network security is a concern and should be addressed as soon as possible by the administrator.

1.  Download and install Terraform using the instructions on the link provided above

2.  [Download the DC/OS Terraform manifests from GitHub](https://github.com/dcos/packet-terraform) into a local  directory.

    ```bash
    git clone https://github.com/dcos/packet-terraform
    ```

3.  From that directory, generate an ssh keypair:

    ```bash
    ssh-keygen -t rsa -f ./packet-key
    ```

4.  Copy `sample.terraform.tfvars` to a new file named `terraform.tfvars`, and edit the new file, filling in the values as desired. The following are blank and if not filled in, you will be prompted by terraform when necessary:

    - packet_api_key - Your Packet API key

    - packet_project_id - Packet Project ID

    - dcos_installer_url - Where to get DC/OS
      https://downloads.dcos.io/dcos/stable/1.9.4/dcos_generate_config.sh

    The following have default values and may be changed depending on your requirements:

    - packet_facility - Packet facility: [ewr1|sjc1|ams1]
      ewr1 is New Jersey, ams1 is Amsterdam, sjc1 is San Jose - default sjc1

    - packet_agent_type - Type of Packet Server to use for the DC/OS Agents: [baremetal_0|baremetal_1|baremetal_3]
      Choose the Packet Server type to use for the DC/OS Private Agents - default baremetal_0

    - packet_master_type - Type of Packet Server to use for the DC/OS Master: [baremetal_0|baremetal_1|baremetal_3]
      Choose the Packet Server type to use for the DC/OS Master Nodes - default baremetal_0

    - packet_boot_type - Type of Packet Server to use for the DC/OS Boot Node: [baremetal_0|baremetal_1|baremetal_3]
      Choose the Packet Server type to use for the DC/OS Boot Server - default baremetal_0

    - dcos_cluster_name - the name of your DC/OS cluster - defaults to packet-dcos

    - dcos_agent_count - Number of private agents to deploy - defaults to  4

    - dcos_public_agent_count - Number of public agents to deploy - defaults to 1

    - dcos_init_pubkey - The path to your ssh public key created in step 4 - defaults to ./packet-key.pub

    - key_file_path - The path to your ssh private key created in step 4 - defaults to ./packet-key

5.  Also from that same directory, run `terraform apply` which will deploy the servers into your project at Packet, and run the DC/OS installation routine. When it completes, you will see output similar to the following, but with the IP addresses assigned to your servers:

    ![terraform apply output](/1.9/img/packet_terraform_output.png)

You may need to wait a few minutes from this point for all the DC/OS services to become active and the control panel available on the master node. After 15 or 20 minutes, you'll want to check out the [troubleshooting](/1.9/installing/oss/troubleshooting/) documentation.

# Launch DC/OS
Launch the DC/OS web interface by entering the Mesos master IP address:

1.  Cut/paste the link provided by after running terraform apply, or by running terraform output from the same directory, into your browser to open the DC/OS web interface. The interface runs on the standard HTTP port 80, so you do not need to specify a port number after the hostname.

2.  Install the DC/OS Command-Line Interface (CLI). You can install the CLI to administer your DC/OS cluster. You can access the documentation at any time by clicking the cluster name in the upper-left side.

    ![install CLI](/1.9/img/install-cli-terminal.png)

## Next steps

- Adding and removing nodes:

  - Run `terraform apply -var ‘dcos_agent_count=N’` to change the private agent count to the number specified. (`‘dcos_public_agent_count’` is also available)

  - Increasing node count is fast, safe, and fun!

  - We recommend against reducing the node count in production. Stateful DC/OS apps and services may suffer outages and failures if nodes are not put into maintenance mode, and their tasks rescheduled through their respective schedulers.
