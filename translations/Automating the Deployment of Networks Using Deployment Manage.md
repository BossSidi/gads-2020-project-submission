# Create templates for the networks
# Create folder

mkdir dmnet
cd dmnet

# To create a new file, click File > New File.

# Name the new file autonetwork-template.jinja, and then open it.

resources:
- name: [RESOURCE_NAME]
  type: [RESOURCE_TYPE]
  properties:
    #RESOURCE properties go here


# To get a list of all available network resource types in Google Cloud, run the following command in Cloud Shell:

gcloud deployment-manager types list | grep network

# In autonetwork-template.jinja, replace [RESOURCE_TYPE] with compute.v1.network.

# Add the following property to autonetwork-template.jinja:

autoCreateSubnetworks: true
 

 # This 

 resources:
- name: {{ env["name"] }}
  type: compute.v1.network
  properties:
    #RESOURCE properties go here
    autoCreateSubnetworks: true



# Create the custom-mode network template
# To create a new file, click File > New File.

# Name the new file customnetwork-template.jinja, and then open it.

# Copy the existing code from autonetwork-template.jinja to customnetwork-template.jinja:

resources:
- name: {{ env["name"] }}
  type: compute.v1.network
  properties:
    autoCreateSubnetworks: true

# In customnetwork-template.jinja, change the autoCreateSubnetworks property from true to false.

resources:
- name: {{ env["name"] }}
  type: compute.v1.network
  properties:
    autoCreateSubnetworks: false


# Create the subnetwork template
# To create a new file, click File > New File.

# Name the new file subnetwork-template.jinja, and then open it.

# Copy the following base code into subnetwork-template.jinja:

resources:
- name: {{ env["name"] }}
  type: [RESOURCE_TYPE]
  properties:
    #RESOURCE properties go here

# To get a list of all available subnetwork resource types in Google Cloud, run the following command in Cloud Shell:

gcloud deployment-manager types list | grep subnetwork

# Locate compute.v1.subnetwork, which is the type needed to create a VPC subnetwork using Deployment Manager.

# In subnetwork-template.jinja, replace [RESOURCE_TYPE] with compute.v1.subnetwork.

# Add the following properties for the VPC subnetwork in subnetwork-template.jinja:

ipCidrRange: {{ properties["ipCidrRange"] }}
network: {{ properties["network"] }}
region: {{ properties["region"] }}

# Verify

resources:
- name: {{ env["name"] }}
  type: compute.v1.subnetwork
  properties:
    ipCidrRange: {{ properties["ipCidrRange"] }}
    network: {{ properties["network"] }}
    region: {{ properties["region"] }}


# Create a template for the firewall rules
# Create the firewall template

# To create a new file, click File > New File.

# Name the new file firewall-template.jinja, and then open it.

# Copy the following base code into firewall-template.jinja:

resources:
- name: {{ env["name"] }}
  type: [RESOURCE_TYPE]
  properties:
    #RESOURCE properties go here


# To get a list of all available firewall rule resource types in Google Cloud, run the following command in Cloud Shell:

gcloud deployment-manager types list | grep firewall 

# In firewall-template.jinja, replace [RESOURCE_TYPE] with compute.v1.firewall.

# In firewall-template.jinja, add the following properties for the firewall rule:

network: {{ properties["network"] }}
sourceRanges: ["0.0.0.0/0"]
allowed:
- IPProtocol: {{ properties["IPProtocol"] }}
  ports: {{ properties["Port"] }}


# Verify that firewall-template.jinja looks like this, including the spacing/indentation:

resources:
- name: {{ env["name"] }}
  type: compute.v1.firewall
  properties:
    network: {{ properties["network"] }}
    sourceRanges: ["0.0.0.0/0"]
    allowed:
    - IPProtocol: {{ properties["IPProtocol"] }}
      ports: {{ properties["Port"] }}


#  Create a template for VM instances
# Create the VM instance template
# To create a new file, click File > New File.

# Name the new file instance-template.jinja, and then open it.

# Copy the following base code into instance-template.jinja:

resources:
- name: {{ env["name"] }}
  type: [RESOURCE_TYPE]
  properties:
    #RESOURCE properties go here


