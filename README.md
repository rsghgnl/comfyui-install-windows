to run:
### Windows
```
wsl -d Debian

```
### Debian
```
docker compose up -d  # Starts the container
docker exec -it comfy-docker bash  # Opens an interactive bash shell inside the running container
docker compose stop  # Stops the container 
docker ps -a # list images
```
### Docker
```
cd ~/ComfyUI
source comfy-env/bin/activate
```
### Python venv
```
python3 main.py --listen --output-directory /workspace/comfy/output
```

### Windows

```
wsl hostname -I
```

# Installation

### Windows
Get wsl up and running and create a Debian instance
```
wsl --list --online
wsl --list
wsl --install Debian
wsl --unregister {{Distribution-Name}}
wsl --export <DistroName> <BackupFilePath.tar> # make a backup
wsl --import <NewDistroName> <InstallLocation> <BackupFilePath.tar>  # restore a backup
wsl --import Debian-01 ../installs Debian-Comfy-0109.tar
wsl --export Debian Debian-Comfy-0109.tar # make a backup
```
make sure Debian has access to enough ram<br>
create this file: C:\\Users\\{user}\\.wslconfig
```
[wsl2]
memory=54GB
```
### Debian
#### https://docs.docker.com/engine/install/debian/
#### https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
```
cd
nvidia-smi
sudo apt update
sudo apt upgrade -y
nano install-docker.sh
```
copy from docker documentation
and run (somthing like this install-docker.sh) 
```
cat << 'EOF' > install-docker.sh
#!/bin/bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
EOF
```
then:
```
sudo sh install-docker.sh
sudo docker run hello-world # install image
sudo docker images # list images
sudo docker pull debian # explicitly install container ??
sudo docker run -it --name comfy-docker debian # install docker container for comfyui
exit
sudo docker stop comfy-docker
sudo docker kill comfy-docker
sudo docker rm comfy-docker
sudo docker ps -a # list images

```
install the nvidia drivers:

```
cat << 'EOF' > install-nvidia.sh
#!/bin/bash

sudo apt install gnupg

curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update

export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
sudo apt-get install -y \
    nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
EOF
```
```
sudo sh install-nvidia.sh
sudo service docker restart
```
create a docker image for comfy-ui<br>
```
cat << 'EOF' > docker-compose.yml
services:
  comfy-docker:
    image: debian
    container_name: comfy-docker
    ports:
      - "8188:8188"
    volumes:
      - /mnt/c/comfy:/workspace/comfy
    stdin_open: true     # equivalent to -it
    tty: true
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

EOF
```
then start docker
```
docker compose up -d  # Starts the container
docker exec -it comfy-docker bash  # Opens an interactive bash shell inside the running container
docker compose stop  # Stops the container 
docker ps -a # list images
```

### docker
#### https://pytorch.org/get-started/locally/
#### https://github.com/comfyanonymous/ComfyUI
```
apt update
python --version # might not be installed
apt install python3 python3-pip python3-venv -y
python3 --version
apt install git -y
git clone https://github.com/comfyanonymous/ComfyUI.git
apt install nano -y
cd ComfyUI
nano requirements.txt
pip install gguf
pip install opencv-python
pip install diffusers
sudo apt install ffmpeg -y
sudo apt install libgl1 -y
```
remove:
torch
torchsde
torchvision
torchaudio
save
```
python3 -m venv comfy-env
source comfy-env/bin/activate
pip install typing-extensions
```
check the pythorh documentation for the next line
```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu129
pip install torchsde
pip install -r requirements.txt
```
### comfyui
create this file: ~/ComfyUI/extra_model_paths.yaml
```
comfyui:
  base_path: /workspace/comfy
  checkpoints: models/checkpoints/
  clip: models/clip/
  clip_vision: models/clip_vision/
  configs: models/configs/
  controlnet: models/controlnet/
  diffusion_models: |
                models/diffusion_models
                models/unet
  embeddings: models/embeddings/
  loras: models/loras/
  upscale_models: models/upscale_models/
  vae: models/vae/
  text_encoders: models/text_encoders
  unet: models/unet


my_custom_nodes:
  custom_nodes: /workspace/comfy/custom_nodes
```


