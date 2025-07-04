# openstack_snort


## 1.0. Introduction
## 1.1. Abstract
This report details the installation and configuration of OpenStack, an open-source cloud computing
platform having a single-node architecture, using DevStack, with a focus on enhancing security through
the integration of Snort, an Intrusion Detection and Prevention System (IDPS) [1]. The project began
by setting up Ubuntu 22.04 LTS on VirtualBox, followed by configuring DevStack to install the latest
OpenStack version. After resolving authentication issues during the addition of an Ubuntu 16.04 Xenial
image, virtual instances named "victim" and "attacker" were created to simulate network scenarios and
test Snort's detection capabilities. Security groups were configured for ICMP and SSH access, and Snort
was installed on the victim instance to monitor and detect attacks, such as TCP SYN flood, using custom
rules. The project setup included detailed hardware and software specifications, and a meticulously
designed networking environment with public, and private networks connected via a router. This report
demonstrates the practical application of OpenStack in building a secure cloud environment,
highlighting the importance of integrating robust security tools like Snort to safeguard against cyber
threats.
## 1.2. Introduction to OpenStack
OpenStack is an open-source cloud computing platform that enables users to deploy and manage
large-scale cloud environments. Key components in OpenStack include:
• Keystone: Provides identity services for user authentication and authorization.
• Glance: Manages and stores images for virtual machines.
• Nova: Handles computing resources and virtual machine management.
• Placement: Tracks resource inventories and manages resource allocation.
• Cinder: Provides block storage services.
• Neutron: Manages networking services and connectivity.
• Horizon: Offers a web-based dashboard to manage cloud services.
These components work together to deliver a robust and scalable cloud environment, facilitating
easier application deployment and management while minimizing the need for physical hardware

## 2.0. Installing OpenStack and deploying a virtual network


To install OpenStack, we utilized DevStack, a collection of scripts designed to set up a complete
OpenStack environment based on the latest versions. The process began with the installation of Ubuntu
22.04 LTS on VirtualBox.
We started by creating a non-root user with sudo privileges (stack user). Following this, we installed
DevStack by cloning the repository using the command:
git clone https://opendev.org/openstack/devstack
Next, we created a local.conf file at the root of the DevStack repository to preset essential passwords
and configurations. The configuration is shown in the figure below:


3 After configuring the ‘local.conf’ file, we initiated the installation by running the ‘./stack.sh’ script.
Upon completion, we accessed the OpenStack dashboard via the ‘HOST_IP’ specified in the
configuration file, using the username ‘admin’ and password defined in the ‘local.conf’.
Although a Cirros image was already installed by default, we required an Ubuntu image for our
instances to support tooles like ‘hping3’ and ‘snort’. We downloaded the Ubuntu 16.04 Xenial image
using the following command:
wget https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
We stored this image in the OpenStack Glance image service using the following command:
openstack --insecure image create --disk-format qcow2 --min-disk 8 --min-ram 512 --file xenial-servercloudimg-amd64-disk1.img --public 16.04
During this process, we encountered an error stating "missing value of URL required for auth-plugin
password." To resolve this, we authenticated through the ‘openrc’ file by downloading it from the
dashboard, saving its contents to a file, and sourcing it to authenticate. After reattempting the image
installation, it succeeded.
With the Ubuntu image in place as shown in the above figure, we proceeded to create two instances
named "victim" and "attacker." After launching the instances, we edited the ‘default’ security group
responsible for traffic control by adding two rules: the first, an ICMP rule to allow pinging and networklevel connectivity, and the second, an SSH rule to enable access to the instances through port 22 via the
SSH key created during the instance setup as shown in the below figure.

4  Once the instances were operational, we installed Snort on the victim instance to detect any attacks. To
monitor for TCP SYN flood attacks, we wrote a custom rule that checks incoming packets, providing a
robust security measure within the cloud environment.
We used single laptop to setup the OpenStack using VirtualBox and their specifications are as follows:
2.1. Hardware Specifications:
• Processor: 11th Gen Intel® Core™ i7-11800H, 2.30 GHz
• RAM: 32 GB
• Storage: 1 TB
2.2. Software Specifications:
• Virtualization Platform: VirtualBox was used to launch the Ubuntu 22.04 LTS operating system.
• Processors Allocated: 6
• Base Memory: 14 GB
• Storage Allocated: 50 GB
• IP address allocated to the instance running Ubuntu 22.04: 10.0.2.15
We used nested virtualization and hence we enabled hardware acceleration, nested paging and
paravirtualization in VirtualBox settings. The specifications of OpenStack instances are as follows


