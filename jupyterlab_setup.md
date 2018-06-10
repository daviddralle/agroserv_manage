# AgroServ computing environment setup

Tutorial for getting setup on Google Compute Engine or a home machine. 

## Getting setup on Google Compute Engine

1. Install the Google Cloud Software Developer’s Kit at [Cloud SDK](https://cloud.google.com/sdk), making sure to use the email address in Step 1. Please follow all the steps in the install tutorial. During the install, you should be able to set the default project to AgroServ because I will have added you as an editor to the AgroServ Google Cloud project. Don’t worry about setting a default region/zone, etc.

2. Google Cloud Console. Create a VM with the following attributes: 
  * Name it something like `agroserv-vm-yourGmailAddress`
  * Set `region` to `us-west1`
  * For boot disk, choose Ubuntu 16 (Xenial), change drive size to 20GB
  * Check `Allow http traffic` and `Allow https traffic`
  * Click on `Managment, disks, network, SSH keys` and go down to `Startup script`. Paste in this script, replacing your Google username with your own.

```
#!/bin/bash
google_user=daviddralle
user_drive=/home/$google_user

if [ -x "$(command -v docker)" ]; then
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
4. Now, the connection step. It's easy. Your computer will listen on http://localhost:1111 for the Jupyter Lab docker. 
```
gcloud beta compute ssh agroserv-vm-daviddralle -- -L 1111:localhost:8888
```
5. Once we're in, we will want to do some installing and authorizing for the tools we want. 

```
curl https://sdk.cloud.google.com | bash
# You may need to restart a terminal after the first one
gcloud auth login
gcloud config set project AgroServe-SOMENUMBER
mkdir ~/bucket
gcsfuse agroserve_bucket ~/bucket
gcloud iam service-accounts keys create service_account_credentials.json --iam-account agroserve@agroserve-202921.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file service_account_credentials.json
gcsfuse --key-file /home/jovyan/service_account_credentials.json agroserve_bucket ~/bucket
echo "alias gcs='gcsfuse --key-file /home/jovyan/service_account_credentials.json agroserve_bucket ~/bucket'" >> ~/.bashrc
earthengine authenticate
python -c "import ee; ee.Initialize()"
R -e 'IRkernel::installspec()'
echo "alias gcs='gcsfuse agroserve_bucket ~/bucket'" >> ~/.bash_aliases
```

## Getting setup at home
1. make an `agroserve_project` folder somewhere on your computer. This will be the equivalent of the home directory on your virtual machine.  
2. Within the above folder, `git clone https://github.com/daviddralle/agroserv`
3. Install [Docker](https://www.docker.com/get-docker)
4. Get the docker machine `docker pull gcr.io/agroserve-202921/agroserv`
5. Make sure you are in the `agroserve_project` folder! Do not go into the `agroserve` folder. This is very important because the `agroserv` folder will be the git repository for all of our work. You do not want the Docker running in there, because it would save sensitive account specific information in a folder that we'd like to keep sync'd with Github. Now, run: 
```
sudo docker run --privileged --rm -p 8888:8888 -v "$PWD":/home/jovyan gcr.io/agroserve-202921/agroserv
```

6. Repeat step 5 from the previous section to get the Docker setup just right. 

