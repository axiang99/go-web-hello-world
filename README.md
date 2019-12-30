# go-web-hello-world

Demo project by Weixiang

## Task 0: Install a ubuntu 16.04 server 64-bit
Skip. 
Use ce-tu170, node-10-210-149-171 in which ubuntu 16.04 has been installed.

## Task 1: Update system
**Main Steps:**
```
uname -msr
 > Linux 4.4.0-131-generic x86_64
sudo apt update
sudo apt upgrade -y
sudo reboot
sudo apt list --upgradeable
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.2/linux-headers-4.11.2-041102_4.11.2-041102.201705201036_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.2/linux-headers-4.11.2-041102-generic_4.11.2-041102.201705201036_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.2/linux-image-4.11.2-041102-generic_4.11.2-041102.201705201036_amd64.deb
dpkg -i *.deb
sudo update-grub
sudo reboot
uname -msr
 > Linux 4.11.2-041102-generic x86_64
```

**Reference:**

https://www.howtoforge.com/tutorial/how-to-upgrade-linux-kernel-in-ubuntu-1604-server/


## Task 2: install gitlab-ce version in the host
**Main Steps:**
```
sudo apt-get install -y curl openssh-server ca-certificates
sudo apt-get install -y postfix
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.cetu170.com" apt-get install gitlab-ce 
Fix startup issue
http://10.210.149.171/ reset password
```
<font color='red'>**Issues:**</font>


* timeout: down: node-exporter: 0s, normally up, want up  

    **Analysis:**
    ```
    sudo gitlab-ctl tail
      > shows errors like "9100 port is hold"
    sudo netstat -anp | grep 9100
      tcp6       0      0 :::9100                 :::*                    LISTEN      1438/node_exporter
      tcp6       0      0 10.210.149.171:9100     10.210.152.91:40210     ESTABLISHED 1438/node_exporter
    ```
    10.210.152.91 is for central monitoring.
    
    **Fix**: 
    Disable node_exporter in Demo Gitlab then restart Gitlab.
    ```
    sudo vi /etc/gitlab/gitlab.rb
       ==> node_exporter['enable'] = false
    sudo gitlab-ctl reconfigure
    ```
       
**Reference:**

https://about.gitlab.com/install/#ubuntu?version=ce

## Task 3&4: create a demo group/project in gitlab & build the app and expose ($ go run) the service to 8081 port


**Main Steps:**

**In Gitlab:**
```
New group: demo
New Project: go-web-hello-world
```

**On node-10-210-149-171:**

Install go:
```
sudo apt install golang-go
```
Build APP:
```
mkdir -p ~/go/src
git clone http://10.210.149.171/demo/go-web-hello-world.git
cd go-web-hello-world/
vi main.go
copy code from https://gowebexamples.com/hello-world/, update "Fprintf" line and "ListenAndServe" to 8081
go build
Run: ./go-web-hello-world
```
Commit code:
```
git add main.go
git config --global user.email "weixiang.guo@ericsson.com"
git config --global user.name "Weixiang Guo" 
git commit
git push

```
**Output:**

http://10.210.149.171:8081 gets 'Go Web Hello World!'

**Reference:**

https://golang.org/
https://gowebexamples.com/hello-world/

## Task 5: install docker
**Main Steps:**
Docker is already the newest version, Anyway it does not harm to go through the steps in the guide:
```
docker version | grep Ver
    > 19.03.5
sudo apt-get remove docker docker-engine docker.io containerd runc 
    Got: Package 'xxx' is not installed, so not removed
    Reason:
    It’s OK if apt-get reports that none of these packages are installed.
    The contents of /var/lib/docker/, including images, containers, volumes, and networks, are preserved. The Docker Engine - Community package is now called docker-ce.

apt-get remove docker docker-engine docker.io containerd runc

apt-get update
apt-get install     apt-transport-https     ca-certificates     curl     gnupg-agent     software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) \
 stable"

apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
    Got: xxx is already the newest version xxx
docker version
```

**Reference:**
https://docs.docker.com/install/linux/docker-ce/ubuntu/

## Task 6: run the app in container

