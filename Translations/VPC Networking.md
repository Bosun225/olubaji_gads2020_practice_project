#LAB TITLE: VPC Networking

## OBJECTIVES:
In this lab, you learn how to perform the following tasks:
-Create an auto mode network with firewall rules
-Convert an auto mode network to a custom mode network
-Create custom mode VPC networks with firewall rules
-Create VM instances using Compute Engine
-Explore the connectivity for VM instances across VPC networks

## Task 1. Create an auto mode network with two instances in us-central1 and europe-west1
First let's create an automode network for our intances

	-To create an automode network run the cmd- gcloud compute networks create mynetwork --subnet-mode=auto

Next, We'll create four firewall-rules(allow-ICMP, allow-internal, allow-rdp, allow-ssh) for mynetwork
	-To create the required firewall-rules for my network run the below cmds-
		For allow-ICMP >>>>> gcloud compute firewall-rules create mynetwork-allow-icmp --action=ALLOW --direction=INGRESS 						 --network=mynetwork --rules=icmp --source-ranges=0.0.0.0/0

		For allow-internal >>>>> gcloud compute firewall-rules create mynetwork-allow-internal --action=ALLOW        									 --direction=INGRESS  --network=mynetwork --rules=all --source-ranges=10.128.0.0/9

		For allow-rdp >>>>> gcloud compute firewall-rules create mynetwork-allow-rdp --action=ALLOW        									 --direction=INGRESS  --network=mynetwork --rules=tcp:3380 --source-ranges=0.0.0.0/0

		For allow-ssh >>>>> gcloud compute firewall-rules create mynetwork-allow-ssh --action=ALLOW        									 	--direction=INGRESS  --network=mynetwork --rules=tcp:3380 --source-ranges=0.0.0.0/0

		###Note: Three of these firewall-rules can be combined and created at once i.e. instead of four cmds we'll only run 2 i.e. we combine the cmd to create ICMP, RDP & SSH in one cmd and then create the allow-internal firewall-rule with a different cmd. We will see this later when we are creating firewall-rules for our custom mode networks 

-To create an instance in us-central1 region run the cmd- gcloud compute instances create mynet-us-vm 																			--machine-type=n1-standard-1 --subnet=mynetwork 																			--zone=us-central1-c
													--hostname=mynet-us-vm.us-central1-c.c.myvpcnetworking-project.internal
	(Note: You need to use the --hostname flag when creating your vm, this allow your vm to resolve the ip using the custom)
	Note: The hostname format should be the zonal internal DNS type i.e. [INSTANCE_NAME].[ZONE].c.[PROJECT_ID].internal 

-To create an instance in europe-west1 region run the cmd- gcloud compute instances create mynet-eu-vm 																			--machine-type=n1-standard-1 --zone=europe-west1-c 																			--subnet=mynetwork
													--hostname=mynet-eu-vm.europe-west1.c.myvpcnetworking-project.internal
	(Note: You need to use the --hostname flag when creating your vm, this allow your vm to resolve the ip using the custom DNS. This enables you to ping one vm from the other using the name of your vm only)
	Note: The hostname format should be the zonal internal DNS type i.e. [INSTANCE_NAME].[ZONE].c.[PROJECT_ID].internal
	

Now, we'll verify connectivity between our vm intances are 


To connect the mynet-us-vm instance using ssh run the cmd- gcloud compute ssh mynet-us-vm --zone=us-central1-c 

