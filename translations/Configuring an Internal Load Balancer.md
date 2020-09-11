#  Configure internal traffic and health check firewall rules.

# Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range

gcloud compute --project=qwiklabs-gcp-04-a89b45b84c6c firewall-rules create fw-allow-lb-access --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=all --source-ranges=10.10.0.0/16 --target-tags=backend-service


# Create the health check rule

gcloud compute --project=qwiklabs-gcp-04-a89b45b84c6c firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=backend-service



 # Create a NAT configuration using Cloud Router

 # Property	Value (type value or select option as specified)
# Gateway name	nat-config
# VPC network	my-internal-app
# Region	us-central1
# Click Cloud Router, and select Create new router.

# For Name, type nat-router-us-central1.

# Click Create.



# Configure instance templates and create instance groups



gcloud beta compute --project=qwiklabs-gcp-04-a89b45b84c6c instance-templates create instance-template-1 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-04-a89b45b84c6c/regions/us-central1/subnetworks/subnet-a --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=91514727525-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=backend-service --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# Create another instance template for subnet-b by copying instance-template-1:

gcloud beta compute --project=qwiklabs-gcp-04-a89b45b84c6c instance-templates create instance-template-2 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-04-a89b45b84c6c/regions/us-central1/subnetworks/subnet-b --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=91514727525-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=backend-service --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-2 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# Create the managed instance groups
# Create a managed instance group in subnet-a (us-central1-a) and subnet-b (us-central1-b).

# Intance group us-central1-a
gcloud compute --project=qwiklabs-gcp-04-a89b45b84c6c instance-groups managed create instance-group-1 --base-instance-name=instance-group-1 --template=instance-template-1 --size=1 --zone=us-central1-a

gcloud beta compute --project "qwiklabs-gcp-04-a89b45b84c6c" instance-groups managed set-autoscaling "instance-group-1" --zone "us-central1-a" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"

# Intance group us-central1-b

gcloud compute --project=qwiklabs-gcp-04-a89b45b84c6c instance-groups managed create instance-group-2 --base-instance-name=instance-group-2 --template=instance-template-2 --size=1 --zone=us-central1-b

gcloud beta compute --project "qwiklabs-gcp-04-a89b45b84c6c" instance-groups managed set-autoscaling "instance-group-2" --zone "us-central1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"


# Verify the backends
# Verify that VM instances are being created in both subnets and create a utility VM to access the backends' HTTP sites.


gcloud beta compute --project=qwiklabs-gcp-04-a89b45b84c6c instances create utility-vm --zone=us-central1-f --machine-type=f1-micro --subnet=subnet-a --private-network-ip=10.10.20.50 --no-address --maintenance-policy=MIGRATE --service-account=91514727525-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=utility-vm --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# To verify the welcome page for instance-group-1-xxxx, run the following command:

curl 10.10.20.2

# To verify the welcome page for instance-group-2-xxxx, run the following command:

curl 10.10.30.2


# Close the SSH terminal to utility-vm:

exit



# Configure the internal load balancer

#  Test the internal load balancer

# To verify that the internal load balancer forwards traffic, run the following command:

curl 10.10.30.5


# Run the same command a couple of times:

curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5


