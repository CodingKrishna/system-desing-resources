# Course Outline
https://jnaapti.com/downloads/cisco-co.pdf

# Handout
https://jnaapti.com/downloads/cisco-jul-2022.pdf

# Linux Prerequisites
https://jnaapti.com/downloads/linux-prereq.pdf

# CNCF Landscape
https://landscape.cncf.io/

# Virtualbox Download
https://www.virtualbox.org/wiki/Downloads

# Ubuntu Server
https://ubuntu.com/download/server -> Go with Option 2 - Manual server installation

# Ubuntu Desktop
https://ubuntu.com/download/desktop

# Install Ubuntu using Virtualbox
https://brb.nci.nih.gov/seqtools/installUbuntu.html

# When installing the VM ensure that the VM has at least:
4GB RAM
20GB disk space
2 CPUs
No swap space

# Kernel driver not installed error on Mac
https://www.howtogeek.com/658047/how-to-fix-virtualboxs-%E2%80%9Ckernel-driver-not-installed-rc-1908-error/

# Install Guest Additions for screen resolution and for copy/paste to work from host
https://www.howtogeek.com/howto/2845/install-guest-additions-to-windows-and-linux-vms-in-virtualbox/

# Bidirectional Shared Clipboard
https://www.howtogeek.com/187535/how-to-copy-and-paste-between-a-virtualbox-host-machine-and-a-guest-machine/

# Port forwarding in VirtualBox
https://www.techrepublic.com/article/how-to-use-port-forwarding-in-virtualbox/

# Installing OpenSSH server in case you are using the server edition
sudo apt install openssh-server
Edit the file: /etc/ssh/sshd_config
Set:
PasswordAuthentication yes
sudo systemctl restart ssh

# Port forward 22 from VM to your host
# Then login using: ssh <username>@localhost

# Getting your Public IP
https://www.google.com/search?q=what+is+my+ip

# Domain to IP mapping for google.com
https://www.whatsmydns.net/#A/google.com

# Relative ping times
ping virtualcoach.jnaapti.io
ping cloudlab.jnaapti.io
ping mindscape.jnaapti.io

# Ping times across the globe
https://wondernetwork.com/pings
# Set source and destination as NewYork, Bangalore, Singapore and London and compare the relative ping times

# Availability zones and regions
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html

# Containers from Scratch
https://ericchiang.github.io/post/containers-from-scratch/

# Installing Docker in Ubuntu
https://docs.docker.com/engine/install/ubuntu/
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verification
sudo docker --version
sudo systemctl status docker
# Use q to quit the pager (if pager has opened)
sudo ctr --version
sudo systemctl status containerd

# Setting up permissions
docker ps
# The above gives Permission denied
ls -l /var/run/docker.sock
# The user does not belong to the group docker
groups
# Add current (non-root) user to docker group
sudo usermod -aG docker $USER
newgrp docker
# Make sure docker is listed in the list of groups
groups

# We don't need sudo anymore to run docker commands as this regular user
# List all running containers
docker ps
# Alias
docker container ls
# List all images
docker images

# How do we create processes in a machine?
# We can create a process by running a command: eg: ls
# ls is actually a file, where is this file?
which ls
# What kind of a file is ls
ls -l /usr/bin/ls
# It's an executable (x bit is set)
# It's an ELF file
file /usr/bin/ls
less /usr/bin/ls
# ls is a binary executable

# Processes are created by running commands and the commands refer to some binary executable
# Relationship between container and image - image contains the binary that we used to create the containerized process

# We can either build our own custom images or we can use readymade images from the Docker Hub
# Docker Hub
https://hub.docker.com/
# Nginx Docker Image
https://hub.docker.com/_/nginx
docker pull nginx
docker images

# Attempt 1 to create a container
screen
# Inside the screen, run this command
docker run nginx
# We can now see nginx running actively in the terminal, the prompt is not back
^-A-D

