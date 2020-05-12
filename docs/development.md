# Development Server Deployment

## Installing kubernetes for a development installation (on a single virtual machine)
============================================================================

You need python2, some python dependencies, a specific version of ansible, and gnu make. Then, you need to download specific ansible roles using ansible-galaxy, and binaries kubectl and helm.

### Option 1) How to install the necessary components locally when using Debian or Ubuntu as your operating system

First, we’re going to install Poetry. We’ll be using it to run ansible playbooks, and manage their python dependencies. See also the poetry documentation.

This assumes you’re using python 2.7 (if you only have python3 available, you may need to find some workarounds):

```
sudo apt update
sudo apt install -y python2.7 python-pip
curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py > get-poetry.py
python2.7 get-poetry.py --yes
source $HOME/.poetry/env
ln -s /usr/bin/python2.7 $HOME/.poetry/bin/python
```

Install the python dependencies to run ansible:

```
git clone https://github.com/social-network/society-server-deployer.git
cd society-server-deployer/ansible
## (optional) if you need ca certificates other than the default ones:
# export CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
poetry install
```

Download the ansible roles necessary to install databases and kubernetes:

```
make download
```

### Option 2) How to use docker on the local host with a docker image that contains all the dependencies

On your machine you need to have the docker binary available. See how to install docker. Then:

```
docker pull quay.io/society/networkless-admin

# cd to a fresh, empty directory and create some sub directories
cd ...  # you pick a good location!
mkdir ./admin_work_dir ./dot_kube ./dot_ssh && cd ./admin_work_dir
# copy ssh key (the easy way, if you want to use your main ssh key pair)
cp ~/.ssh/id_rsa ../dot_ssh/
# alternatively: create a key pair exclusively for this installation
ssh-keygen -t ed25519 -a 100 -f ../dot_ssh/id_ed25519
ssh-add ../dot_ssh/id_ed25519
# make sure the server accepts your ssh key for user root
ssh-copy-id -i ../dot_ssh/id_ed25519.pub root@<server>

docker run -it --network=host -v $(pwd):/mnt -v $(pwd)/../dot_ssh:/root/.ssh -v $(pwd)/../dot_kube:/root/.kube quay.io/society/networkless-admin
# inside the container, copy everything to the mounted host file system:
cp -a /src/* /mnt
# and make sure the git repos are up to date:
cd /mnt/society-server && git pull
cd /mnt/society-server-deployer && git pull
cd /mnt/society-server-deployer-networkless && git pull
```

(The name of the docker image contains networkless because it was originally constructed for high-security installations without connection to the public internet. Since then it has grown to be our recommended general-purpose installation platform.)

Now exit the docker container. On subsequent times:

```
cd admin_work_dir
docker run -it --network=host -v $(pwd):/mnt -v $(pwd)/../dot_ssh:/root/.ssh -v $(pwd)/../dot_kube:/root/.kube quay.io/society/networkless-admin
cd society-server-deployer/ansible
# do work.
```

Any changes inside the container under the mount-points listed in the above command will persist (albeit as user root), everything else will not, so be careful when creating other files.

To connect to a running container for a second shell:

```
docker exec -it `docker ps -q --filter="ancestor=quay.io/society/networkless-admin"` /bin/bash
```

## How to set up your hosts.ini file
-------------------------------------

Assuming a single virtual machine with a public IP address running Ubuntu 16.04 or 18.04, with at least 4 CPU cores and at least 8 GB of memory.

From `server-deploy/ansible`:

```
cp hosts.example-demo.ini hosts.ini
```

Open hosts.ini and replace `X.X.X.X` with the IP address of your virtual machine that you use for ssh access.  You can try using ``sed -i 's/X.X.X.X/1.2.3.4/g' hosts.ini``.

## How to install kubernetes
--------------------------

From `server-deploy/ansible`:

```
poetry run ansible-playbook -i hosts.ini kubernetes.yml -vv
```

When the playbook finishes correctly (which can take up to 20 minutes), you should have a folder ``artifacts`` containing a file ``admin.conf``. Copy this file:

```
mkdir -p ~/.kube
cp artifacts/admin.conf ~/.kube/config
```

Make sure you can reach the server:

  `kubectl version`

should give output similar to this:

```
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.2", GitCommit:"66049e3b21efe110454d67df4fa62b08ea79a19b", GitTreeState:"clean", BuildDate:"2019-05-16T16:23:09Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.2", GitCommit:"66049e3b21efe110454d67df4fa62b08ea79a19b", GitTreeState:"clean", BuildDate:"2019-05-16T16:14:56Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```
