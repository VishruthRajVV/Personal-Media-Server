# Personal Media Server
A complete guide to building your own personal self hosted server for streaming and ad-blocking for all your devices.

You don't need powerful hardware to set this up. I use an old core 2 duo computer. Raspberry pi would be preferred to make your life easier.

I am using Ubuntu Desktop in this guide. You can use any Linux Distribution, the steps are not that different.

You can Download ubuntu desktop from https://ubuntu.com/download/desktop or ubuntu server from https://ubuntu.com/download/server. Create a bootable USB drive using rufus. Plug the usb on your computer, and select the usb drive from the boot menu and install ubuntu server. Follow the steps to install and configure ubuntu, but ensure you do not install docker during the setup. Ubuntu Snap uses an often outdated version of docker that we want to avoid.

Also make sure "Install OpenSSH server" is checked or you will not be able to access the machine remotely.
Once installation finishes you can now reboot and connect to your machine remotely using ssh.

    ssh username@server-ip 
    // username you selected during installation
    // Type ip a to find out the ip address of your server. Will be present against device like **enp4s0** prefixed with 192.168.
  
#Create directories for Movies, Music , TV and Anime.

I have kept all my media in ~/server/media. 

We will be using hardlinks (https://trash-guides.info/Hardlinks/Hardlinks-and-Instant-Moves/) so once the torrents are downloaded they are linked to media directory as well as torrents directory without using double storage space.

    # Creating the directories
    mkdir ~/server
    mkdir ~/server/media # Media directory
    mkdir ~/server/torrents # Torrents

    # Creating the directories for torrents
    cd ~/server/torrents
    mkdir Incomplete  Movies  Music  TV Anime 

    cd ~/server/media
    mkdir  Movies  Music  TV Anime 

# Installing docker and docker-compose

Once you've installed, rebooted, and logged in you should be left with a fresh install of Linux. We're going to keep it that way by only installing two things: docker and docker-compose.

# Docker (https://docs.docker.com/engine/install/ubuntu/)

    # install packages to allow apt to use a repository over HTTPS
    sudo apt-get update
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    # Add Docker’s official GPG key:
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    # Setup the repository
    echo \
      "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    # Install Docker Engine
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    # Add user to the docker group to run docker commands without requiring root
    sudo usermod -aG docker $(whoami) 



  