# In the main terminal run
docker ps
docker inspect <container-name>
ip addr show
# Look for a docker0 interface in the output of this command
curl <ip-from-above> # Example 172.17.0.2

# In the process manager of the host, I am seeing processes running as containers
ps aux | grep nginx
ps faux

# Go back to the screen
screen -R
# View the access logs in the output
^C
# Terminate the screen
^D
# Back in the main terminal
docker ps
# There is no nginx process running anymore
ps aux | grep nginx
# curl won't work either
curl 172.17.0.2
# The container is still there and the Status says "Exited (0)"
docker ps -a

# Attempt 2
docker run -d --name webserver1 -p 80:80 nginx
docker ps

# We now have a proxy listening on port 80 in the host
ps aux | grep docker-proxy
sudo ss -nltp

# Get your private/public IP
ip addr show
# And then connect to this from your laptop's browser
# If you are using Virtualbox, setup a port forwarding rule for port 80 and then connect to http://localhost from your laptop's browser (host of the VM)
# Worst case you can try this in the VM and this should work
curl localhost

# See the logs
docker logs webserver1

# Stopping the process (equivalent of ^C)
docker stop webserver1

docker ps
docker ps -a

# At this point, the page no longer opens in the browser, since the container is not running
ps aux | grep nginx
ps aux | grep docker-proxy
sudo ss -nltp
curl localhost

# Start the container
docker start webserver1
docker ps
# Observe that the container ID is the same, CREATED time is the same, but Status is more recent
ps aux | grep nginx
ps aux | grep docker-proxy
# The page is accessible in the browser
curl localhost

# Remove stopped containers
docker rm <container-name>

# To remove a running container
docker rm -f <container-name>

# Remove all containers
docker rm -f $(docker ps -aq)

# Restrictions can be applied in docker run itself
docker run --help
docker run --help | grep cpu
docker run --help | grep memory

# Container lifecycle - one step at a time
docker container create --help
docker container create --name webserver1 -p 80:80 nginx
docker ps
docker ps -a
ps aux | grep nginx
docker start webserver1
docker ps
ps aux | grep nginx
curl localhost
docker stop webserver1
docker ps -a
ps aux | grep nginx
docker rm webserver1
docker ps -a

# Similarities between ctr CLI and docker CLI
# Switch to root
sudo su
ctr container ls
ctr c ls
ctr i ls
ctr i pull docker.io/library/nginx:latest
ctr i ls
ctr c create docker.io/library/nginx:latest webserver1
ctr c ls
ctr c info webserver1
ctr tasks
ctr tasks start -d webserver1
ctr tasks ls
ps aux | grep nginx
ps faux
ctr tasks kill webserver1
ctr tasks ls
ps aux | grep nginx
ctr c rm webserver1
ctr c ls
^D

# Relationship between Docker and containerd
docker run -d --name webserver1 -p 80:80 nginx
docker ps
# Docker uses containerd as the underlying container runtime
# We don't see the Docker containers in the ctr c ls output
sudo ctr c ls
# This is because containerd has namespaces and all Docker containers are in a namespace called moby
sudo ctr ns ls
sudo ctr --namespace moby c ls
# Shortform is -n
sudo ctr -n moby c ls
# Compare the container ids of the docker ps and ctr c ls output
# Check the process id and compare with ps aux output
sudo ctr -n moby tasks ls
ps aux | grep nginx
sudo ctr -n moby c info <container-id>
# Stop the process via containerd
sudo ctr -n moby tasks kill <containerid-listed-in-ctr-output>
docker ps -a

# How to run custom commands in a container (we don't want the default command)
# Image and command - relationship
# Syntax: docker run <image> <command>
docker run nginx ls /
# 1. Run the command called "ls /"
# 2. The binary used to run the ls command is present in the image nginx
# Compare the above output to
ls /
# The first command docker run nginx ls / lists the container's filesystem. The process ls is running inside the container and it has visibility to only the container's filesystem.
# The second command ls / lists the host's filesystem
# The container exited right away because ls process terminated
docker ps -a

