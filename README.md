# Week 9 Codepath Honeypot Deployment

Honeypot Setup using Google Cloud Platform
-

In this assignment, I will be explaining the process of setting up multiple honeypots using Google Cloud Platform. The reasons you may want to deploy honeypots include: trying to find out if there is anyone trying to gain access into your systems/doing something suspicious such as scanning for open ports, what tools those attackers are using, how exactly they are getting in, etc. It is a way to collect information from attackers without harming your actual network. 

Getting Started: Downloading and installing the GCP SDK on your local machine
-
To get started, we will need to download and install the GCP SDK on our machine. using [this](https://cloud.google.com/sdk/install)
 link, we can download it, and after it is installed, we will initialize it so the region and zone are correct. To initialize it, run the command ```gcloud init```. Set the appropriate region and zone for you depending on where you are located, then to confirm your settings, run the command ```gcloud config list```.

![Initializing](https://github.com/sevvp/Week-9-Codepath/blob/master/Initializing%20GCP%20SDK.gif)

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
NOTE: If you are having issues with this step, check out [this](https://github.com/threatstream/mhn/issues/521) github link that explains how to fix it.

Now that the script is running, towards the end it will ask if you want to run in debug mode, type "n". Then, enter a superuser email and password for the MHN application. After that, it will ask for some values. We just want the default values, so click "Enter" for each of them, and type "n" for using TLS and SSL for email. Once that part is done, it will ask you if you want to integrate with splunk and if you want to install ELK, type "n" for both. Now, our VM should be all set up and ready to use. However, we need to allow HTTP traffic for our VM. Visit console.cloud.google.com in your browser of choice, log in, on the left pane click "compute engine" and you will see your mhn-admin VM. Click on the title of it, allow HTTP traffic, and save your changes. Now, visit the external IP in a browser of your choice, but make sure the URL starts with HTTP, not HTTPS (we only allowed HTTP traffic.) You may log in using your superuser email and password you made earlier, and see what the UI looks like and what it offers.
 
Create a MHN Honeypot VM
-
Now, it is time to deploy a honeypot! We will deploy the dionaea over HTTP honeypot, which is used to trap malware samples. We will need to create another VM for it. To do this, we will enter in two commands to configure our firewall and deploy the VM:
```
gcloud beta compute firewall-rules create mhn-allow-honeypot --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=all --source-ranges=0.0.0.0/0 --target-tags=mhn-honeypot
gcloud compute instances create "mhn-honeypot-1" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-1"
```
Once this is done, SSH into your honeypot VM. To do this, enter the command ```gcloud compute ssh mhn-honeypot-1```. 

Install the Honeypot Application
-
Now that we have our honeypot VM, we have to install a honeypot onto it. To do this, go back to your MHN dashboard, click "Deploy" at the top, and select the "Ubuntu - Dionaea with HTTP" script. Once you select it, you will see the deployment command below it. It starts with "wget." Copy this command, and run it in your honeypot VM terminal. Now, your honeypot is being installed. Once it is done, if you did everything correctly, you should be able to go back to your MHN dashboard, at the top click "sensors" then "view sensors" and see your honeypot listed.

Testing the Honeypot with nmap
-
To test if the honeypot if working properly, we will use nmap to run a port scan on the honeypot's IP. To do this, run the command ```nmap {external IP of your honeypot}```
You will see three ports open, and you can switch back to your MHN dashboard, click "attacks" on the top, and you will see your IP address along with several port scan records. This proves that the honeypot is working properly. MHN will continue gathering more attacks, they will be originating from all over the world.
![nmap](https://github.com/sevvp/Week-9-Codepath/blob/master/nmap%20Scan.gif)

Results
-
After leaving two honeypots running for four days (dionaea and snort), I captured 17,000+ attacks. The attacks came in under many protocols, but the main ones that I saw were "pcap" (packet capture) and httpd. I got attacks from all around the world, and it seems that the majority of them are just attackers scanning for open and vulnerable ports on my honeypot. 
![Attacks](https://github.com/sevvp/Week-9-Codepath/blob/master/Attacks.png)
![Payloads](https://github.com/sevvp/Week-9-Codepath/blob/master/Payloads.png)

My session.json file:
-
[session.json](https://github.com/sevvp/Week-9-Codepath/blob/master/session.json)
