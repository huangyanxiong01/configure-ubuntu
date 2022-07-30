# configure-ubuntu
make your ubuntu is great

## Git proxy

```bash
git config --global http.proxy http://127.0.0.1:8889
```

## Ubuntu Vscode color
`/usr/share/applications/code.desktop`
```bash
Exec=/usr/share/code/code --force-color-profile=srgb --unity-launch %F
```
## Install Docker 
```
#!/usr/bin/env bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
sudo docker run hello-world
#sudo groupadd docker
sudo usermod -aG docker ubuntu
newgrp docker
docker run hello-world
echo '{"exec-opts": ["native.cgroupdriver=systemd"]}' | sudo tee /etc/docker/daemon.json # fix kubernetes c group driver
```

## Install kubernetes and init
```bash
#!/usr/bin/env bash

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
sudo kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## V2ray In Server
### nginx
```nginx
server {
  listen       80;
  listen       [::]:80;
  index index.html index.htm index.nginx-debian.html;
  server_name  your_domain_name;
  root /var/www/html;

  location / {
    try_files $uri $uri/ =404;
  }

  location /v2 {
    proxy_redirect off;
    proxy_pass http://127.0.0.1:31291;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    # proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
Now,setup HTTPS certificates with [certbot](https://certbot.eff.org/)

### v2ray config.json
```json
{
  "log": {
    "loglevel": "warning"
  },
  "stats": {},
  "api": {
    "services": [
      "StatsService"
    ],
    "tag": "api"
  },
  "policy": {
    "levels": {
      "1": {
        "handshake": 4,
        "connIdle": 300,
        "uplinkOnly": 2,
        "downlinkOnly": 5,
        "statsUserUplink": false,
        "statsUserDownlink": false
      }
    },
    "system": {
      "statsInboundUplink": true,
      "statsInboundDownlink": true
    }
  },
  "allocate": {
    "strategy": "always",
    "refresh": 5,
    "concurrency": 3
  },
  "inbounds": [
    {
      "port": 31291,
      "listen": "127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "a4f7ef9b-6951-2397-098d-bb1e660b3805",
            "alterId": 0,
            "email": "your_name@gmail.com"
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/v2"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {
      }
    }
  ],
  "routing": {
    "settings": {
      "rules": [
        {
          "inboundTag": [
            "api"
          ],
          "outboundTag": "api",
          "type": "field"
        }
      ]
    },
    "strategy": "rules"
  }
}
```
Now, install v2ray
```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```
## V2ray In Client
```
vmess://ws+tls:a4f7ef9b-6951-2397-098d-bb1e660b3805-0@your_domain_name:443/?path=/v2&tlsServerName=your_domain_name#your_domain_name
```

youtube-dl config
```
-f bestvideo+bestaudio[ext=m4a]/bestvideo+bestaudio/best
--proxy http://127.0.0.1:8889
--merge-output-format mp4
-o /media/common/myfreax/youtube/%(title)s/%(title)s.%(ext)s
```