# The image nginx does not have all binaries of the host
docker run nginx ps
docker run nginx docker ps

# Remove all containers
docker rm -f $(docker ps -aq)

# Create webserver1 container again
docker run -d --name webserver1 -p 80:80 nginx
# Ensure container is running
docker ps
# exec to create another subsequent process in the running container
docker exec webserver1 ls /
# The binary of ls is from the container's filesystem itself
# Since we don't have ps or docker binary in this container, this will throw an error
docker exec webserver1 ps
docker exec webserver1 docker ps
# You cannot exec in a stopped container
docker stop webserver1
docker exec webserver1 ls /
# Start the PID 1 process and now exec works
docker start webserver1
docker exec webserver1 ls /

# Open a shell (exec into the container)
docker exec -it webserver1 /bin/bash
# Inside the container
ls /
ps
docker
cat /proc/1/cmdline
# There is already nginx running in this container, so we can't run another nginx process
nginx
echo Hello > /usr/share/nginx/html/index.html 
^D
curl localhost # Verify that the page has changed
# We can also see this from the host's browser
# Observe that the logs are not reset after container restart
docker stop webserver1
docker logs webserver1
docker start webserver1
docker logs webserver1
# Even index.html is the modified one
curl localhost

# But if we destroy the container and recreate it, all files are reset
docker rm -f webserver1
docker run -d --name webserver1 -p 80:80 nginx
curl localhost
docker logs webserver1

# Overlay Filesystem
https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/

# Difference between the initial state of the filesystem and current state
docker diff webserver1

# The 3 types of changes
docker exec -it webserver1 /bin/bash
# Inside the container
ls /usr/share/nginx/html/
echo Hello > /usr/share/nginx/html/index.html
rm /usr/share/nginx/html/50x.html
echo About > /usr/share/nginx/html/about.html
ls /usr/share/nginx/html/
^D
docker diff webserver1

# Commit the changes of webserver1 as a new image mywebserver:v1
docker commit webserver1 mywebserver:v1
docker images

# Create a container using the changes of webserver2
docker run -d --name webserver2 -p 8080:80 mywebserver:v1
# Note that the proxies have to listen on unique host ports
docker ps
curl localhost:8080
curl localhost:8080/about.html

# The changes made in webserver1 are visible in webserver2, but they don't appear in the diff of webserver2 since they are in the image
docker diff webserver2

# Dockerfile - Attempt 1
mkdir -p ~/data/mywebserver
cd ~/data/mywebserver
cat > Dockerfile <<DELIM
FROM nginx

RUN echo Hello > /usr/share/nginx/html/index.html
RUN rm /usr/share/nginx/html/50x.html
RUN echo About > /usr/share/nginx/html/about.html
DELIM
docker build -t mywebserver:v2 .
docker images
docker images -a
docker image history mywebserver:v2

# Dockerfile - Attempt 2
cat > Dockerfile <<DELIM
FROM nginx

RUN echo Hello > /usr/share/nginx/html/index.html \
 && rm /usr/share/nginx/html/50x.html \
 && echo About > /usr/share/nginx/html/about.html
DELIM

# Labels
docker image inspect nginx | grep -A2 Labels

# Viewing the environment variables set in the Dockerfile
# Environment variables that are common across all containers
docker run nginx env

# For environment variables that are different for each container, use docker run -e
docker run --help | grep env
docker run -e DB_HOST=dbserver nginx env

# Files copied are present here
docker run nginx ls /
docker run nginx ls /docker-entrypoint.d

# Use of -P and EXPOSE
docker run --name webserver3 -P -d nginx
docker run --name webserver4 -P -d nginx
docker run --name webserver5 -P -d nginx
docker ps