## 2.3. OpenStack Instances Specifications:
• 10.0.2.15 - horizon IP
• 2 instances-victim and attacker
Specifications Victim Attacker
Type m1.small ds1G
RAM 2GB 1 GB
Disk 20GB 10GB

5 VCPU 1 1
Floating IP 172.24.4.39 172.24.4.106
IP Address 10.0.0.39 10.0.0.14
Image Running Ubuntu 16.04 LTS Ubuntu 16.04 LTS

## 2.4. OpenStack Router (router1):
• Acts as the bridge between different networks (Public and Private).
• It's connected to the public network (2001:db8::/64, 172.24.4.0/24) and the private network
(fdae:d2ff:f79b::/64, 10.0.0.0/26).
2.4. Networks:
• Public Network (blue): Connected to router1 and accessible from the external internet.
• Shared Network (orange): A network that may be used by multiple instances or tenants.
• Private Network (green): This network is typically isolated and used for internal
communication between instances.
2.5. Instances :
• Attacker Instance: Connected to the private network (10.0.0.14).
• Victim Instance: Connected to the private network (10.0.0.39).

6 The setup in the above figure shows a typical scenario where both an attacker and victim are in the same
private network, which might be used to simulate attacks and test the detection capabilities of Snort.
The shared and public networks allow for communication outside of the private network, and the router
facilitates the traffic routing between these networks.
## 3.0 Attack Scenario:
In our setup, we simulated an attack using two instances: an attacker and a victim. The attack was
executed as follows:

3.1 Establishing Connection:

OpenSSH Setup: We first ensured that OpenSSH was properly configured on both the attacker and
victim instances to allow for secure remote connections.
Connection: We stored the public key generated by OpenStack using the key/pair feature to Ubuntu
22.04 machine and then we connected to each OpenStack instance using this key to verify
connectivity and configure the environment for the attack and detection.
=> ssh –i <ssh_key> user@<ip address>

In the above command -
• ssh_key is the respective public key generated by OpenStack to connect to the instances
whose private key is connected with the instances by the OpenStack using key/pair feature.
• user is the hostname of the instance to connect with.
• ip_address is the ip of the instance.
7

## 3.2 Launching the Attack:

Attack Tool: We used hping3, a network tool for crafting and analyzing TCP/IP packets, to simulate a TCP SYN flood attack.

Attack Description: We used TCP SYN flood attack which falls under the category of Denial-ofService attack. TCP uses 3-way handshake mechanism to establish a connection between a client and a server which follows a SYN, SYN-ACK, ACK packets in the order. Now to perform the attack, we used hping3 to continuously flood the server (victim in this case) with SYN packets without waiting
to reply for these packets which eventually increased the load on server and denied the server from serving the requests from any other source.
SYN Flood: From the attacker instance, we executed hping3 to generate a high volume of SYN packets targeting the victim instance.

In the above figure:
• -S flag is used to specify hping3 to send only SYN packets to target 10.0.0.39
• -p is used to specify port which is 80
• -c is used to specify the packets count per second
• --flood is used to enter flood mode and continuously flood the target with SYN packets without
waiting for replies.
From the above figure we can see that the attack was launched successfully from the attacker’s instance
to the victim’s instance.

4.0 Detection Scenario:

Step 1: Installation of Prerequisites
We updated our system and installed necessary tools and libraries such as build-essential, libpcap-dev,
libpcre3-dev, libdumbnet-dev, zlib1g-dev, bison, flex, and libssl-dev [6].

Step 2: Installation of DAQ (Data Acquisition library)
We installed the DAQ source code from the official Snort website, extracted the tar file, compiled and
installed DAQ on our victim instance. [6]

