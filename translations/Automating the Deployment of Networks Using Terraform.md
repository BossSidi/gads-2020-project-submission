# Set up Terraform and Cloud Shell
# Configure your Cloud Shell environment to use Terraform.

terraform --version

# Create a directory for your Terraform configuration by running the following command:

mkdir tfnet

# In Cloud Shell, click Open editor (Cloud Shell Editor).

# Expand the tfnet folder in the left pane of the code editor.

# Initialize Terraform
# Terraform uses a plugin-based architecture to support the numerous infrastructure and service providers available. Each "provider" is its own encapsulated binary distributed separately from Terraform itself. Initialize Terraform by setting Google as the provider.

# To create a new file, click File > New File.

# Name the new file provider.tf, and then open it.

# Copy the code into provider.tf:

provider "google" {}

# Initialize Terraform by running the following command:

cd tfnet
terraform init

# Create managementnet and its resources
# Create the custom-mode network managementnet along with its firewall rule and VM instance (managementnet-us-vm).

# Configure managementnet
# Create a new configuration and define managementnet.

# To create a new file, click File > New File.

# Name the new file managementnet.tf, and then open it.

# Copy the following base code into managementnet.tf:

# Create the managementnet network
resource [RESOURCE_TYPE] "managementnet" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}

# In managementnet.tf, replace [RESOURCE_TYPE] with "google_compute_network"

# In managementnet.tf, replace [RESOURCE_NAME] with "managementnet"

# Add the following property to managementnet.tf:

auto_create_subnetworks = "false"

# Verify that managementnet.tf looks like this:

 # Create the managementnet network
 resource "google_compute_network" "managementnet" {
 name = "managementnet"
 auto_create_subnetworks = "false"
 }

 
 # Add a subnet to managementnet
# Add managementsubnet-us to the VPC network.

# Add the following resource to managementnet.tf:

# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
name          = "managementsubnet-us"
region        = "us-central1"
network       = google_compute_network.managementnet.self_link
ip_cidr_range = "10.130.0.0/20"
}


# Configure the firewall rule
# Define a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet

# Add the following base code to managementnet.tf:

# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource [RESOURCE_TYPE] "managementnet-allow-http-ssh-rdp-icmp" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}

# In managementnet.tf, replace [RESOURCE_TYPE] with "google_compute_firewall"

# In managementnet.tf, replace [RESOURCE_NAME] with "managementnet-allow-http-ssh-rdp-icmp"

# Add the following property to managementnet.tf:

network = google_compute_network.managementnet.self_link


# Add the following properties to managementnet.tf:

allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }

# Verify that your additions to managementnet.tf look like this:

# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
name = "managementnet-allow-http-ssh-rdp-icmp"
network = google_compute_network.managementnet.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
}

# Configure the VM instance
# Define the VM instance by creating a VM instance module. A module is a reusable configuration inside a folder. You will use this module for all VM instances of this lab.

# To create a new folder inside tfnet, select the tfnet folder, and then click File > New Folder.
# Name the new folder instance.
# To create a new file inside instance, select the instance folder, and then click File > New File.
# Name the new file main.tf, and then open it.
# Copy the following base code into main.tf:


resource [RESOURCE_TYPE] "vm_instance" {
  name = [RESOURCE_NAME]
  #RESOURCE properties go here
}

# In main.tf, replace [RESOURCE_TYPE] with "google_compute_instance"
# In main.tf, replace [RESOURCE_NAME] with "${var.instance_name}"


# Add the following properties to main.tf:

  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"


# Add the following properties to main.tf:

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }

  # Add the following properties to main.tf:

  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }

# verify 

resource google_compute_instance "vm_instance" {
 variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_subnetwork" {}

resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
}

# To save main.tf, click File > Save.

# Add the following VM instance to managementnet.tf:

# Add the managementnet-us-vm instance
module "managementnet-us-vm" {
  source           = "./instance"
  instance_name    = "managementnet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}

# Create managementnet and its resources

terraform fmt

# Initialize Terraform by running the following command:

terraform init

# Create an execution plan by running the following command:

terraform plan

# Apply the desired changes by running the following command:

terraform apply

# Confirm the planned actions by typing:
yes


# Create privatenet and its resources
# Create the custom-mode network privatenet along with its firewall rule and VM instance (privatenet-us-vm).

# Configure privatenet
# Create a new configuration and define privatenet.

# To create a new file, click File > New File.
# Name the new file privatenet.tf, and then open it.


# Add the VPC network by copying the following code into privatenet.tf:

# Create privatenet network
resource "google_compute_network" "privatenet" {
name                    = "privatenet"
auto_create_subnetworks = false
}

# Add the privatesubnet-us subnet resource to privatenet.tf:

# Create privatesubnet-us subnetwork
resource "google_compute_subnetwork" "privatesubnet-us" {
name          = "privatesubnet-us"
region        = "us-central1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.16.0.0/24"
}

# Add the privatesubnet-eu subnet resource to privatenet.tf:

# Create privatesubnet-eu subnetwork
resource "google_compute_subnetwork" "privatesubnet-eu" {
name          = "privatesubnet-eu"
region        = "europe-west1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.20.0.0/24"
}

# Configure the firewall rule
# Define a firewall rule to allow HTTP, SSH, and RDP traffic on privatenet.

# Add the firewall resource to privatenet.tf:

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
name    = "privatenet-allow-http-ssh-rdp-icmp"
network = google_compute_network.privatenet.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
}

# Configure the VM instance
# Use the instance module to configure privatenet-us-vm.

# Add the VM instance resource to privatenet.tf:


# Add the privatenet-us-vm instance
module "privatenet-us-vm" {
  source           = "./instance"
  instance_name    = "privatenet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}

# Create privatenet and its resources

terraform fmt

# Initialize Terraform by running the following command:

terraform init

# Create an execution plan by running the following command:

terraform plan


# Apply the desired changes by running the following command:

terraform apply



# Create mynetwork and its resources
# Create the auto-mode network mynetwork along with its firewall rule and two VM instances (mynet_us_vm and mynet_eu_vm).

# Configure mynetwork
# Create a new configuration and define mynetwork.

# To create a new file, click File > New File.
# Name the new file mynetwork.tf, and then open it.

# Copy the following code into mynetwork.tf:

# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name                    = "mynetwork"
#RESOURCE properties go here
}

# dd the following property to mynetwork.tf:

auto_create_subnetworks = "true"

# Verify that mynetwork.tf looks like this:

# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name                    = "mynetwork"
auto_create_subnetworks = true
}

# Configure the firewall rule
Define a firewall rule to allow HTTP, SSH, and RDP traffic on mynetwork.

Add the firewall resource to mynetwork.tf:

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
name    = "mynetwork-allow-http-ssh-rdp-icmp"
network = google_compute_network.mynetwork.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
}

# Configure the VM instance
# Use the instance module to configure mynetwork-us-vm and mynetwork-eu-vm.

# Add the following VM instances to mynetwork.tf:

# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}

# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-d"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}

# Create mynetwork and its resources
# It's time to apply the mynetwork configuration.

# Rewrite the Terraform configurations files to a canonical format and style by running the following command:

terraform fmt

# Apply the desired changes by running the following command:

terraform apply