# STOPSIGNAL SIGQUIT
kill -l
docker stop webserver3
docker logs webserver3
# Observe how the container received SIGQUIT signal

# Remove all containers
docker rm -f $(docker ps -aq)

# Run a container without any command
docker run -d --name webserver1 nginx
docker inspect webserver1 | head -n10
# Check Path and Args
docker run -d --name webserver2 nginx ls /
docker inspect webserver2
# The Path is the same /docker-entrypoint.sh but Args is ls /

# An image (eg: ubuntu) may not have ENTRYPOINT and may have only CMD
https://github.com/tianon/docker-brew-ubuntu-core/blob/fbca80af7960ffcca085d509c20f53ced1697ade/jammy/Dockerfile

# In this case, CMD runs if we don't specify any command
docker run --name container1 ubuntu:22.04
docker inspect container1 | head -n10
# Path is bash and Args is []
docker run --name container2 ubuntu:22.04 ls /
docker inspect container2 | head -n10
# Path is ls and Args is ["/"]

# scratch image
https://hub.docker.com/_/scratch

# Docker Networking
# Building the ping image
mkdir -p ~/data/ping
cd ~/data/ping
cat > Dockerfile <<DELIM
FROM ubuntu:latest

RUN apt-get update && apt-get -y install iputils-ping net-tools
DELIM
docker build -t ping .
docker images

# Remove all existing containers
docker rm -f $(docker ps -aq)
docker ps -a

docker run -itd --name c1 ping
docker run -itd --name c2 ping
docker ps

docker inspect c1 | grep IPAddress
docker inspect c2 | grep IPAddress

# The docker0 interface
ip addr show docker0

ping 172.17.0.2

docker exec -it c1 /bin/bash
# Inside the container
ping 172.17.0.3
ping 172.17.0.1
ping 8.8.8.8
ping google.com
# By default, the container picks the DNS entries from the host
cat /etc/resolv.conf
ifconfig
ping c2
^D

# Docker - Custom Networks
docker network ls
docker network inspect bridge

# CIDR calculator
https://mxtoolbox.com/subnetcalculator.aspx

docker network create mynet1
docker network ls
ip addr show
docker network inspect mynet1
docker rm -f $(docker ps -aq)
docker run -itd --name c1 --net mynet1 ping
docker run -itd --name c2 --net mynet1 ping
docker ps
docker network inspect mynet1

ping 172.18.0.2

docker exec -it c1 /bin/bash
# Inside the container
ping 172.18.0.3
ping 172.18.0.1
ping 8.8.8.8
ping google.com
ping c2
cat /etc/resolv.conf
ifconfig
^D

# Activity
# Create another network mynet2 and create 2 containers c3 and c4 in it
# Verify that c3 and c4 can ping each other but cannot ping c1 and c2
# Now connect c3 to mynet1 as well using - "docker network connect mynet1 c3"
# Now verify which containers can communicate with which ones

# Kubernetes Installation

# Remove all docker containers
docker rm -f $(docker ps -aq)

# Kubernetes Learning Environment - Options
https://kubernetes.io/docs/tasks/tools/

# Installing a container runtime
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

# Install and configure containerd
https://docs.docker.com/engine/install/ubuntu/
# We already have containerd installed
ctr --version

# Create a default containerd configuration
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
# Restart containerd for the settings to take effect
sudo systemctl restart containerd
# Verify that containerd is running
sudo systemctl status containerd

https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
# Set the cgroup driver to systemd
sudo sed -i 's/SystemdCgroup.*/SystemdCgroup = true/g' /etc/containerd/config.toml
# Restart containerd for the settings to take effect
sudo systemctl restart containerd
# Verify that containerd is running
sudo systemctl status containerd

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# Apply sysctl params without reboot
sudo sysctl --system

