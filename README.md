# kubernetes
Kubernetes-Installation unter VirtualBox

# Szenario 1:
Einfache Installation. Die Server erhalten alle eine IP vom Router und haben nur einen Netzwerkadapter im Modus "Bridge".

## Voraussetzungen:
* Ubuntu-Image v16.04.4
* master01 mit mind. 2 CPU, IP 192.168.170.10
* worker01 IP 192.168.170.11
* worker02 IP 192.168.170.12
* Alle virtuellen Maschinen haben Internetzugriff per Port 80 und 443.

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
## Auf Master01:
Kubernetes installieren. Dabei das Pod-Netzwerk und Advertise-Netzwerk fest vorgeben:

```
kubeadm init --apiserver-advertise-address=192.168.170.10 --pod-network-cidr=10.244.0.0/16
```

Die Installation dauert. Dem Ganzen ruhig 10 Minuten Zeit geben.

Nach der Installation erhält man einen `kubeadmin join`-Befehl mit einem Token. Diesen muss man sich kopieren und später auf den Nodes ausführen. 

Bevor man weitermacht die Konfiguration entweder seinem eigenen User oder einem eigenen Kubernetes-User übergeben, damit man nicht als Root arbeiten muss:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Ab sofort alle weiteren Befehle auf dem Master **mit diesem User** ausführen.

Danach kann der Network-Controller hinzugefügt werden. Eine Auswahl an möglichen Controllern gibt es hier: https://kubernetes.io/docs/concepts/cluster-administration/addons/.

Ich habe mich für Calico entschieden, weil es viele Dinge vereinfacht:

```
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml
```
Nun kann man mit `watch kubectl get pods --all-namespaces` zuschauen wie die Pods erstellt werden. Sie müssen alle im Status "running" laufen.

Wenn alles läuft werden die Worker-Nodes eingerichtet.

## Auf den Worker-Nodes

Als Root: Hier wird der o. g. `kubeadm join`-Befehl ausgeführt. Danach wechselt man wieder zum Master

## Auf dem Master zurück
Die Installation mit `kubectl get nodes` überprüfen.

Fertig
