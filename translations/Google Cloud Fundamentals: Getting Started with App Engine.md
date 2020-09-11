
# You can list the active account name with this command:
gcloud auth list

# You can list the project ID with this command:
gcloud config list project

# Initialize App Engine

gcloud app create --project=$DEVSHELL_PROJECT_ID

# Clone the source code repository for a sample application in the hello_world directory:

git clone https://github.com/GoogleCloudPlatform/python-docs-samples

# Navigate to the source directory:

cd python-docs-samples/appengine/standard_python3/hello_world

# Run Hello World application locally

# update the packages list.
sudo apt-get update

# Python virtual environments are used to isolate package installations from the system
sudo apt-get install virtualenv
# If prompted [Y/n], press Y and then Enter.
virtualenv -p python3 venv

# Activate the virtual environment.

source venv/bin/activate

# Navigate to your project directory and install dependencies.

pip install  -r requirements.txt

# Run the application: 
python main.py


# Yo can preview your app in browser on port 8080


# Deploy and run Hello World on App Engine

# Navigate to the source directory:

cd ~/python-docs-samples/appengine/standard_python3/hello_world

# Deploy your Hello World application.

gcloud app deploy

# If prompted "Do you want to continue (Y/n)?", press Y and then Enter.

# Launch your browser to view the app at http://YOUR_PROJECT_ID.appspot.com

gcloud app browse


