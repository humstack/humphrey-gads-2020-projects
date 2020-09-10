# LAB VPC Networking

## Objectives

In this lab, you learn how to perform the following tasks:

    - Explore the default VPC network
    - Create an auto mode network with firewall rules
    - Convert an auto mode network to a custom mode network
    - Create custom mode VPC networks with firewall rules
    - Create VM instances using Compute Engine
    - Explore the connectivity for VM instances across VPC networks

## Steps

1. Explore the default VPC network

    - In case the project is not set. The project id can be obtained from the qwiklabs lab launch page.

        gcloud config set project {PROJECT-ID}

    - View the subnets

        gcloud compute networks subnets list | grep default

    - View the routes

        gcloud compute routes list | grep default

    - View the firewall rules

        gcloud compute firewall-rules list | grep default

    - Delete the Firewall rules

        gcloud compute firewall-rules delete default-allow-icmp default-allow-internal default-allow-rdp default-allow-ssh

        Type Y to confirm deletion

    - Delete the default network

        gcloud compute networks delete default

        Type Y to confirm deletion

    - Verify there are no routes

        gcloud compute routes list

        Result: 0 results

    - Verify there are no firewall rules

        gcloud compute firewall-rules

        Result: 0 results

    - Try to create a VM instance

        gcloud compute instances create vpc-vm --zone=us-central1-c

        Result: VM creation fails

2. Create an auto mode network with firewall rules

    - Create an auto mode VPC network with firewall rules

        gcloud compute networks create mynetwork --subnet-mode=auto

        gcloud compute firewall-rules create mynetwork-allow-internal --network mynetwork --allow tcp,udp,icmp --source-ranges 0.0.0.0/0

        gcloud compute firewall-rules create mynetwork-allow-rdp-ssh-icmp --network mynetwork --allow tcp:22,tcp:3389,icmp

    - Create a VM instance in us-central1

        gcloud compute instances create mynet-us-vm --machine-type=n1-standard-1 --network=mynetwork --zone=us-central1-c

    - Create a VM instance in europe-west1

        gcloud compute instances create mynet-eu-vm --machine-type=n1-standard-1 --network=mynetwork --zone=europe-west1-c

    - Verify connectivity for the VM instances

        gcloud compute ssh mynet-us-vm --tunnel-through-iap --zone=us-central1-c

        Type Y to confirm creation of ssh directory and enter passphrase twice for confirmation

        Result: Successful connection

    - Ping the Europe VM using its internal and external IPs as well as its name

        ping -c 3 {mynet-eu-vm-internal-ip}

        Result: This should work

        ping -c 3 mynet-eu-vm

        Result: This should work

        ping -c 3 {mynet-eu-vm-external-ip}

        Result: This should work

    - To close and return to cloud shell
        exit

3. Convert an auto mode network to a custom mode network

    - Convert the network to a custom mode network

        gcloud compute networks update mynetwork --switch-to-custom-subnet-mode

        Type Y to confirm

4. Create custom mode VPC networks with firewall rules

    - Create the managementnet network

        gcloud compute networks create managementnet --subnet-mode=custom

        gcloud compute networks subnets create managementsubnet-us --region=us-central1 --network managementnet --range=10.130.0.0/20

    - Create the privatenet network

        gcloud compute networks create privatenet --subnet-mode=custom

        gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

        gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

    - List all networks

        gcloud compute networks list

    - List all subnets by network

        gcloud compute networks subnets list --sort-by=NETWORK

    - Create the firewall rules for managementnet

        gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --network managementnet --direction=INGRESS --priority=1000 --source-ranges 0.0.0.0/0 --allow tcp:22,tcp:3389,icmp

    - Create the firewall rules for privatenet

        gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

    - List all firewall rules

        gcloud compute firewall-rules list --sort-by=NETWORK

5. Create VM instances using Compute Engine

    - Create the managementnet-us-vm instance

        gcloud compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=f1-micro --network=managementnet --subnet=managementsubnet-us

    - Create the privatenet-us-vm instance

        gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us

    - List all running VMs

        gcloud compute instances list --sort-by=ZONE

    - Connect to mynet-eu-vm using ssh

        gcloud compute ssh mynet-us-vm --tunnel-through-iap --zone=us-central1-c

        Result: Successful connection

    - Test connectivity to mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm external IPs

        ping -c 3 <mynet-eu-vm's external IP here>

        Result: This should work!

        ping -c 3 <managementnet-us-vm's external IP here>

        Result: This should work!

        ping -c 3 <privatenet-us-vm's external IP here>

        Result: This should work!

    - Test connectivity to mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm internal IPs

        ping -c 3 <mynet-eu-vm's internal IP here>

        Result: You can ping the internal IP address of mynet-eu-vm because it is on the same VPC network as the source of the ping (mynet-us-vm), even though both VM instances are in separate zones, regions, and continents!

        ping -c 3 <managementnet-us-vm's internal IP here>

        Result: This should not work, as indicated by a 100% packet loss!

        ping -c 3 <privatenet-us-vm's internal IP here>

        Result: This should not work, as indicated by a 100% packet loss!
