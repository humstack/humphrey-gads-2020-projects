# LAB: Implement Private Google Access and Cloud NAT

## Objectives

In this lab, you learn how to perform the following tasks:

    - Configure a VM instance that doesn't have an external IP address
    - Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel
    - Enable Private Google Access on a subnet
    - Configure a Cloud NAT gateway
    - Verify access to public IP addresses of Google APIs and services and other connections to the internet

## Steps

1. Configure a VM instance that doesn't have an external IP address

    - In case the project is not set. The project id can be obtained from the qwiklabs lab launch page.

        gcloud config set project {PROJECT-ID}

    - Create a VPC network and firewall rules

        gcloud compute networks create privatenet --project=$DEVSHELL_PROJECT_ID --subnet-mode=custom --bgp-routing-mode=regional

        gcloud compute networks subnets create privatenet-us --project=$DEVSHELL_PROJECT_ID --network=privatenet --range=10.130.0.0/20 --region=us-central1

        gcloud compute firewall-rules create privatenet-allow-ssh --network privatenet --allow tcp:22 --source-ranges 35.235.240.0/20

    - Create the VM instance with no public IP address

        gcloud compute instances create vm-internal --zone=us-central1-c --machine-type=n1-standard-1 --network=privatenet --subnet=privatenet-us --no-address

2. Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel

    - SSH to vm-internal to test the IAP tunnel

        gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

        Result: Type y to accept to save the ssh key. Type and repeat the passphrase to generate the key.

    - To test the external connectivity of vm-internal, run the following command:

        ping -c 2 www.google.com

        Result: This should not work

    - To return to your Cloud Shell instance, run the following command:

        exit

3. Enable Private Google Access on a subnet

    - Create a Cloud Storage bucket

        gsutil mb gs://$DEVSHELL_PROJECT_ID

    - Copy an image file into your bucket

        gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://$DEVSHELL_PROJECT_ID

    - Access the image from your VM instance

        gsutil cp gs://$DEVSHELL_PROJECT_ID/*.svg .

    - Connect to vm-internal

        gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

    - Try copying the image to vm-internal

        gsutil cp gs://{PROJECT-ID}/*.svg .

        Result: This should not work: vm-internal can only send traffic within the VPC network because Private Google Access is disabled (by default).

    - Press Ctrl+C to stop the request.

    - To return to your Cloud Shell instance, run the following command:

        exit

    - Enable Private Google Access

        gcloud compute networks subnets update privatenet-us --enable-private-ip-google-access --region us-central1

    - Connect to vm-internal

        gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

    - Try copying the image to vm-internal

        gsutil cp gs://{PROJECT-ID}/*.svg .

        Result: This should work because vm-internal's subnet has Private Google Access enabled!

    - To return to your Cloud Shell instance, run the following command:

        exit

4. Configure a Cloud NAT gateway

    - Try to update the VM instance

        sudo apt-get update

        Result: This should work because Cloud Shell has an external IP address!

    - Connect to vm-internal

        gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

    - Try to update the VM instance

        sudo apt-get update

        Result: This should only work for Google Cloud packages because vm-internal only has access to Google APIs and services!

    - To return to your Cloud Shell instance, run the following command:

        exit

    - Configure a Cloud NAT gateway

        gcloud compute routers create nat-router --network=privatenet --region=us-central1

        gcloud compute routers nats create nat-config --router=nat-router --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges --router-region us-central1

5. Verify access to public IP addresses of Google APIs and services and other connections to the internet

    - Connect to vm-internal

        gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

    - Try to update the VM instance

        sudo apt-get update

        Result: This should work because vm-internal is using the NAT gateway!

    - To return to your Cloud Shell instance, run the following command:

        exit

    - Enabling logging

        gcloud compute routers nats update nat-config --enable-logging

    - Generating logs

        - Connect to vm-internal

            gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

        - Try to update the VM instance

            sudo apt-get update

        - To return to your Cloud Shell instance, run the following command:

            exit
