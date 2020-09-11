# Create a Deployment Manager deployment
 
 # Export your zone in bash
 export MY_ZONE=us-central1-b

# At the Cloud Shell prompt, download an editable Deployment Manager template:

gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml

# In the Cloud Shell, use the sed command to replace the PROJECT_ID placeholder string with your Google Cloud Platform project ID using this command:

sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml

# In the Cloud Shell, use the sed command to replace the ZONE placeholder string with your Google Cloud Platform zone using this command:

sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml

# View the mydeploy.yaml file, with your modifications, with this command:

cat mydeploy.yaml

# Build a deployment from the template:

gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml


# Update a Deployment Manager deployment

# Launch the nano text editor to edit the mydeploy.yaml file:

nano mydeploy.yaml

# Find the line that sets the value of the startup script, value: "apt-get update", and edit it so that it looks like this:

value: "apt-get update; apt-get install nginx-light -y"


# Enter this command to cause Deployment Manager to update your deployment to install the new startup script:

gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml

# View the Load on a VM using Cloud Monitoring

# In the ssh session on my-vm, execute this command to create a CPU load:

dd if=/dev/urandom | gzip -9 >> /dev/null &