Step 3: Installation of Snort
We downloaded the Snort source code, extracted it, and configured the installation with the --enablesourcefire option. Then we compiled and installed Snort.

Step 4: Configure Snort
We then created the necessary directories for Snort, such as /etc/snort, /etc/snort/rules,
/etc/snort/preproc_rules, /var/log/snort, and /usr/local/lib/snort_dynamicrules and set up a user and
group for Snort, and change the ownership of these directories to the Snort user [6].
We edited the Snort configuration file located at /etc/snort/snort.conf to define paths, network variables,
and rules according to our network setup.

Step 5: Test and Run Snort
We validated the Snort configuration using the command to check for any errors.

Step 6: Create and Manage Rules
We created our own custom rules in the /etc/snort/rules/local.rules directory [6].
The above SNORT rule listensfor any tcp SYN packets coming from any source ip addressto the victim
on port 80 and starts alerting the victim when it detects packet count of 100 or more per second with
respective alert message.
The above command enable SNORT to listen for incoming packet traffic on interface ‘ens3’ and outputs
alert messages if any on console.

The above figure displays the alert messages of Snort we received when we performed the attack
described on the victim instance. Snort successfully detected the TCP SYN flood attack.

## 5.0 Challenges and Solution
During the initial setup of the OpenStack environment, we encountered several issues that required a
shift in the approach to successfully deploy and configure the cloud infrastructure.

5.1. Initial Setup Challenges with CentOS:

The initial goal of our project was to install OpenStack on CentOS in a VirtualBox environment.
However, we soon encountered a major compatibility issue: CentOS could not support the network
infrastructure required for OpenStack, mainly due to handling network services
The biggest challenge is that CentOS needs to disable NetworkManager to make OpenStack’s network
components work properly. Unfortunately, disabling NetworkManager on CentOS proved problematic
as in the latest version required network services (such as network.service) were either unavailable or
not working as expected This caused problems with the network connectivity needed to provide
OpenStack has been out of order, resulting in a connection failure.
Despite several attempts to manually configure network configuration and force the system to revert to
default network configuration the configuration remained unstable Lack of a properly configured
network environment prevented us from deploying OpenStack implemented correctly, as the underlying
network services required for communication between the controller, computer and storage and the
nodes are not available
Due to these ongoing issues, we decided to move to Ubuntu 22.04 LTS, which offers better support for
both OpenStack and DevStack. Ubuntu's network management tools are closely aligned with
OpenStack's requirements, allowing deployment to continue without the network configuration issues
we encountered with CentOS

# 5.2. Challenges with Glance Image and Snort Installation:

Having successfully switched to Ubuntu, we ran into another set-up problem. Initially, we used CirrOS
images in Glance for our examples. Although CirrOS is lightweight and useful for rapid testing, both
the attacker and victim instances lacked the tools and libraries needed to support the Snort installation
This limitation in CirrOS became apparent when we tried to configure Snort for intrusion detection.
The image's minimal environment did not support Snort or other required packages, causing multiple
failures in the system. To overcome this, we switched to the Ubuntu 16.04 Xenial image, which provided
the necessary compatibility and enabled Snort to install and configure correctly.


# 5.3. Multi-Node Deployment Challenges:

In our efforts to create a more scalable OpenStack environment, we also tested a three-node deployment
on VirtualBox, with separate VMs for the controller, and two compute nodes. The installation process
and errors faced are described further in the APPENDIX section of this report.
10
After exhausting troubleshooting options, we decided to simplify our setup by reverting to a singlenode deployment using DevStack, which allowed us to focus on implementing and testing security
features without the complexities of a multi-node setup.


# 5.4. DNS Resolution Problems: 

After we were able to successfully take the remote access of the
attacker and victim server instances using SSH we were faces with yet another challenge.
In order to download the required tools, we needed to update the servers. The ‘sudo apt update’
command was failing but ping request to an IP was getting successful. So, the issue was with the DNS
resolution as shown in below figure.
We changed the nameserver specified in the /etc/resolv.conf file to 1.1.1.1 and 1.0.0.1 which are
mapped to the public DNS servers of Cloudflare and it solved our issue as shown in the figure below:


##  APPENDIX
7.1. Installation of Bobcat, OpenStack 2023.2
In this installation, we attempted a minimal deployment for Bobcat, OpenStack 2023.2. This release
was selected because although it is not under development any more, it is still maintained. This means
bugs have been fixed and most solutions are available online.
We attempted this deployment on Oracle VirtualBox with one controller (controller) and two compute
nodes (compute1, compute2). The VM specifications were as follows:
• Storage: 25GB for each VM.
• RAM: 4GB each, with 2 VCPUs.
• OS: Ubuntu 22.04 LTS
We launched controller and compute1, configured the network, and confirmed that the two nodes
communicated (ping) with each other and the Internet. Compute2 was spun up later by cloning
compute1.
On the controller, we successfully configured and verified operation of chrony (clock synchronization),
MariaDB, RabbitMQ, Memcached, and Etcd. In addition, this deployment required identity service
(Keystone), image service (Glance), placement service (Placement), compute service (Nova),
networking service (Neutron), and the dashboard (Horizon).
1. Keystone identity service
This was done on controller. We installed keystone package and added configurations to keystone.conf
file. However, when trying to populate the service database, an error was thrown:
13
We found a fix for this under the reported keystone bugs [5] that involved making a try-except
construction for the get_ident function in thread.py as shown below:
Following this, we successfully populated the database and configured Apache HTTP server.
2. Glance Image service
This was done on controller. We successfully installed glance package, edited glance-api.conf file and
populated the database for this service. An image of Ubuntu 16.04 was successfully uploaded.
3. Placement service
This was done on controller node only. The placement-api package was installed, the placement.conf
file edited, and we verified the successful operation of the service.
4. Nova compute service
This was done on both the controller and compute1. After installing the packages and confirming that
all nova services were running on both controller and compute1, we verified the operation by
successfully registering the compute node on the controller using the “nova-manage discover_hosts”
command.
At this point, we cloned the compute1 VM to create compute2. All hostnames and IP were edited as
required in the three nodes. However, on starting the nova-compute service an error was thrown:
We determined that this error was as a result of nova-compute service picking the ID of compute1.
Since we had already started the service on compute1, the ID was stored in /var/lib/nova/compute_id
and was copied during the cloning. Examination of the error revealed that the VM’s hostname was tied
to the compute_id. As such, hypervisor in compute2 was picking the hostname of compute1 instead of
compute2. We fixed this error by removing the compute_id in compute2 using “rm
/var/lib/nova/compute_id” and restarting the nova-compute service, which then ran successfully.
5. Neutron networking service
We selected Networking Option 2: Self-service networks. This option allows a regular, non-privileged
account to create and manage virtual networks without involving an administrator. Neutron was
configured on both the controller and compute nodes. We also created a bridge for neutron to manage
network traffic through open vswitch bridges. All neutron services were restarted successfully.
6. Horizon dashboard
14
This was done on controller node. We successfully installed openstack-dashboard package, edited the
local_settings.py file as required and reloaded the apache2.service web server configurations. To verify,
we visited http://10.0.0.11/horizon and successfully logged in as admin and demo users.
7. Launching an instance
We successfully created a provider network and subnet. We encountered an error when trying to use the
demo account to create a private network. The demo user (non-privileged account) had no permissions
to create a network. We therefore created a policy.yaml file in neutron on the controller to allow nonprivileged accounts to create their own networks, subnets and routers.
We were then able to successfully create a subnet with CIDR 192.168.100.0/24 and a router that would
provide NAT services.
As seen in the above, we noticed that the private network was DOWN. This impacted our ability to
deploy instances through our horizon dashboard and through CLI, even though we were able to create
m1.nano and m1.tiny flavors through Horizon dashboard. Error logs showed the following error:
We tried to change the vnic_type to different options including ‘direct’ and ‘direct physical’ but the
error persisted. We verified our configurations in neutron.conf, nova.conf, and openvswitch_agent.ini.
We also checked that the bridge and the networks were configured correctly and working.
At this point, we could not find any solutions to making the network status ACTIVE. We therefore
decided to stick with the single-node architecture for our OpenStack deployment.
