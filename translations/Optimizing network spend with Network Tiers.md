# Optimizing network spend with Network Tiers


# Create the VM instances 

# Create the Premium Tier VM
gcloud beta compute --project=qwiklabs-gcp-03-753ff608082f instances create vm-premium --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=998502436524-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=vm-premium --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# Create the Standard Tier VM

gcloud beta compute --project=qwiklabs-gcp-03-753ff608082f instances create vm-standard --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --network-tier=STANDARD --maintenance-policy=MIGRATE --service-account=998502436524-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=vm-standard --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# Explore the latency and network paths

# ping the  [premium-IP] 
ping -c 3 <premium-IP>

# ping the the [standard-IP] 

ping -c 3 <standard-IP>