# Verify Hardware requirements
# Ensure that the "Available" memory is close to 3000+
# Also ensure Swap is all 0
free -m
# To switch off swap
sudo swapoff -a
# Verify again that Swap numbers are 0
# You can disable it permanently by commenting the appropriate line in /etc/fstab
free -m
lscpu | grep "CPU(s):"
# Go to VM Settings -> System -> Process - increase the CPUs
# You have to shut down the VM before doing this
df -h | grep -P "/$"

# Install kubeadm, kubectl and kubelet
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
# Install GPG key
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
# Add k8s repository
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Install kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Creating a cluster with kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# You should see this message
# Your Kubernetes control-plane has initialized successfully!

# Control plane components
# The containers are created as containerd containers and they are created in the k8s.io namespace
sudo ctr ns ls
sudo ctr -n k8s.io c ls
# Another client to interact with CRI compatible runtimes
sudo crictl -r unix:///run/containerd/containerd.sock ps
# Setting the runtime so that we don't have to pass it everytime
echo "runtime-endpoint: unix:///run/containerd/containerd.sock" | sudo tee /etc/crictl.yaml
sudo crictl ps

# kubelet is running as a daemon
sudo systemctl status kubelet
# Use q to quit

# Set up the kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Get the status of the nodes
kubectl get nodes

# One thing in the context is the cluster info (IP and Port of the API server)
cat ~/.kube/config
# Who is listening on 6443
sudo ss -nltp

# To know why it is not ready, run a describe and look at the Conditions section in the output
kubectl describe nodes

# Install the Network plugin
kubectl apply -f \
        https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Check the status after about a minute and it should show as Ready
kubectl get nodes

# Although the node is ready, it's not schedulable
kubectl describe nodes | grep Taints

kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-
# If hostname doesn't match node name do this
node_name=`kubectl get nodes | grep control-plane | awk '{print $1}'`
kubectl taint nodes $node_name node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes $node_name node-role.kubernetes.io/control-plane:NoSchedule-

# You are ready when get nodes shows READY and Taints is None
kubectl get nodes
kubectl describe nodes | grep Taints

# Kubernetes Basics
kubectl get pods
kubectl run --image nginx webserver1
# Almost equivalent to: docker run -d --name webserver1 nginx
kubectl get pods
kubectl describe pod webserver1
curl <pod-ip> # Example: curl 10.244.0.4
kubectl logs webserver1
kubectl exec -it webserver1 -- /bin/bash
# Inside the pod
echo "Hello" > /usr/share/nginx/html/index.html
^D
curl <pod-ip> # Example: curl 10.244.0.4
# Equivalent of -p 8080:80
kubectl port-forward pod/webserver1 8080:80 --address 0.0.0.0
# This will run actively - now go to your browser and view the page at http://<ip>:8080/
# In VirtualBox port forward 8080 -> 8080, and access at http://localhost:8080/ in your host's browser
# Kill the port-forward using ^C once done
# Once killed, we can't connect to the service anymore

# There is no pull, build
# All these will throw errors
kubectl pull
kubectl images
kubectl build
# These commands are however there in the CRI runtime
sudo crictl images

# Relationship between Kubernetes and the underlying container runtime (containerd)
ps aux | grep nginx
ps faux
sudo ctr -n k8s.io c ls | grep nginx
sudo crictl ps | grep webserver1
# Make a note of the container id (first column)
sudo crictl exec -it <container-id> /bin/bash
# Inside the container
cat /usr/share/nginx/html/index.html
^D
sudo crictl rm -f <container-id>
# We see that kubernetes has recreated the container
kubectl get pods
curl <pod-ip>
# Note the time of creation is a few seconds ago
sudo crictl ps | grep webserver1
sudo ctr -n k8s.io c ls | grep nginx
# Delete pod nginx
kubectl delete pod webserver1
# The pod may be listed as Terminating for some time and after some time it will disappear from the get pods list
kubectl get pods
# All nginx related containers are removed from containerd
sudo ctr -n k8s.io c ls | grep nginx
sudo crictl ps | grep webserver1

