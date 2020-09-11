# Create custom mode VPC networks with firewall rules
# Create two custom networks, managementnet and privatenet, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic.


gcloud compute networks create managementnet --project=qwiklabs-gcp-00-1562baef01f3 --subnet-mode=custom --bgp-routing-mode=regional

gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-00-1562baef01f3 --range=10.130.0.0/20 --network=managementnet --region=us-central1


# Create the privatenet network


gcloud compute networks create privatenet --subnet-mode=custom


# create the privatesubnet-us subnet:

gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

# create the privatesubnet-eu subnet:

gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

# list the available VPC networks:

gcloud compute networks list

# list the available VPC subnets (sorted by VPC network):

gcloud compute networks subnets list --sort-by=NETWORK


# Create the firewall rules for managementnet
# Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network


gcloud compute --project=qwiklabs-gcp-00-1562baef01f3 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0


# Create the firewall rules for privatenet

gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

# list all the firewall rules (sorted by VPC network):

gcloud compute firewall-rules list --sort-by=NETWORK


