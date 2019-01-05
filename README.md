# 404notfoundhard_infra
404notfoundhard Infra repository

bastion_IP=35.207.140.251
someinternalhost_IP=10.156.0.3

### one line connect
```
ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@10.156.0.3 -o "proxycommand ssh -W %h:%p -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.2"
```
#
additional task
#

#### first case
```
alias 2bastion='ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.251'
alias 2interalgcp='ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@10.156.0.3 -o "proxycommand ssh -W %h:%p -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.2"'

funcsave 2bastion
funcsave 2internalgcp
```
#### second case
```
Host 35.207.140.251
  Hostname 35.207.140.251
  User gcp_appUser
  IdentityFile  ~/.ssh/gcp_appUser.pub
Host 10.156.0.*
  IdentityFile  ~/.ssh/gcp_appUser.pub
  User gcp_appUser
  ProxyCommand ssh -W %h:%p  gcp_appUser@35.207.140.251
```
## gcloud: create firewall rule
```
gcloud compute firewall-rules create allow-puma-service \
                                                                       --allow tcp:9292 \
                                                                       --description='allow connect to puma service' \
                                                                       --direction INGRESS \
                                                                       --target-tags=puma-server
```
## gcloud: create instance with startup script
#### first case with content
```
gcloud compute instances create reddit-app\
                                                   --boot-disk-size=10GB \
                                                   --image-family ubuntu-1604-lts \
                                                   --image-project=ubuntu-os-cloud \
                                                   --machine-type=g1-small \
                                                   --tags puma-server \
                                                   --restart-on-failure \
                                                   --metadata startup-script='#!/bin/bash
                                                 apt update
                                                 apt install -y ruby-full ruby-bundler build-essential
                                                 sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
                                                 sudo bash -c \'echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" > /etc/apt/sources.list.d/mongodb-org-3.2.list\'
                                                 apt update
                                                 apt install -y mongodb-org
                                                 systemctl start mongodb.service
                                                 systemctl enable mongodb.service
                                                 git clone -b monolith https://github.com/express42/reddit.git
                                                 cd reddit/
                                                 bundle install
                                                 puma -d'
```
#### second case with remote script
```
gcloud compute instances create reddit-app\
                                               --boot-disk-size=10GB \
                                               --image-family ubuntu-1604-lts \
                                               --image-project=ubuntu-os-cloud \
                                               --machine-type=g1-small \
                                               --tags puma-server \
                                               --restart-on-failure \
                                               --scopes storage-ro \
                                               --metadata startup-script-url=https://gist.githubusercontent.com/404notfoundhard/426db8a75e0a7fd8f4b37d1a8d57291a/raw/478068f6dbd73c3d04acbb248fe21e948d2ee1ef/startup_script.sh
```
#### third case with local script
```
gcloud compute instances create reddit-app\
                                               --boot-disk-size=10GB \
                                               --image-family ubuntu-1604-lts \
                                               --image-project=ubuntu-os-cloud \
                                               --machine-type=g1-small \
                                               --tags puma-server \
                                               --restart-on-failure \
                                               --scopes storage-ro \
                                               --metadata-from-file startup-script=/path/to/my/startup_script.sh
```
