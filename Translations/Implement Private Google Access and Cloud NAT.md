#Lab Title: Implement Private Google Access and Cloud NAT

##OBJECTIVE:
- Configure a VM instance that doesn't have an external IP address
- Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel
- Enable Private Google Access on a subnet
- Configure a Cloud NAT gateway
- Verify access to public IP addresses of Google APIs and services and other connections to the internet


#Task 1. Create the VM instance 
To achieve the above task we are required to:
  (1) Create a VPC network, subnet and firewall rules
  (2) Create a VM instance with no external IP address
  (3) Connect to the instance using an IAP tunnel.

To create a VPC network run the cmd- gcloud compute networks create privatenet --subnet-mode=custom

To create a subnet in the privatenet network run the cmd- gcloud compute networks subnets create privatenet-us --range=10.130.0.0/20 --network=privatenet --region=us-central1

To create firewall rules for this network run the cmd- gcloud compute firewall-rules create privatenet-allow-ssh --action=ALLOW --direction=INGRESS --network=privatenet --rules=tcp:22 --source-ranges=35.235.240.0/20

To create a VM instance with no external IP address run the cmd- gcloud compute instances create vm-internal --machine-type=n1-standared-1 --zone-us-central-c --subnet=privatenet-us --no-address

To connect the isntance using IAP tunnel run the cmd- gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap ----- (N.B: If prompted about continuing, type Y. When prompted for a passphrase, press ENTER. When prompted for the same passphrase, press ENTER)

To test the external connectivity of vm-internal run the cmd- ping -c 2 www.google.com


#Task 2. Enable Private Google Access
To achieve the above task I will:
 (1) Create a storage bucket with location set to multi-region
 (2) Copy an image file to my bucket. 
 (3) Access the image from the VM created earlier (vm-internal)

 To create a storage bucket to hold the image run the cmd- gsutil mb -l us gs://bosun225

 To copy an image file to my bucket run the cmd- gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://bosun225

 Let try to access the image from our VM with private google access disabled and cloudNAT not configured:

	Try to copy the image to cloud shell with the cmd- gsutil cp gs://bosun225/*.svg . (This should work because clod shell has an external IP address)

	To connect to vm-internal, run the following cmd- gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap ----- (N.B: If prompted about continuing, type Y. When prompted for a passphrase, press ENTER. When prompted for the same passphrase, press ENTER)

	I'll try to copy the image to vm-internal, using the cmd- gsutil cp gs://bosun225/*.svg . (This should not work: vm-internal can only send traffic within the VPC network because Private Google Access is disabled by default) 

	Press Ctrl+C to stop the request. 

Now I will enable private google access:
	To enable private google access run the cmd- gcloud compute networks subnets update privatenet-us --region=us-central1 --enable-private-ip-google-access

	Let's try to access the image from our VM with private google access enabled.

		Connect to the VM "vm-internal" from cloud shell by running the cmd- gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

		Now copy the image file to the vm using the cmd- gsutil cp gs://bosun225/*.svg . (This should work because private google access has been enabled)

		Now I will configure cloudNAT gateway

#Task 3. Configure a Cloud NAT gateway
First, you will confirm if the vm instances have access to the internet for updates and patches.
	From the cloud shell i'll run the cmd- sudo apt-get update   (This will work because cloud shell has an external IP address)

	From cloud shell ssh into vm-internal using the cmd- gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

	Next, run the cmd- sudo apt-get update (This will not work because vm-internal only has access to google APIs and services) 

Now lets configure a Cloud NAT gateway: 

First create a router for your cloudNAT
To create a router run the cmd- gcloud compute routers create nat-router --network=privatenet --region=us-central1

To configure and cloudNAT gateway run the cmd- gcloud compute routers nats create nat-config --router=nat-router --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges --region=us-central1

Verify that the cloudNAT gateway is functional-
	From cloud shell ssh into vm-internal using the cmd- gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap (so that you are running your cmds from the vm instance)

	Next, run the cmd- sudo apt-get update (This will work now that cloudNAT has been configured)

	Exit from the vm instance "vm-internal" and also exit cloud shell.