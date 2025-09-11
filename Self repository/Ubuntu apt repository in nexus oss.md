How to Set Ubuntu apt repository in nexus oss

1:open the sources.list
sudo nano /etc/apt/sources.list
==============================================================================================================================================================================
2:set this address
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

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://ir.archive.ubuntu.com/ubuntu/ jammy main restricted
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://ir.archive.ubuntu.com/ubuntu/ jammy-updates main restricted
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://ir.archive.ubuntu.com/ubuntu/ jammy universe
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy universe
deb http://ir.archive.ubuntu.com/ubuntu/ jammy-updates universe
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://ir.archive.ubuntu.com/ubuntu/ jammy multiverse
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy multiverse
deb http://ir.archive.ubuntu.com/ubuntu/ jammy-updates multiverse
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://ir.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src http://ir.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted
deb http://security.ubuntu.com/ubuntu/ jammy-security universe
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security universe
deb http://security.ubuntu.com/ubuntu/ jammy-security multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security multiverse
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy main restricted
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates main restricted
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy universe
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy universe
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates universe
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy multiverse
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy multiverse
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates multiverse
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src http://nova.clouds.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu jammy-security main restricted
# deb-src http://security.ubuntu.com/ubuntu jammy-security main restricted
deb http://security.ubuntu.com/ubuntu jammy-security universe
# deb-src http://security.ubuntu.com/ubuntu jammy-security universe
deb http://security.ubuntu.com/ubuntu jammy-security multiverse
# deb-src http://security.ubuntu.com/ubuntu jammy-security multiverse



==============================================================================================================================================================================
3:get Certificate
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
curl -fsSLo kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

4:
curl -fsSLo kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
sudo mv kubernetes-archive-keyring.gpg /usr/share/keyrings/
sudo chmod a+r /usr/share/keyrings/kubernetes-archive-keyring.gpg


=======================================================================================

5:use apt update

sudo apt update
=======================================================================================
6:enjoy
=======================================================================================
