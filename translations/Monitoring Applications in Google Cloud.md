# Step 1: 

# Download a sample app from Github
# Download a sample application from GitHub and preview it in Cloud Shell.


# To create a folder called gcp-logging, run the following command:

mkdir gcp-logging

## Change to the folder you just created:

cd gcp-logging

## Clone a simple Python Flask app from Github:

git clone https://GitHub.com/GoogleCloudPlatform/training-data-analyst.git


## Change to the deploying-apps-to-gcp folder:

cd training-data-analyst/courses/design-process/deploying-apps-to-gcp

## In Cloud Shell, click Launch code editor (Cloud Shell Editor).

## Expand the gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp folder in the navigation pane, and then click main.py to open it.

## Add the following import statement at the top of the file (line 2):

import googlecloudprofiler

# After the main() function, add the following code snippet to start Profiler (after line 11):

try:
    googlecloudprofiler.start(verbose=3)
except (ValueError, NotImplementedError) as exc:
    print(exc)


## You also have to add the Profiler library to your requirements.txt file. Open that file in the code editor and add the following:

google-cloud-profiler

## Profiler has to be enabled in the project. In Cloud Shell, enter the following command:

gcloud services enable cloudprofiler.googleapis.com

## To test the program, enter the following commands to install requirements and then start the program:

sudo pip3 install -r requirements.txt

python3 main.py


# To see the program running, click Web Preview (Web Preview) in the Google Cloud Shell toolbar. Then select Preview on port 8080.
# The program should be displayed in a new browser tab.



# Step 2 : Deploy an application to App Engine
## Now you will deploy the program to App Engine and use Google Cloud tools to monitor it.

# In the Cloud Shell code editor, in the Explorer pane, select the gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp folder.

# On the File menu, click New File, and then name the file app.yaml.

# Paste the following into the file you just created:

runtime: python37


## Save your changes.
# In a project, an App Engine application has to be created. This is done just once using the gcloud app create command and specifying the region where you want the app to be created. Enter the following command:

gcloud app create --region=us-central

## Now deploy your app with the following command:

gcloud app deploy --version=one --quiet


## Step 3: View Profiler information
## In the Cloud Console, on the Navigation menu (Navigation menu), click Profiler. The screen should look similar to this:


## On the Navigation menu, click Compute Engine.

## Click Create to create a virtual machine.

# Change the region to someplace other than us-central1 (the App Engine app is in us-central1). Accept all the rest of the defaults and click Create.

# When the VM is ready, click SSH to log in to it.

# You will generate some traffic to your App Engine app using the web testing tool called Apache Bench. Enter the following commands to install it:


sudo apt update

sudo apt install apache2-utils -y

# Enter the following command to generate some traffic to your App Engine application:

ab -n 1000 -c 10 https://<your-project-id>.appspot.com/


# Enter the ab command again

ab -n 1000 -c 10 https://<your-project-id>.appspot.com/

