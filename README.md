# kubernetes
Kubernetes-Installation unter VirtualBox

# Szenario 1:
Einfache Installation. Die Server erhalten alle eine IP vom Router und haben nur einen Netzwerkadapter im Modus "Bridge".

## Voraussetzungen:
* Ubuntu-Image v16.04.4
* master01 mit mind. 2 CPU, IP 192.168.170.10
* worker01 IP 192.168.170.11
* worker02 IP 192.168.170.12

## Vorbereitung auf allen Servern
* Swap ausschalten mit `swapoff -a` und Entfernen der Swap-Partition in `/etc/fstab`.
* Docker installieren:
```
sudo apt-get install docker.io
sudo systemctl enable docker
sudo service docker start
```
* Docker-cgorup-Treiber auf systemd stellen
```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
```
* Restart docker.
```
systemctl daemon-reload
systemctl restart docker
```

* Kubernetes installieren:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main"|tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install kubeadm 
```