**Main Steps:**
```
vi Dockerfile 
docker build -t go-web-hello-world .
docker images
docker run -p 8082:8081 -it --rm --name run-web-hello-world go-web-hello-world
fix issue 'Error starting userland proxy: listen tcp 0.0.0.0:8082: bind: address already in use.' then rerun
In browser:  http://10.210.149.171:8082/  shows 'Go Web Hello World!'
```
**Isssues:**
* Error starting userland proxy: listen tcp 0.0.0.0:8082: bind: address already in use.

    **Analysis:**
    ```
    $ sudo netstat -anp | grep 8082
    tcp        0      0 127.0.0.1:8082          0.0.0.0:*               LISTEN      16105/sidekiq 5.2.7
    tcp        0      0 127.0.0.1:8082          127.0.0.1:34076         ESTABLISHED 16105/sidekiq 5.2.7
    tcp        0      0 127.0.0.1:34076         127.0.0.1:8082          ESTABLISHED 16075/prometheus
    ```

    **Fix**: 
    Change sidekiq listen_port then restart Gitlab.
    ```
    sudo vi /etc/gitlab/gitlab.rb
       ==> sidekiq['listen_port'] = 8084    //sudo netstat -anp |grep 8084 shows 8084 is not in use so change it to 8084
    sudo gitlab-ctl reconfigure
    ```

**Reference:**

https://docs.docker.com/engine/reference/commandline/build/

## Task 7: push image to dockerhub

**Main Steps:**
```
docker tag go-web-hello-world:latest axiang99/go-web-hello-world:v0.1
docker login 
docker push  axiang99/go-web-hello-world:v0.1

```
**Output:**

Repo: https://hub.docker.com/repository/docker/axiang99/go-web-hello-world

**Reference:**

https://docs.docker.com/engine/reference/commandline/build/


## Task 8: document the procedure of step 0-7 in a MarkDown file

Do it in Gitlab GUI using Edit.

## Task 9: install a single node Kubernetes cluster using kubeadm
**Main Steps:**
```
- Installing kubeadm, kubelet and kubectl

sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

- Initializing control-plane node

sudo su
kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
              
kubectl get node
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
kubectl get node
kubectl get pods --all-namespaces  
kubectl taint nodes --all node-role.kubernetes.io/master-

git add  admin.conf
git commit
git push

```
**Reference:**

https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

## Task 10: deploy the hello world container and expose the service to nodePort 31080
**Main Steps:**
```
kubectl run go-web-hello-world --image axiang99/go-web-hello-world:v0.1  --replicas=1

kubectl expose deployment go-web-hello-world --type=NodePort --port=31080 --target-port=8081
Or
kubectl expose deployment go-web-hello-world --type=NodePort --port=8081     // --target-port == port by default

kubectl edit svc go-web-hello-world
   - nodePort: 31080  //replace the random nodePort with 31080
```
**Output:**

http://10.210.149.171:31080/ shows 'Go Web Hello World!''


## Task 11: install kubernetes dashboard and expose the service to nodeport 31081
**Main Steps:**
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
kubectl get svc -n kubernetes-dashboard
kubectl edit svc kubernetes-dashboard  -n kubernetes-dashboard    
    spec:
      clusterIP: 10.96.166.28
      ports:
      - port: 443
        protocol: TCP
        targetPort: 8443
        NodePort: 31081   => Insert this line
      selector:
        k8s-app: kubernetes-dashboard
      sessionAffinity: None
      type: nodePort    => relace ClusterIP with nodePort
```
**Issue:**
* https://10.210.149.171:31081 (Privacy error - NET::ERR_CERT_INVALID)
    ** Fix **
    Create a new certificate to replace the original one. Guide: https://github.com/kubernetes/dashboard/issues/2954
```
    kubectl get secret -n kubernetes-dashboard
    kubectl get secret/kubernetes-dashboard-certs -n kubernetes-dashboard
    
    mkdir certs
    openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
    openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
    
    kubectl delete secret/kubernetes-dashboard-certs -n kubernetes-dashboard
    kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
    
    kubectl get po -n kubernetes-dashboard
    kubectl delete po/kubernetes-dashboard-5996555fd8-h8jvm  -n kubernetes-dashboard
    kubectl get po -n kubernetes-dashboard
```
**Output:**

https://10.210.149.171:31081 from PC asks for kubeconfig or token.

**Reference:**

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Task 12: generate token for dashboard login in task 11
**Main Steps:**
```
Copy the yaml file in the guide ans save to files sa.yml and rb.yml
kubectl apply -f sa.yml
kubectl apply -f rb.yml
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Paste the string after "token:" into the "Enter token" textbox on page above
```        
**Output:**

See dashboards charts.

**Reference:**

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

## Task 13: publish your work
Repo: https://github.com/axiang99/go-web-hello-world
