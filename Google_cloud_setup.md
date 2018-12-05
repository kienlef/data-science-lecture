# Google GPU environment easy setup:

https://cloud.google.com/deep-learning-vm/docs/quickstart-marketplace

Google cloud SDK:

https://cloud.google.com/sdk/docs/

---
# Google GPU environment manual installation:

1. Upgrade to paid account and request quota for Tesla K-80 Gpu on the Google Cloud Platform(GCP).

IAM & Admin → Quotas:
Region: choose a zone with NVIDIA K80 GPU and Intel Broadwell CPU.
Select NVIDIA K80 GPUs (without “preemptible”) → Edit Quotas → Change to “1” → Submit Request.
Receive email approval of quota increase

---
2. Create your virtual machine instance on GCP


Compute Engine → VM instances → Create
Cores: 4 vCPU  
Memory: 26 GB  
CPU platform: Intel Broadwell or later 
GPUs: 1 with NVIDIA Tesla K80 
Boot disk: Ubuntu 16.04 LTS, 250 GB (SSD charges extra money), I used 250 GB
Firewall: Check 'Allow HTTP/HTTPS traffic'

---
3. Remove current kernel to install old one to avoid conflicts with cuda drivers(If installing cuda 9.1 on ubuntu 16.04) .

 sudo apt-get install -y linux-image-4.4.0-112-generic
 sudo apt-get purge -y linux-image-4.15.0-1023-gcp 
 sudo update-grub
 sudo reboot
  
---
4. Check if the right Kernal is installed

uname -r
---
5. Script to install cuda drivers, docker , nvidia docker and Anaconda. Save it in home directory and run it:
```
#!/bin/bash

# First you must install the 4.4.0 kernel:
# $ sudo apt-get install linux-image-4.4.0-112-generic
# find all the other kernels and remove them:
# $ sudo apt-get purge linux-image-4.15.0-1023-gcp
# $ sudo update-grub
# $ sudo reboot
```
```
sudo apt-get update && sudo apt-get install -y \
	build-essential \
	apt-transport-https \
	ca-certificates \
	curl \
	software-properties-common \
	linux-headers-$(uname -r)

#### Install Nvidia CUDA

wget -nc https://developer.nvidia.com/compute/cuda/9.1/Prod/local_installers/cuda_9.1.85_387.26_linux
sudo sh cuda_9.1.85_387.26_linux -silent
rm -rf cuda_9.1.85_387.26_linux

#### Install Docker

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update && sudo apt-get install -y docker-ce

#### Install Nvidia Docker

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get -qq update
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

#### Build Docker image
cat << EOF > Dockerfile.tensorflow_gpu_jupyter
FROM tensorflow/tensorflow:latest-gpu
RUN apt-get update && apt-get install -y python-opencv python-skimage git
RUN pip install requests ipywidgets seaborn
RUN jupyter nbextension enable --py widgetsnbextension
RUN git clone git://github.com/keras-team/keras.git && pip install keras[tests] && rm -rf keras
CMD ["/run_jupyter.sh", "--allow-root", "--NotebookApp.token=''", "--NotebookApp.password='sha1:c05e4b238956:8ac2f926ab754e180ec201ce0485b2a55f679ceb'"]
EOF

sudo docker build -t tensorflow_gpu_jupyter -f Dockerfile.tensorflow_gpu_jupyter .
sudo nvidia-docker run -dit --restart unless-stopped -p 8888:8888 tensorflow_gpu_jupyter

echo "The password to the notebook is 'tensorflow_gpu_jupyter'"
```

---
6. download and install google cloud sdk ( Necessary to open jupyter documents on local systems )

gcloud init to connect to your instance.


---
7. Run the following command (replace your instance name with the actual name of your instance) to  map port 8888 of local machine to port 8888 of gcp cloud instance.

gcloud compute ssh <your instance name> --ssh-flag=“-L” --ssh-flag=“8888:localhost:8888”


---
8. Navigate to localhost:8888 on your browser and enter "tensorflow_gpu_jupyter" as password (Just the first time) to work with your jupyter notebooks from local machines.
