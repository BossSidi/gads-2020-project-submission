#  Dynamic VPN gateways with Cloud Routers

# Create the first network
gcloud compute networks create gcp-vpc --project=qwiklabs-gcp-02-cc5103123585 --description=Dynamic\ VPN\ gateways\ with\ Cloud\ Routers --subnet-mode=custom --bgp-routing-mode=regional

gcloud compute networks subnets create subnet-a --project=qwiklabs-gcp-02-cc5103123585 --range=10.5.4.0/24 --network=gcp-vpc --region=us-central1


# Create the second network

gcloud compute networks create on-prem --project=qwiklabs-gcp-02-cc5103123585 --description=on-prem\ network --subnet-mode=custom --bgp-routing-mode=regional

gcloud compute networks subnets create subnet-b --project=qwiklabs-gcp-02-cc5103123585 --range=10.1.3.0/24 --network=on-prem --region=europe-west1


# Create the utility VMs


gcloud beta compute --project=qwiklabs-gcp-02-cc5103123585 instances create gcp-server --zone=us-central1-a --machine-type=n1-standard-1 --subnet=subnet-a --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=826143334613-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=gcp-server --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any



# Create the second instance


gcloud beta compute --project=qwiklabs-gcp-02-cc5103123585 instances create on-prem-1 --zone=europe-west1-b --machine-type=n1-standard-1 --subnet=subnet-b --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=826143334613-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=on-prem-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any


# Create the firewall rules 


gcloud compute --project=qwiklabs-gcp-02-cc5103123585 firewall-rules create allow-icmp-ssh-gcp-vpc --direction=INGRESS --priority=1000 --network=gcp-vpc --action=ALLOW --rules=tcp:22,icmp --source-ranges=0.0.0.0/0


# Allow traffic to on-prem
# Click Create Firewall Rule.


gcloud compute --project=qwiklabs-gcp-02-cc5103123585 firewall-rules create allow-icmp-ssh-on-prem --direction=INGRESS --priority=1000 --network=on-prem --action=ALLOW --rules=tcp:22,icmp --source-ranges=0.0.0.0/0


# Test connectivity from gcp-server to on-prem-1

# For gcp-server, click SSH to launch a terminal and connect.

ping -c 3 <Enter on-prem-1's external IP address here>

# To test connectivity to on-prem-1's internal IP address, run the following command, replacing on-prem-1's internal IP address with the value noted earlier:

ping -c 3 <Enter on-prem-1's internal IP address here>



# Test connectivity from on-prem-1 to gcp-server

ping -c 3 <Enter gcp-server's external IP address here>

# To test connectivity to gcp-server's internal IP address, run the following command, replacing gcp-server's internal IP address with the value noted earlier:

ping -c 3 <Enter gcp-server's internal IP address here>


# Create the Cloud Routers


gcloud compute routers create gcp-vpc-cr --project=qwiklabs-gcp-02-cc5103123585 --region=us-central1 --network=gcp-vpc --asn=65470


# Create the on-prem Cloud Router

gcloud compute routers create on-prem-cr --project=qwiklabs-gcp-02-cc5103123585 --region=europe-west1 --network=on-prem --asn=65503



#  Prepare for VPN Gateways configuration

# gcp-server
gcloud compute addresses create gcp-vpc-ip --project=qwiklabs-gcp-02-cc5103123585 --region=us-central1

# on-prem
gcloud compute addresses create on-prem-ip --project=qwiklabs-gcp-02-cc5103123585 --region=europe-west1


# Create the first VPN

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" target-vpn-gateways create "vpn-1" --region "us-central1" --network "gcp-vpc"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-1-rule-esp" --region "us-central1" --address "34.122.247.97" --ip-protocol "ESP" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-1-rule-udp500" --region "us-central1" --address "34.122.247.97" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-1-rule-udp4500" --region "us-central1" --address "34.122.247.97" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" vpn-tunnels create "vpn-1-tunnel-1" --region "us-central1" --peer-address "34.76.228.228" --shared-secret "gcprocks" --ike-version "2" --target-vpn-gateway "vpn-1"


# Create the second VPN
35.226.199.25

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" target-vpn-gateways create "vpn-2" --region "europe-west1" --network "on-prem"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-2-rule-esp" --region "europe-west1" --address "104.155.63.170" --ip-protocol "ESP" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-2-rule-udp500" --region "europe-west1" --address "104.155.63.170" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" forwarding-rules create "vpn-2-rule-udp4500" --region "europe-west1" --address "104.155.63.170" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-02-cc5103123585" vpn-tunnels create "vpn-1-tunnel-1" --region "europe-west1" --peer-address "35.226.199.25" --shared-secret "gcprocks" --ike-version "2" --target-vpn-gateway "vpn-2"


# For gcp-server, click SSH to launch a terminal and connect.

# To test connectivity to on-prem-1's internal IP address, run the following command, replacing on-prem-1's internal IP address with the value noted earlier:

ping -c 3 <Enter on-prem-1's internal IP address here>


# Exit the gcp-server SSH terminal.

# For on-prem-1, click SSH to launch a terminal and connect.

# To test connectivity to gcp-server's internal IP address, run the following command, replacing gcp-server's internal IP address with the value noted earlier:

ping -c 3 <Enter gcp-server's internal IP address here>

# Demonstrate Dynamic Routing

# Create a new subnet in on-prem

# Property	Value (type value or select option as specified)
# Name	subnet-c
# Region	europe-west1
# IP address range	10.4.2.0/24

# Create a new utility VM in the new subnet

gcloud beta compute --project=qwiklabs-gcp-02-cc5103123585 instances create on-prem-2 --zone=europe-west1-c --machine-type=n1-standard-1 --subnet=subnet-c --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=826143334613-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=on-prem-2 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any


# Test connectivity

# For gcp-server, click SSH to launch a terminal and connect.

ping -c 3 <Enter on-prem-2's internal IP address here>