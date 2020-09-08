# Setting Up Development Environme Commands

## Task 1: Creating a Compute Engine Virtual Machine Instance

#### Create the dev virtual machine
gcloud beta compute --project=qwiklabs-gcp-03-ab1383c938f0 instances create dev-instance --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=111049558943-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=dev-instance --reservation-affinity=any

#### Configure the firewall rules to allow http traffic
gcloud compute --project=qwiklabs-gcp-03-ab1383c938f0 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

#### SSH Into the dev-instance
gcloud beta compute ssh --zone "us-central1-a" "dev-instance" --project "qwiklabs-gcp-03-ab1383c938f0"


## Task 2: Install software on the VM instance

sudo apt-get update
sudo apt-get install git
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt install nodejs

## Task 3: Configure the VM to Run Application Software

To check the version of Node.js, execute the following command:
node -v

To clone the class repository, execute the following command:
git clone https://github.com/GoogleCloudPlatform/training-data-analyst

To change the working directory, execute the following command:
cd ~/training-data-analyst/courses/developingapps/nodejs/devenv/

To run a simple web server, execute the following command:
sudo node server/app.js

Note the external IP address of the instance using command:
gcloud compute instances list

Open your browers to:
http://<external_ip>

Return to the SSH window, and stop the application by pressing Ctrl+C.

To install the Node.js library for Compute Engine, execute the following command:
npm install

To run a simple Node.js application that lists Compute Engine instances, execute the following command:
node list-gce-instances.js