# To get a list of all available instance resource types in Google Cloud, run the following command in Cloud Shell:

gcloud deployment-manager types list | grep instance

# Specify the VM instance properties
# To create a VM instance in the correct zone and network, you need to define these as properties.

# In instance-template.jinja, add the following properties for the VM instance:

 machineType: zones/{{ properties["zone"] }}/machineTypes/{{ properties["machineType"] }}
 zone: {{ properties["zone"] }}
 networkInterfaces:
  - network: {{ properties["network"] }}
    subnetwork: {{ properties["subnetwork"] }}
    accessConfigs:
    - name: External NAT
      type: ONE_TO_ONE_NAT
 disks:
  - deviceName: {{ env["name"] }}
    type: PERSISTENT
    boot: true
    autoDelete: true
    initializeParams:
      sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9





# Verify that instance-template.jinja looks like this, including the spacing/indentation:

resources:
- name: {{ env["name"] }}
  type: compute.v1.instance  
  properties:
     machineType: zones/{{ properties["zone"] }}/machineTypes/{{ properties["machineType"] }}
     zone: {{ properties["zone"] }}
     networkInterfaces:
      - network: {{ properties["network"] }}
        subnetwork: {{ properties["subnetwork"] }}
        accessConfigs:
        - name: External NAT
          type: ONE_TO_ONE_NAT
     disks:
      - deviceName: {{ env["name"] }}
        type: PERSISTENT
        boot: true
        autoDelete: true
        initializeParams:
          sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9



# Create and deploy the configuration

# Import the templates
# A huge configuration file can be difficult to manage. Templates are a way to break down configurations into composable units that can be separately updated and can be reused. Templates are included in the *.yaml configuration using import:.

# To create a new file, click File > New File.

# Name the new file config.yaml, and then open it.

# copy the following into config.yaml:


imports:
- path: autonetwork-template.jinja
- path: customnetwork-template.jinja
- path: subnetwork-template.jinja
- path: firewall-template.jinja
- path: instance-template.jinja


# Configure mynetwork with firewall rules
# Now that you have defined your templates, you need to define all the resources that you want to create from these templates. Start by configuring the mynetwork auto-mode network with its firewall rules.

# Add mynetwork to config.yaml (after the import block):

resources:
- name: mynetwork
  type: autonetwork-template.jinja


# Add a firewall rule to allow HTTP, SSH, and RDP traffic on mynetwork to config.yaml (within the resources block):

- name: mynetwork-allow-http-ssh-rdp
  type: firewall-template.jinja
  properties:
    network: $(ref.mynetwork.selfLink)
    IPProtocol: TCP
    Port: [22, 80, 3389]

# Add a firewall rule to allow ICMP traffic on mynetwork to config.yaml (within the resources block):

- name: mynetwork-allow-icmp
  type: firewall-template.jinja
  properties:
    network: $(ref.mynetwork.selfLink)
    IPProtocol: ICMP
    Port: []


# Configure managementnet with firewall rules
# Next, configure the managementnet custom-mode network with its subnetwork and firewall rules.

# Add managementnet to config.yaml (within the resources block):


- name: managementnet
  type: customnetwork-template.jinja

  # Add the subnetwork for managementnet to config.yaml (within the resources block):

- name: managementsubnet-us
  type: subnetwork-template.jinja
  properties:
    ipCidrRange: 10.130.0.0/20
    network: $(ref.managementnet.selfLink)
    region: us-central1

# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet to config.yaml (within the resources block):

- name: managementnet-allow-http-ssh-rdp
  type: firewall-template.jinja
  properties:
    network: $(ref.managementnet.selfLink)
    IPProtocol: TCP
    Port: [22, 80, 3389]

# Add a firewall rule to allow ICMP traffic on managementnet to config.yaml (within the resources block):

- name: managementnet-allow-icmp
  type: firewall-template.jinja
  properties:
    network: $(ref.managementnet.selfLink)
    IPProtocol: ICMP
    Port: []

# Configure privatenet with firewall rules
# Next, configure the privatenet custom-mode network with its subnetworks and firewall rules.

# Add privatenet to config.yaml (within the resources block):

- name: privatenet
  type: customnetwork-template.jinja

# Add the subnetwork for privatenet to config.yaml (within the resources block):

