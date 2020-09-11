# Implement Private Google Access and Cloud NAT 

# Create the VM instance 
# Create a VPC network and firewall rules
 
 # Network
gcloud compute networks create privatenet --project=qwiklabs-gcp-02-c4323b6d6459 --subnet-mode=custom --bgp-routing-mode=regional

gcloud compute networks subnets create privatenet-us --project=qwiklabs-gcp-02-c4323b6d6459 --range=10.130.0.0/20 --network=privatenet --region=us-central1

# Firewall

gcloud compute --project=qwiklabs-gcp-02-c4323b6d6459 firewall-rules create privatenet-allow-ssh --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20


# Create the VM instance with no public IP address

gcloud beta compute --project=qwiklabs-gcp-02-c4323b6d6459 instances create vm-internal --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatenet-us --no-address --maintenance-policy=MIGRATE --service-account=724238923170-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=vm-internal --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# SSH to vm-internal to test the IAP tunnel
# To connect to vm-internal

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

# To test the external connectivity of vm-internal 

ping -c 2 www.google.com


# Enable Private Google Access

# Create a Cloud Storage bucket
# bucket 

gsutil mb gs://[my_bucket]

# Copy an image file into your bucket

gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://[my_bucket]

# In Cloud Shell, to try to copy the image from your bucket, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .

# To connect to vm-internal

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

# To try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .

# Enable Private Google Access

gcloud compute networks subnets update privatenet-us \
--region=us-central1 \
--enable-private-ip-google-access

# In Cloud Shell for vm-internal, to try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

gsutil cp gs://[my_bucket]/*.svg .

# exit 
exit

# Configure a Cloud NAT gateway 

# Try to update the VM instance
# in Cloud Shell
sudo apt-get update

# To connect to vm-internal, run the following command:

gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

# To try to re-synchronize the package index of vm-internal, run the following command:

sudo apt-get update

# Configure a Cloud NAT gateway

gcloud compute routers nats create nat-config \
    --router=nat-router \
    --auto-allocate-nat-external-ips \
    --nat-all-subnet-ip-ranges \
    --enable-logging


# Verify the Cloud NAT gateway
# In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

sudo apt-get update

# To return to your Cloud Shell instance, run the following command:

exit

# Configure and view logs with Cloud NAT Logging

# Generating logs
gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap

# If prompted, type Y to continue.

# Try to re-synchronize the package index of vm-internal by running the following:

sudo apt-get update

# Exit 
exit 