https://onlineyamltools.com/convert-yaml-to-json

# YAML is compatible with JSON
{"name": "John"}

# Representing an object
"name": "John"
"age": 10

# Keys don't need to be in quotes
name: "John"
age: 10

# Values (if unambiguous) don't need to be in quotes
name: John
age: 10

# Boolean (also try with false)
name: John
age: 10
present: true

# null value
name: John
age: 10
present: null

# Arrays
name: John
age: 10
present: true
emails: ["john@example.com", "john@example.org"]

# Attempt 1 to simply arrays
name: John
age: 10
present: true
emails:
- "john@example.com"
- "john@example.org"

# Attempt 2 - remove quotes
name: John
age: 10
present: true
emails:
- john@example.com
- john@example.org

# Array of arrays
name: John
age: 10
present: true
emails:
- john@example.com
- john@example.org
values:
-
  - 10
  - 20
-
  - 30
  - 40

# Heterogenous array
name: John
age: 10
present: true
emails:
- john@example.com
- john@example.org
values:
-
  - 10
  - 20
-
  - 30
- 40

# Array of objects
name: John
age: 10
present: true
emails:
- value: john@example.com
- value: john@example.org

# Array of objects - example 2
name: John
age: 10
present: true
emails:
- type: Personal
  value: john@example.com
- type: Work
  value: john@example.org

# Object containing object
name:
  first: John
  last: Doe
age: 10
present: true
emails:
- type: Personal
  value: john@example.com
- type: Work
  value: john@example.org

# Multi-line strings
name:
  first: John
  last: Doe
description: |
  Hello
  World

name:
  first: John
  last: Doe
description: |-
  Hello
  World

# Use of > and >-
command: >
  apt-get install
  nginx
  curl

command: >-
  apt-get install
  nginx
  curl

# Document separation
name: John Doe
---
name: George

# Delete all resources
kubectl delete all --all

# Declarative spec for a pod
cd ~/data
cat > my-first-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: webserver1
spec:
  containers:
  - image: nginx
    name: nginx
DELIM
# Make sure you don't have a pod already
# Then run this spec
kubectl apply -f my-first-pod.yaml
# Note the word: "created" in the output
kubectl get pods
# Apply again - this time it says "unchanged"
kubectl apply -f my-first-pod.yaml
echo $?
kubectl describe pod webserver1 | grep IP
kubectl get pods -o wide
curl <pod-ip>

# Change the image and apply again
cat > my-first-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: webserver1
spec:
  containers:
  - image: nginx:alpine
    name: nginx
DELIM
# Before apply
kubectl describe pod webserver1 | grep -P "Image|Container ID"
sudo crictl images --digests | grep nginx
sudo crictl ps | grep webserver1
# RESTARTS is 0
kubectl get pods
# When we apply we will see it say pod/nginx configured
kubectl apply -f my-first-pod.yaml
# Observe RESTARTS has become 1 in this output
kubectl get pods
# We see 2 nginx images
sudo crictl images --digests | grep nginx
# Even the Container ID has changed
sudo crictl ps | grep webserver1
kubectl describe pod webserver1 | grep -P "Image|Container ID"

# Delete using spec
kubectl delete -f my-first-pod.yaml
kubectl get pods

# Multi-container pod patterns
https://betterprogramming.pub/understanding-kubernetes-multi-container-pod-patterns-577f74690aee

# Multiple containers - this doesn't work
cat > my-first-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: webserver1
spec:
  containers:
  - name: nginx1
    image: nginx:alpine
  - name: nginx2
    image: nginx:alpine
DELIM
kubectl apply -f my-first-pod.yaml
kubectl get pods
# The orchestrator does not give up
kubectl get pods -w
kubectl logs webserver1 -c nginx1
kubectl logs webserver1 -c nginx2
kubectl delete -f my-first-pod.yaml

