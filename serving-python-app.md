## using standalone runing app
```
uvicorn server:app --host 0.0.0.0 --port 443 \
    --ssl-keyfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/privkey.pem \
    --ssl-certfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/fullchain.pem
```
## using pm2 running app
```
pm2 start "uvicorn server:app --host 0.0.0.0 --port 443 --ssl-keyfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/privkey.pem --ssl-certfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/fullchain.pem" --name pdrm_demo
```

## backend server preparation
https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu
https://docs.nvidia.com/cuda/cuda-quick-start-guide/index.html#linux

```
sudo apt install python3 python3-pip ffmpeg python3.10-venv -y
sudo apt install python3.10-venv
sudo apt install ubuntu-drivers-common
sudo ubuntu-drivers install --gpgpu
nvidia-smi
lspci | grep -i nvidia
# sudo apt-key del 7fa2af80
# wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
# sudo dpkg -i cuda-keyring_1.1-1_all.deb
# sudo apt-get update
# sudo apt-get install cuda-toolkit
# sudo apt-get install -y nvidia-open
# sudo apt-get install nvidia-gds

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo  apt-get -y install cuda-toolkit-12-6

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cudnn

apt-get install -y nvidia-open

sudo reboot
nvidia-smi

export PATH=/usr/local/cuda-12.6/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.6/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```
```
# Create a virtual environment (optional but recommended)
sudo python3 -m venv demoenv
source demoenv/bin/activate

# Install pip libraries
pip install fastapi uvicorn faster-whisper webrtcvad googletrans==4.0.0-rc1 numpy websockets starlette
#Note: googletrans==4.0.0-rc1 is a specific version needed for Google Translator compatibility.
pip install starlette


# Copy the python code and run it
uvicorn server:app --host 0.0.0.0 --port 443 \
    --ssl-keyfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/privkey.pem \
    --ssl-certfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/fullchain.pem

# Or to use pm2
pm2 start "uvicorn server:app --host 0.0.0.0 --port 443 --ssl-keyfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/privkey.pem --ssl-certfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/fullchain.pem" --name pdrm_demo

pm2 start "uvicorn server:app --host 0.0.0.0 --port 443 --ssl-keyfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/privkey.pem --ssl-certfile /etc/letsencrypt/live/pdrmdemo.pixlr.com/fullchain.pem" --name pdrm_demo -i max
```
test websocket is running using this command
```
wscat -c wss://pdrmdemo.pixlr.com/ws
```
```
# pm2 auto restart
pm2 startup
pm2 save
```
```
sudo chmod 755 /home
sudo chmod 755 /home/ubuntu
sudo chmod 755 /home/ubuntu/work
sudo chmod -R 755 /home/ubuntu/work/mallam_v1

sudo chmod 644 /home/ubuntu/work/mallam_v1/index.html
sudo chmod -R 755 /home/ubuntu/work/mallam_v1/docs

sudo apt update
sudo apt install certbot python3-certbot-nginx -y
sudo certbot certonly --standalone -d pdrmdemo.pixlr.com


wscat -c wss:/pdrmdemo.pixlr.com/ws
Connected (press CTRL+C to quit)
```

### Create a test script test.py to test CUDA driver is working
```
import torch
print(torch.cuda.is_available())
```

## Deploy to S3 and invalidate
```
aws s3 cp ./mallam_v1/ s3://pixlr-playground-apse5/ --recursive \
  --include "index.html" \
  --include "login.html" \
  --include "pixlr-logo.jpeg" \
  --include "js/*" \
  --include "css/*"
  --profile pixlr_development
```
```
aws cloudfront create-invalidation \
    --distribution-id E1AI4LGLL1EF7B \
    --paths "/css/*" "/js/*" "/index.html" \
    --profile pixlr_development

aws cloudfront create-invalidation \
    --distribution-id E1AI4LGLL1EF7B \
    --paths "/logo.png" \
    --profile pixlr_development
```
