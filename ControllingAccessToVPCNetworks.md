# Controlling Access to VPC Networks

## Task 1. Create the web servers

### Create the blue server
Create the blue server with a network tag.
gcloud beta compute --project=qwiklabs-gcp-01-d78de7e53527 instances create blue --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1086245059736-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=web-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=blue --reservation-affinity=any

### Create the green server
gcloud beta compute --project=qwiklabs-gcp-01-d78de7e53527 instances create green --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1086245059736-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=green --reservation-affinity=any

### Install nginx and customize the welcome page

ssh into blue using command:
gcloud beta compute ssh --zone "us-central1-a" "blue" --project "qwiklabs-gcp-01-d78de7e53527"

Install ngix
sudo apt-get install nginx-light -y

Run the following command to open the welcome page in the nano editor:
sudo nano /var/www/html/index.nginx-debian.html

Replace the <h1>Welcome to nginx!</h1> line with <h1>Welcome to the blue server!</h1>.

Press CTRL+O, ENTER, CTRL+X.

Run the following command to verify the change:
cat /var/www/html/index.nginx-debian.html

Close the SSH terminal to blue:
exit

For green, launch ssh using command
gcloud beta compute ssh --zone "us-central1-a" "green" --project "qwiklabs-gcp-01-d78de7e53527"

Run the following command to install nginx:
sudo apt-get install nginx-light -y

Run the following command, to open the welcome page in the nano editor:
sudo nano /var/www/html/index.nginx-debian.html

Replace the <h1>Welcome to nginx!</h1> line with <h1>Welcome to the green server!</h1>

Press CTRL+O, ENTER, CTRL+X.

Run the following command to verify the change:
cat /var/www/html/index.nginx-debian.html

Close the SSH terminal to green:
exit

## Task 2. Create the firewall rule

### Create the tagged firewall rule
gcloud compute --project=qwiklabs-gcp-01-d78de7e53527 firewall-rules create allow-http-web-server --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=web-server

### Create a test-vm
gcloud compute instances create test-vm --machine-type=n1-standard-1 --subnet=default --zone=us-central1-b

### Test HTTP connectivity

Use command
gcloud compute instances list

Note the internal and external IP addresses of blue and green

For test-vm, ssh into the server using command:
gcloud beta compute ssh --zone "us-central1-b" "test-vm" --project "qwiklabs-gcp-01-d78de7e53527"

To test HTTP connectivity to blue's internal IP, run the following command, replacing blue's internal IP:
curl <Enter blue's internal IP here>
You should see the Welcome to the blue server! header

To test HTTP connectivity to green's internal IP, run the following command, replacing green's internal IP:
curl <Enter green's internal IP here>
You should see the Welcome to the green server! header.

To test HTTP connectivity to blue's external IP, run the following command, replacing blue's external IP:
curl <Enter blue's external IP here>
You should see the Welcome to the blue server! header.

To test HTTP connectivity to green's external IP, run the following command, replacing green's external IP:
curl <Enter green's external IP here>
This should not work!

## Task 3. Explore the Network and Security Admin roles

Return to the SSH terminal of the test-vm instance.

Run the following command to try to list the available firewall rules:
gcloud compute firewall-rules list
This should not work!

Run the following command to try to delete the allow-http-web-server firewall rule:
gcloud compute firewall-rules delete allow-http-web-server
This should not work!

### Create a service account

gcloud iam service-accounts create network-admin --display-name="Network-admin"

gcloud projects add-iam-policy-binding"qwiklabs-gcp-01-d78de7e53527" \
    --member=user:network-admin@qwiklabs-gcp-01-d78de7e53527.iam.gserviceaccount.com --role=compute-network-admin
    
### Authorize test-vm

Stop test-vm
gcloud compute instances stop test-vm

Set the service account
gcloud compute instances set-service-account test-vm --zone=us-central1-b --service-account=network-admin

### Verify permissions

Run the following command to try to list the available firewall rules:
gcloud compute firewall-rules list
This should work!

Run the following command to try to delete the allow-http-web-server firewall rule:
gcloud compute firewall-rules delete allow-http-web-server
This should not work!

### Update service account and verify permissions

Update the Network-admin service account by providing it with the Security Admin role.
gcloud projects add-iam-policy-binding"qwiklabs-gcp-01-d78de7e53527" \
    --member=user:network-admin@qwiklabs-gcp-01-d78de7e53527.iam.gserviceaccount.com --role=compute-security-admin
    
Run the following command to try to list the available firewall rules:
gcloud compute firewall-rules list
This should work!

Run the following command to try to delete the allow-http-web-server firewall rule:
gcloud compute firewall-rules delete allow-http-web-server
This should work!

### Verify the deletion of the firewall rule
Return to the SSH terminal of the test-vm instance.

To test HTTP connectivity to blue's external IP, run the following command, replacing blue's external IP:
curl <Enter blue's external IP here>
This should not work!