- name: privatesubnet-us
  type: subnetwork-template.jinja
  properties:
    ipCidrRange: 172.16.0.0/24
    network: $(ref.privatenet.selfLink)
    region: us-central1

# Add another subnetwork for privatenet to config.yaml (within the resources block):

- name: privatesubnet-eu
  type: subnetwork-template.jinja
  properties:
    ipCidrRange: 172.20.0.0/24
    network: $(ref.privatenet.selfLink)
    region: europe-west1

# Add a firewall rule to allow HTTP, SSH, and RDP traffic on privatenet to config.yaml (within the resources block):

- name: privatenet-allow-http-ssh-rdp
  type: firewall-template.jinja
  properties:
    network: $(ref.privatenet.selfLink)
    IPProtocol: TCP
    Port: [22, 80, 3389]

# - name: privatenet-allow-icmp
  type: firewall-template.jinja
  properties:
    network: $(ref.privatenet.selfLink)
    IPProtocol: ICMP
    Port: []

# Configure VM instances in each network
Create the following 4 VM instances:

# mynet-us-vm

# mynet-eu-vm

# privatenet-us-vm

# managementnet-us-vm

# Add the mynet-us-vm instance to config.yaml (within the resources block):

- name: mynet-us-vm
  type: instance-template.jinja
  properties:
    zone: us-central1-a
    machineType: n1-standard-1
    network: $(ref.mynetwork.selfLink)
    subnetwork: regions/us-central1/subnetworks/mynetwork

# Add the mynet-eu-vm instance to config.yaml (within the resources block):

- name: mynet-eu-vm
  type: instance-template.jinja
  properties:
    zone: europe-west1-d
    machineType: n1-standard-1
    network: $(ref.mynetwork.selfLink)  
    subnetwork: regions/europe-west1/subnetworks/mynetwork

# Add the privatenet-us-vm instance to config.yaml (within the resources block):

- name: privatenet-us-vm
  type: instance-template.jinja
  properties:
    zone: us-central1-a
    machineType: n1-standard-1
    network: $(ref.privatenet.selfLink)
    subnetwork: $(ref.privatesubnet-us.selfLink)

# Add the managementnet-us-vm instance to config.yaml (within the resources block):

- name: managementnet-us-vm
  type: instance-template.jinja
  properties:
    zone: us-central1-a
    machineType: n1-standard-1
    network: $(ref.managementnet.selfLink)
    subnetwork: $(ref.managementsubnet-us.selfLink)

# To save config.yaml, click File > Save.
# Deploy the configuration
# It's time to deploy your configuration from Cloud Shell.

# In Cloud Shell, run the following command:

gcloud deployment-manager deployments create gcpnet --config=config.yaml

# If something goes wrong with the deployment, try to use the error messages to troubleshoot the source of the issue. You must delete the Deployment Manager configuration before you can try deploying it again. This can be achieved with this command in Cloud Shell:


gcloud deployment-manager deployments delete gcpnet


# Verify your deployment

# Verify your networks in the Cloud Console
# In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > VPC networks.

# View the managementnet, mynetwork, and privatenet VPC networks with their subnetworks.

# On the Navigation menu, click VPC network > Firewall Rules.

# Sort the firewall rules by Network.

# View the allow-http-ssh-rdp and allow-icmp firewall rules for each VPC network that was created.

# Verify your VM instances in the Cloud Console
# On the Navigation menu (Navigation menu), click Compute Engine > VM instances.

# View the mynet-us-vm, mynet-eu-vm, privatenet-us-vm, and managementnet-us-vm instances.

# Note the internal IP addresses for mynet-eu-vm and privatenet-us-vm.

# For mynet-us-vm, click SSH to launch a terminal and connect.

# To test connectivity to mynet-eu-vm's internal IP address, run the following command in the SSH terminal (replacing mynet-eu-vm's internal IP address with the value noted earlier):

ping -c 3 <Enter mynet-eu-vm's internal IP here>

# To test connectivity to privatenet-us-vm's internal IP address, run the following command in the SSH terminal (replacing mynet-eu-vm's internal IP with the value noted earlier):

ping -c 3 <Enter privatenet-us-vm's internal IP here>

