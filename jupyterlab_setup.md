# AgroServ computing environment setup

Tutorial for getting setup on Google Compute Engine or a home machine. 

## Getting setup on Google Compute Engine

1. Install the Google Cloud Software Developer’s Kit at [Cloud SDK](https://cloud.google.com/sdk), making sure to use the email address in Step 1. Please follow all the steps in the install tutorial. During the install, you should be able to set the default project to AgroServ because I will have added you as an editor to the AgroServ Google Cloud project. Don’t worry about setting a default region/zone, etc.

2. Google Cloud Console. Create a VM with the following attributes: 
  * Name it something like `agroserv-vm-yourGmailAddress`
  * Set `region` to `us-west1`
  * For boot disk, choose Ubuntu 16 (Xenial), change drive size to 20GB
  * Check `Allow http traffic` and `Allow https traffic`
  * Click on `Managment, disks, network, SSH keys` and go down to `Startup script`. Paste in this script, replacing your Google username with your own. If we ever need new software, better whatever, the Docker container can be re-compiled and pushed to the cloud. The startup script on the VM will then automatically run that updated Docker. 

```
#!/bin/bash
google_user=daviddralle
user_drive=/home/$google_user

if [ -x "$(command -v docker)" ]; then
	# If the VM has already been setup, check to see if there is an update to the Docker, then run it
	sudo docker pull gcr.io/agroserve-202921/agroserv
	sudo docker run --name jlab --privileged --rm -p 8888:8888 -v "$user_drive":/home/jovyan gcr.io/agroserve-202921/agroserv
else
	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
	sudo apt-get update
	sudo apt-get install -y docker-ce
	sudo usermod -aG docker $google_user
	sudo docker pull gcr.io/agroserve-202921/agroserv
	sudo docker run --privileged --rm -p 8888:8888 -v "$user_drive":/home/jovyan gcr.io/agroserve-202921/agroserv
fi
```

  * Press `Create`

3. Now give it some time. It will take a while to download and build the docker with Jupyter Lab. 
4. Now, the connection step. It's easy. Your computer will listen on http://localhost:8888 for the Jupyter Lab docker. 
```
gcloud beta compute ssh agroserv-vm-daviddralle -- -L 8888:localhost:8888
```
5. You may be prompted for a password, which I have set to `agroserve`. Once Jupyter Lab is open, we will want to do some installing and authorizing for the tools we'll be using (Earth Engine, etc). Open a terminal in the Jupyter Lab launcher and run the following lines of code (run this code one or two lines at a time; you'll be following some printed links to authenticate at several points). 

```
curl https://sdk.cloud.google.com | bash  
# You may need to open a new terminal after this command if the terminal doesn't 

# login and set to the agroserve project
gcloud auth login
gcloud config set project agroserve-202921

# this is the location where we'll mount the agroserve cloud storage bucket
mkdir ~/bucket

# authenticating a service account to use the cloud storage bucket mounting tool (gcsfuse)
gcloud iam service-accounts keys create /home/jovyan/service_account_credentials.json --iam-account agroserve@agroserve-202921.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file /home/jovyan/service_account_credentials.json

# connect to the bucket
gcsfuse --key-file /home/jovyan/service_account_credentials.json agroserve_bucket ~/bucket

# list files in the bucket to see that it worked
ls ~/bucket

# optionally, create a shortcut to the above command
# this makes a shortcut 'gcs', which you can type directly into the command line to mount the bucket for your next session
echo "alias gcs='gcsfuse --key-file /home/jovyan/service_account_credentials.json agroserve_bucket ~/bucket'" >> ~/.bashrc

# authenticate earth engine
earthengine authenticate

# run this; if you get no output or errors, then earth engine is working!
python -c "import ee; ee.Initialize()"

# Make sure that Jupyter Lab knows that R is installed
# On this first startup, you may not have seen the option to start an R kernel
# from now on, you will
R -e 'IRkernel::installspec()'
```

## Getting setup at home
1. make an `agroserve_project` folder somewhere on your computer. This will be the equivalent of the home directory on your virtual machine.  
2. Open a terminal and `cd` into `agroserve_project`, then run `git clone https://github.com/daviddralle/agroserv`
3. Make sure that you have [Docker](https://www.docker.com/get-docker) on your computer. 
4. Get the docker machine for the agroserv project by running this command: `docker pull gcr.io/agroserve-202921/agroserv`. This will take quite a while because the docker is 7GB! So make sure you have the space on your machine. 
5. For this step, make double triple sure you are in the `agroserve_project` folder! Do not run the below command in `agroserve_project/agroserv` folder. This is very important because the `agroserv` folder will be the git repository for all of our work. You do not want the Docker running in there, because we will endup saving sensitive account specific information for using Google Cloud API's that you probably don't want on GitHub. You can check that you're in the right folder with the command `echo $PWD`. Now, run: 
```
sudo docker run --privileged --rm -p 8888:8888 -v "$PWD":/home/jovyan gcr.io/agroserve-202921/agroserv
```

6. Go to `http://localhost:8888` in a browser to open Jupyter Lab. You may need to input the default password, `agroserve`. 
7. To authenticate Google Cloud stuff, go through [step 5](#getting-setup-on-google-compute-engine) in the previous section. 
