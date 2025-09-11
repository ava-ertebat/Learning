# How to Set Ubuntu apt repository in nexus oss

Step 1:open the sources.list
```
sudo nano /etc/apt/sources.list
```
Step 2:set this address
```
deb [trusted=yes] https://registry.nichosting.ir/repository/apt_ubuntu_local/ jammy main
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy_main_restricted jammy main restricted
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-updates_main_restricted jammy-updates main restricted
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy_universe jammy universe
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-updates_universe jammy-updates universe
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy_multiverse jammy multiverse
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-updates_multiverse jammy-updates multiverse
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-backports_main_restricted_universe_multiverse jammy-backports main restricted universe multiverse
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-security_main_restricted jammy-security main restricted
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-security_universe jammy-security universe
deb [trusted=yes] https://registry.nichosting.ir/repository/ubuntu_jammy-security_multiverse jammy-security multiverse
deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://registry.nichosting.ir/repository/apt_ubuntu_docker_proxy jammy stable
```

Step 3:
get Certificate With This Command
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
And This Command
```
curl -fsSLo kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Step 4:
Use This Curl Command
```
curl -fsSLo kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
sudo mv kubernetes-archive-keyring.gpg /usr/share/keyrings/
sudo chmod a+r /usr/share/keyrings/kubernetes-archive-keyring.gpg
```

Step 5:
use apt update
```
sudo apt update
```

Step 6: 
## enjoy
