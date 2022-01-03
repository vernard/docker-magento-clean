This repository is used to create a fresh Magento 2.4.1-p1 installation. This repo is based off from [Mark Shust](https://github.com/markshust/docker-magento)'s work. If you're interested how it's all set up, check out his repo. He also has a [free video guide](https://m.academy/courses/set-up-magento-2-development-environment-docker/) for it.

# Requirements
You must have [docker](https://docker.com/) and [docker-compose](https://docs.docker.com/compose/install/) command installed to use this repository.

It is highly recommended to use a unix-based system (Ubuntu, MacOS, etc.) because the setup is much easier, faster, and debugging code is also easier. 

Invest time in setting up an [Ubuntu](https://ubuntu.com/) machine or you can set your computer to dual-boot so you'll use Linux when coding.

### For Windows users:
1. Open CMD as administrator (Press Win key, type CMD, Run as administrator)

2. Execute `wsl --install`
> If you already have WSL installed, execute `wsl --install -d Ubuntu` instead to install Ubuntu alongside currently installed WSLs.

3. Reboot PC

4. Upon Reboot, you'll find that "Ubuntu" will run and you'll see this message:
 "Installing, this may take a few minutes..."
 
5. After that's done, it will ask you for a UNIX username and password

6. Run "sudo apt update" on Ubuntu

7. Download and install [Docker Desktop on Windows](https://docs.docker.com/desktop/windows/install/)
    > Make sure the "Install required Windows component for WSL2" is checked
 
8. Reboot PC

9. Open Docker Desktop and accept terms if running for first time

10. (optional) Go ahead with the tutorial if you want to learn Docker. Skip if you're already familiar.

11. Double check that `docker` and `docker-compose` commands are installed properly. Run these commands:
    ```
    docker -v # This should return Docker's current version
    docker-compose -v # This should return Docker Compose current version
    ```
12. Add your SSH keys in `~/.ssh` if you already have one. The `/mnt/c/` directory in Ubuntu is the C:/ partition in your Windows machine. The same applies if you have other partitions like D: 
    ```
    mkdir ~/.ssh
    cp /mnt/c/Users/YOUR_USER/.ssh/id_rsa ~/.ssh/id_rsa # Replace YOUR_USER with your actual Windows user
    cp /mnt/c/Users/YOUR_USER/.ssh/id_rsa.pub ~/.ssh/id_rsa.pub # Replace YOUR_USER with your actual Windows user
    chmod 600 ~/.ssh/id_rsa
    ```
    
    If you don't have an SSH key yet, you can generate one by running 
    ```
    ssh-keygen
    ```
    
    After creating your key, make sure to add your public key to your [Github account](https://github.com/settings/keys). 
13. Follow the "How to install" instructions below, but make sure your INSTALLATION_FOLDER is under /mnt/c or /mnt/d (That's C: or D: in your drive) so you can open them in your code editor.


### For Linux users:

Follow the Docker installation instructions here: https://docs.docker.com/engine/install/ubuntu/
1. Set up the repository
    ```
    sudo apt-get update
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
2. Install Docker Engine
    ```
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

3. Verify that docker is installed
    ```
    docker -v
    ```

4. Install Docker Compose by following these instructions: https://docs.docker.com/compose/install/
    ```
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose --version # Check if it's installed
    ```

### For Mac Users

I haven't tried this so there's no documentation (feel free to add your Docker & Docker compose installation here)

Here are the relevant links for installing Docker & Docker Compose on Mac.
* https://docs.docker.com/desktop/mac/install/
* https://docs.docker.com/compose/install/

# How to install
> Do not move forward if `docker -v` and `docker-compose -v` doesn't work.
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
# For Linux or Mac
cp docker-compose.dev-linux.yml docker-compose.dev.yml

# For Windows
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

# If you're on Windows, you must run the commands below to copy Magento code inside the container
# 1. Copy the zipped Magento code into the container
docker-compose cp magento-2.4.3-p1.tar.gz phpfpm:/var/www/html
# 2. Go into container's CLI
bin/bash
# 3. Extract the magento code
tar -zxf magento-2.4.3-p1.tar.gz
# 4. Extracted files are under src/, we want to move them to ~/html
#    Some folder/files may not be copied, that's okay.
mv src/* .
# 5. Remove the remaining src/ folder
rm -rf src/
# 6. Remove the zipped Magento code
rm magento-2.4.3-p1.tar.gz
# 7. Go out of container's CLI
exit
# 8. Copy app/ folder in host to container
docker-compose cp src/app/ phpfpm:/var/www/html/
```

#### Step 5: Restore from database backup and run `setup:upgrade` command
```
cd db/
tar -zxf db.sql.tar.gz
cd ..

bin/mysql < db/db.sql # You can open phpMyAdmin at http://127.0.0.1:8080 after running this
bin/magento setup:upgrade

bin/setup-domain magento.test

# If you're on Windows, you'll have to manually add that domain in your hosts file
# 1. Open Notepad as an administrator (Press Win key, type 'Notepad', right click it, Run as administrator)
# 2. Open C:\Windows\System32\drivers\etc\hosts file
# 3. Add "127.0.0.1 magento.test" at the end as a new line and save it (it won't save if you didn't open Notepad as an administrator)
```

And you're done!

Visit [https://magento.test/admin](https://magento.test/admin) and use the following credentials.

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

If you're getting 403 Forbidden error when opening `https://magento.test` then you should check the `nginx.conf` file inside the container. 

If it's an empty file, copy the `nginx.conf.sample` to `nginx.conf` by running this command:

```
bin/bash # Go inside the container
cp nginx.conf.sample nginx.conf
exit # Exit CLI in the container

docker-compose exec app nginx -s reload # Reload the nginx service to apply your changes
```

### Clearing Cache

In case you want to clear cache but can't run `bin/magento cache:clear`, you can clear cache directly in Redis container.
```
bin/redis redis-cli FLUSHALL # Should return "OK"
```

