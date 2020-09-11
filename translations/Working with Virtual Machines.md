# Working with Virtual Machines


# Create the VM

gcloud beta compute --project=qwiklabs-gcp-01-14aea41382cc instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --address=35.222.141.30 --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=301367857289-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --image=debian-10-buster-v20200714 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-01-14aea41382cc/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any


# Prepare the data disk
# Create a directory and format and mount the disk
# For mc-server, click SSH to open a terminal and connect.
# To create a directory that serves as the mount point for the data disk, run the following command:

sudo mkdir -p /home/minecraft

# To format the disk, run the following command:
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk

# To mount the disk, run the following command:

sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft

# Install and run the application

# Install the Java Runtime Environment (JRE) and the Minecraft server
# In the SSH terminal for mc-server, to update the Debian repositories on the VM, run the following command:

sudo apt-get update

# After the repositories are updated, to install the headless JRE, run the following command:

sudo apt-get install -y default-jre-headless

# To navigate to the directory where the persistent disk is mounted, run the following command:

cd /home/minecraft

# To install wget, run the following command:

sudo apt-get install wget

# To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:

sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar

# Initialize the Minecraft server
# To initialize the Minecraft server, run the following command:

sudo java -Xmx1024M -Xms1024M -jar server.jar nogui

# To see the files that were created in the first initialization of the Minecraft server, run the following command:

sudo ls -l

# To edit the EULA,

sudo nano eula.txt

# Create a virtual terminal screen to start the Minecraft server

# To install screen

sudo apt-get install -y screen

# To start your Minecraft server in a screen virtual terminal, run the following command: (Use the -S flag to name your terminal mcs)

sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui

# Detach from the screen and close your SSH session
sudo screen -r mcs
#  exit the SSH terminal
exit



# Schedule regular backups
export YOUR_BUCKET_NAME=<Enter your bucket name here>
echo $YOUR_BUCKET_NAME
 
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
# Create a backup script
cd /home/minecraft
sudo nano /home/minecraft/backup.sh

#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
endBach
 
sudo chmod 755 /home/minecraft/backup.sh
. /home/minecraft/backup.sh
 
sudo crontab -e
 
0 */4 * * * /home/minecraft/backup.sh

# Server maintenance
 
 
sudo screen -r -X stuff '/stop\n'



