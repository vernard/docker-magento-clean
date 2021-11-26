This repository is used to create a fresh Magento 2.4.1-p1 installation.

# Requirements
You must have [docker](https://docker.com/) and [docker-compose](https://docs.docker.com/compose/install/) command installed to use this repository.

It is highly recommended to use a unix-based system (Ubuntu, MacOS, Debian, etc.) because the setup is much easier, faster, and debugging code is also easier. 

Invest time in setting up an [Ubuntu](https://ubuntu.com/) machine or you can set your computer to dual-boot so you'll use Linux when coding.

For Windows users:
* [Docker Desktop](https://docs.docker.com/desktop/windows/install/)
* [WSL](https://docs.docker.com/desktop/windows/wsl/) 
* [Debian](https://www.microsoft.com/en-us/p/debian/9msvkqc78pk6)

For Mac users:
* [Docker Desktop](https://docs.docker.com/desktop/windows/install/)


Meaning, if you're on Windows OS, you'll need to [install  WSL](https://docs.docker.com/desktop/windows/wsl/) (Windows Subsystem for Linux) and a unix-based command line like [Debian](https://www.microsoft.com/en-us/p/debian/9msvkqc78pk6#activetab=pivot:overviewtab).


> Do not move forward if `docker -v` and `docker-compose -v` doesn't work.

# How to install

> These commands will only work for unix-based command line like Debian. Running these command via Windows CLI won't work.

#### Step 1: Clone this repository to your local machine
```
# Replace $INSTALLATION_FOLDER with an actual directory in your machine
mkdir $INSTALLATION_FOLDER
cd $INSTALLATION_FOLDER
git clone git@github.com:vernard/docker-magento-clean.git .
```

#### Step 2: Setup `docker-compose.dev.yml` and `docker-compose.override.yml` files
```
# For Linux OS
cp docker-compose.dev-linux.yml docker-compose.dev.yml

# For Windows/Mac
cp docker-compose.dev-windows.yml docker-compose.dev.yml

cp docker-compose.dev.yml docker-compose.override.yml
```

#### Step 3: Extract the clean magento installation

Download the [magento files here](https://drive.google.com/drive/folders/1tK71vugVIWn41GUqY1VcKBt-7M0vdwMB?usp=sharing) and place them in the installation folder.

```
# This will create the `src` folder which will contain the Magento code.
tar -zxf magento-2.4.3-p1.tar.gz
``` 

#### Step 4: Fire up the containers
Before running this, make sure no other docker containers are running, or else you will encounter port forwarding issues because they're already taken.

If you have other application (like XAMPP, WAMP, etc.) that uses port 80, 443, 3306 like Apache/Nginx, MySQL services -- please turn those off before continuing. 
```
bin/start 
```

#### Step 5: Restore from database backup and run `setup:upgrade` command
```
cd db/
tar -zxf db.sql.tar.gz
cd ..

bin/mysql < db/db.sql
bin/magento setup:upgrade

bin/setup-domain magento.test
```

And you're done!

Visit `https://magento.test/admin` and use the following credentials.

Username: john.smith

Password: password123

# Troubleshooting

### Permission Error

```
PermissionError: [Errno 13] Permission denied:
```
If you're getting this error, your user may not have access to `/var/run/docker.sock` file.

To fix this, change the permission of the `docker.sock` file.
```$xslt
sudo chown $(whoami):$(whoami) /var/run/docker.sock
```
Another, more permanent solution for your dev environment, is to modify the user ownership of the unix socket creation. This will give your user the ownership, so it'll stick between restarts:

```
sudo nano /etc/systemd/system/sockets.target.wants/docker.socket
```
Change the file content to this:
```
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=YOUR_USERNAME_HERE # Edit this line to what your linux user is
SocketGroup=docker

[Install]
WantedBy=sockets.target
```

### 403 Forbidden