To verify connectivity:
	-ping mynet-eu-vm instance using the internal IP address by running the cmd- ping -c 3 10.132.0.6 (This will work 		because of the allow-internal firewall rule.)

	-ping mynet-eu-vm instance unsing the external IP address by running the cmd- ping -c 3 mynet-eu-vm (This will not work because the vm can't resolve the internal ip address using the vm name, this is where the --hostname flag we added to our vm becomes useful)

-Let's try to resolve this:
	1, Make a copy of hostname which you added to mynet-us-vm when creating the vm on a notepad ... you'll need it later or you   	 can get it from the CLI by running the cmd- hostname -A
	2, Exit mynet-us-vm by typing exit at the prompt.
	3, ssh into mynet-eu-vm uing the cmd- gcloud compute ssh mynet-eu-vm --zone=europe-west1-c
	4, open the resolv.conf file using the cmd- sudo vi /etc/resolv.conf
	5, Copy the hostname name from your nopepad, you will only copy the "[ZONE].c.[PROJECT_ID].internal" part of the 		   hostname make sure you dont copy the dot before the zone the hostname you will paste in resolv.conf file will now look like this "us-central1-c.c.myvpcnetworking-project.internal"
	6, Edit the resolv.conf file by adding the hostname you copied to the end of the search option in the resolv.conf file.
	7, It should now look like this "search europe-west1-c.c.myvpcnetworking-project.internal. c.myvpcnetworking-project.internal. google.internal us-central1-c.c.myvpcnetworking-project.internal"
	8, Save and exit the resol.conf file
	9, Now ssh into mynet-us-vm from the cmd line using the cmd- gcloud compute ssh mynet-us-vm --zone=us-central1-c 
	10,ping mynet-eu-vm again (This will work because we have added the hostname to search and the internal DNS can resolve)

Convert the network to a custom mode network:

-To convert mynetwork to a custom mode network run the cmd- gcloud compute networks update mynetwork --switch-to-custom-subnet-mode


##Task 2. Create custom mode networks:
We will create two additional custom networks, managementnet and privatenet, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic and VM instances 

	-To create the managementnet network run the cmd- gcloud compute networks create managementnet --subnet-mode=custom

	-To create the managementnetsubnet run the cmd- gcloud compute networks subnets create managementsubnet-us --network=managementnet --region=us-central1 --range=10.130.0.0/20

Create the privatenet network:
	-To create the privatenet network, run the cmd- gcloud compute networks create privatenet --subnet-mode=custom

	-To create the privatesubnet-us subnet run the cmd- gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

	-To create the privatesubnet-eu subnet run the cmd- gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

	-To list the available VPC networks run the cmd- gcloud compute networks list

	-To list the available VPC subnets sorted by VPC network run the cmd- gcloud compute networks subnets list --sort-by=network

Create icmp, ssh & rdp firewall rules for managementnet:

	-To create firewall-rules for managementnet run the cmd- gcloud compute firewall-rules create manamgementnet-allow-icmp-ssh-rdp	--action=ALLOW --direction=INGRESS --network=managementnet --rules=icmp,tcp:22,tcp:3389 --priority=1000 --source-ranges=0.0.0.0/0

Create icmp, ssh & rdp firewall rules for privatenet:
	-To create the privatenet-allow-icmp-ssh-rdp firewall rule run the cmd- gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0

To list all the firewall rules sorted by VPC network run the cmd- gcloud compute firewall-rules list --sort-by=network


Next, create two VM instances:
	-managementnet-us-vm in managementsubnet-us

	-privatenet-us-vm in privatesubnet-us

To create managementnet-us-vm in managementsubnet-us run the cmd- gcloud compute instances create managementnet-us-vm 			--machine-type=f1-micro --subnet=managementsubnet --zone=us-central1-c


To create privatenet-us-vm instance run the cmd- gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us

To list all the VM instances sorted by zone run the cmd- gcloud compute instances list --sort-by=zone


##Task 3. Explore the connectivity across networks

To explore connectivity across public internet ping the external IP addresses using the following cmds:

	-Test connectivity to mynet-eu-vm's external IP by running the cmd- ping -c 3 34.77.251.248

	-Test connectivity to managementnet-us-vm's external IP by running the cmd- ping -c 3 35.232.240.114

	-Test connectivity to privatenet-us-vm's external IP by running the cmd- ping -c 3 34.121.168.170


To explore connectivity within the VPC network ping the internal IP addresses using the following cmds:

	-Test connectivity to mynet-eu-vm's internal IP running the cmd- ping -c 3 10.132.0.6

	-Test connectivity to managementnet-us-vm's internal IP by running the cmd- ping -c 3 10.132.0.6

	-Test connectivity to privatenet-us-vm's internal IP by running the cmd- ping -c 3 172.16.0.2


This last 2 pings should not work either, as indicated by a 100% packet loss! You cannot ping the internal IP address of managementnet-us-vm and privatenet-us-vm because they are in separate VPC networks from the source of the ping































  gcloud network-management connectivity-tests create test-icmp --source-instance=mynet-us-vm --destination-instance=mynet-eu-vm --protocol=tcp:22

  gcloud compute instances describe mynet-us-vm --format='get(hostname)'



  gcloud compute networks update NAME [--async] [--bgp-routing-mode=MODE     | --switch-to-custom-subnet-mode] [GCLOUD_WIDE_FLAG …]