# Multiple containers - something that works
cat > my-first-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: webserver1
spec:
  containers:
  - image: nginx
    name: nginx
  - image: curlimages/curl
    name: curl
    stdin: true
    tty: true
    command: ["/bin/sh"]
DELIM
# The curl container is almost like
# docker run --name curl -it curlimages/curl /bin/sh
kubectl apply -f my-first-pod.yaml
# Make a note of the IP of the pod
kubectl get pods -o wide
kubectl exec -it -c curl webserver1 -- /bin/sh
# Inside the container
ps aux
# But something is listening on port 80
netstat -nltp
# We see the same IP as the pod IP printed above
ifconfig
curl localhost
^D

https://jimmysong.io/en/blog/sidecar-injection-iptables-and-traffic-routing/

# Replicasets
cd ~/data
cat > my-first-rs.yaml <<DELIM
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
DELIM
kubectl delete all --all
kubectl apply -f my-first-rs.yaml
kubectl get replicasets
# OR kubectl get rs
kubectl get pods -o wide
# Make a note of the IP of any one of the replicas
curl <pod-ip>
# There are 3 sets of nginx processes in the machine now
ps faux

# If I apply again, k8s says "unchanged"
kubectl apply -f my-first-rs.yaml
# We see that k8s says replicaset.apps/nginx unchanged
# Change replicas to 6 and apply again
sed -i 's/replicas: 3/replicas: 6/g' my-first-rs.yaml
kubectl apply -f my-first-rs.yaml
kubectl get rs -w
kubectl get pods
# Change replicas to 2 and apply again
sed -i 's/replicas: 6/replicas: 2/g' my-first-rs.yaml
kubectl apply -f my-first-rs.yaml
kubectl get pods
# Replicaset controller did it
kubectl describe rs webserver

# What happens if we delete one of the pods
# Make a note of one of the pod names with the label app=nginx
kubectl get pods -l app=nginx
pod_name=`kubectl get pods -l app=nginx | grep webserver | head -n1 | awk '{print $1}'`
# Delete one of them
kubectl delete pod $pod_name
# Kubernetes will now create the pod again
kubectl get pods
# The replicaset controller creates it
kubectl describe rs nginx

#  Reusing the same label in a pod that was created by us (and not by a replicaset)
cat > my-first-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    app: nginx
spec:
  containers:
  - image: nginx
    name: nginx
DELIM
kubectl apply -f my-first-pod.yaml
# The pod nginx gets created but we see that it is immediately terminated
kubectl get pods -w
# Note how replicaset controller deleted the pod even though it did not create it
kubectl describe rs webserver

# Kubernetes Components
https://kubernetes.io/docs/concepts/overview/components/

# Feature branch workflow model
https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow

# Deployment
# Start afresh
kubectl delete all --all

# Let us create a deployment spec
cd ~/data
cat > my-first-deployment.yaml <<DELIM
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
DELIM
kubectl apply -f my-first-deployment.yaml
kubectl get deployments # OR kubectl get deploy
kubectl get rs
kubectl get pods

kubectl describe deploy nginx
# Role of Deployment controller - this is the one that creates the ReplicaSet

# Verify that the image is 1.20
kubectl describe deploy nginx | grep Image
kubectl describe pods | grep Image

# Change the image to 1.21
sed -i 's/1.20/1.21/g' my-first-deployment.yaml
kubectl apply -f my-first-deployment.yaml
kubectl get pods -w
kubectl describe pods | grep Image

# How Kubernetes manages deployments
# We see that there are 2 replicasets
kubectl get rs
# Describe the replicaset that is having 0 replicas as of now (this is the old one)
kubectl describe rs <old-rs> | grep Image
# Observe that the image is nginx:1.20
# The new replicaset has image nginx:1.21
kubectl describe rs <new-rs> | grep Image
# Describe the deployment to see the sequence of operations
kubectl describe deploy nginx

