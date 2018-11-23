# Week-9-Codepath

Honeypot Setup using Google Cloud Platform
-

In this assignment, I will be explaining the process of setting up multiple honeypots using Google Cloud Platform. The reasons you may want to deploy honeypots include: trying to find out if there is anyone trying to gain access into your systems/doing something suspicious such as scanning for open ports, what tools those attackers are using, how exactly they are getting in, etc. It is a way to collect information from attackers without harming your actual network. 

Getting Started: Downloading and installing the GCP SDK on your local machine
-
To get started, we will need to download and install the GCP SDK on our machine. using [this](https://cloud.google.com/sdk/install)
 link, we can download it, and after it is installed, we will initialize it so the region and zone are correct. To initialize it, run ```gcloud init```, then run the command ```gcloud init```. Set the appropriate region and zone for you depending on where you are located, then to confirm your settings, run the command ```gcloud config list```.
 
 Creating the mhn-admin VM
 -
 In this step, we will deploy a virtual machine which will be used to host our honeypot. You will have to use Ubuntu, allow HTTP traffic, and have TCP ports 3000 and 10000 open. To do this, we will use gcloud commands. To create the firewall rule to allow ports 3000 and 1000, we will use the command 
 ```gcloud beta compute firewall-rules create mhn-allow-admin --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:3000,tcp:10000 --source-ranges=0.0.0.0/0 --target-tags=mhn-admin```. Once this is done, we will create the VM itself, titling it "mhn-admin."
 ```
 gcloud compute instances create "mhn-admin" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-admin","http-server","https-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-admin"
 ```
 Now, you should have your VM up and running. To connect to it, we will use SSH. Run the command ```gcloud compute ssh mhn-admin``` and you will have access to the machine. 
 
 Installing the MHN Admin Application
 -
 Now, we will install the MHN Admin Application which is used to deploy honeypots and analyze the data that they collect. First, we will need to update apt and install git on our machine. To do this, run the following commands: 
 ```
 sudo apt-get update
 sudo apt-get install git -y
 ```
Once this is done, we can clone the MHN code into the /opt directory, and run the install.sh file. In order to change into the /opt directory and clone the MHN code, run the following commands: 
 ```
 cd /opt
 sudo git clone https://github.com/RedolentSun/mhn.git
 cd mhn
 sudo ./install.sh
 ```
 