# Kubernetes Services
https://kubernetes.io/docs/concepts/services-networking/service/

# Services
cd ~/data
cat >> my-first-deployment.yaml <<DELIM
---     
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
DELIM
kubectl apply -f my-first-deployment.yaml
# Observe that k8s says deployment unchanged, but service created
kubectl get services # OR kubectl get svc
# Make a note of the ClusterIP of nginx
curl <cluster-ip>
# From which pod is this being served
# It is one of these
kubectl get endpoints # OR kubectl get ep
# The ep's are nothing but the IPs of the pods of nginx
kubectl get pods -o wide

# Where is the DNS server?
sudo crictl ps
kubectl get pods --all-namespaces
kubectl get deploy --all-namespaces
kubectl get svc --all-namespaces
# Note the IP address of the DNS service of Kubernetes (it is 10.96.0.10)
# What is the IP address of DNS in your VM?
cat /etc/resolv.conf
# We are not using k8s dns
# So this lookup fails
nslookup nginx
# Or the full name of the service
nslookup nginx.default.svc.cluster.local
nslookup nginx.default.svc.cluster.local 10.96.0.10

# Create a curl pod
cat > curl-pod.yaml <<DELIM
apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
  - name: curl
    image: curlimages/curl
    stdin: true
    tty: true
    command: ["/bin/sh"]
DELIM
kubectl apply -f curl-pod.yaml
kubectl get pods
kubectl exec -it curl -- /bin/sh
# Inside the container
# Cluster IP is resolvable in the Pod
curl <nginx-cluster-ip>
# The DNS server in the container is the Kubernetes DNS server
cat /etc/resolv.conf
nslookup nginx
curl nginx
^D

# Calico
https://tanzu.vmware.com/developer/guides/container-networking-calico-refarch/

# Set service's type to LoadBalancer in my-first-deployment.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

# Check the service before apply
kubectl get svc
kubectl apply -f my-first-deployment.yaml
kubectl get svc

# Observe that External IP says pending

# Adding (hacking) an External IP

  externalIPs:
  - <your-vm-ip>

# Doing something similar in Docker
https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-18-04

# Check the service before apply
kubectl get svc
kubectl apply -f my-first-deployment.yaml
kubectl get svc

sudo ss -nltp | grep 80
sudo ss -nltp | grep "kube-proxy"

curl <public-vm-ip>
curl <private-vm-ip>:<node-port>
curl <cluster-ip>
curl <pod-ip>

# Ingress Controller
https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/
https://github.com/nginxinc/kubernetes-ingress/blob/main/deployments/deployment/nginx-ingress.yaml
https://github.com/nginxinc/kubernetes-ingress/blob/main/deployments/service/loadbalancer.yaml
https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/basic-configuration/

https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/

https://kubernetes.io/docs/concepts/services-networking/ingress/

https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/

# End to end application
https://github.com/buzypi/users-service
https://github.com/buzypi/payments-service

# Choices for Microservice implementation
https://www.solo.io/blog/announcing-gloo-the-function-gateway/

# The Art of Unix Programming
http://www.catb.org/~esr/writings/taoup/html/

# Serverless
https://knative.dev/docs/
https://aws.amazon.com/serverless/sam/

# Docker Metrics
docker stats

# Kubernetes Metrics server
https://github.com/kubernetes-sigs/metrics-server

# DaemonSet and Fluentd
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

# Docker images for Elastic products
https://www.docker.elastic.co/

# Open Tracing
https://opentracing.io/

# Prometheus SD Configs
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config

# Node exporter
https://github.com/prometheus/node_exporter

# Resources
https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/docs/vmware-kubernetes-up-running-dive-into-the-future-of-infrastructure.pdf
https://www.packtpub.com/free-learning
https://www.katacoda.com/
https://thenewstack.io/
https://thenewstack.io/ebooks/
https://www.thoughtworks.com/radar
https://radar.cncf.io/
