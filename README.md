# Personal Media Server
A complete guide to building your own personal self hosted server for streaming and ad-blocking (Bonus) for all your devices.

You don't need powerful hardware to set this up. I use an old core 2 duo computer. Raspberry pi would be preferred to make your life easier.

I am using Ubuntu Desktop in this guide. You can use any Linux Distribution, the steps are not that different.

You can Download ubuntu desktop from https://ubuntu.com/download/desktop or ubuntu server from https://ubuntu.com/download/server. Create a bootable USB drive using rufus. Plug the usb on your computer, and select the usb drive from the boot menu and install ubuntu server. Follow the steps to install and configure ubuntu, but ensure you do not install docker during the setup. Ubuntu Snap uses an often outdated version of docker that we want to avoid.

Also make sure "Install OpenSSH server" is checked or you will not be able to access the machine remotely.
Once installation finishes you can now reboot and connect to your machine remotely using ssh.

    ssh username@server-ip 
    # username you selected during installation
    # Type ip a to find out the ip address of your server. Will be present against device like **enp4s0** prefixed with 192.168.
  
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
Tip: You can either type exit or use Control-D to log out.

# Docker Compose (https://docs.docker.com/compose/install/)

        # Download the current stable release of Docker Compose
        sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
        # Apply executable permissions to the binary
        sudo chmod +x /usr/local/bin/docker-compose

# Building the Compose File

First setup Media Server in a new compose file.

Docker compose uses a yml file. All of the files contain version and services object.

Create a directory for keeping the compose files.

        mkdir ~/server/compose
        mkdir ~/server/compose/media-server
        vi ~/server/compose/media-server/docker-compose.yml
        
As mentioned above all compose files contain a version string and a services object. The version string for this guide is 3.0, so the file will start out looking like:

        version: "3.0"
        services:
       
Save the following content to the docker-compose.yml file. 

# Jackett

 Jackett is where you define all your torrent indexers. All the *arr apps use the tornzab feed provided by jackett to search torrents.
 
        jackett:
        container_name: jackett
        image: linuxserver/jackett
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/${USER}/server/configs/jackett:/config'
          - '/home/${USER}/server/torrents:/downloads'
        ports:
          - '9117:9117'
        restart: unless-stopped
        
# Sonarr - TV

Sonarr is a TV show scheduling and searching download program. It will take a list of shows you enjoy, search via Jackett, and add them to the qbittorrent downloads queue.

        sonarr:
            container_name: sonarr
            image: linuxserver/sonarr
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            ports:
              - '8989:8989'
            volumes:
              - '/home/${USER}/server/configs/sonarr:/config'
              - '/home/${USER}/server:/data'
            restart: unless-stopped

# Radarr - Movies

        radarr:
            container_name: radarr
            image: linuxserver/radarr
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            ports:
              - '7878:7878'
            volumes:
              - '/home/${USER}/server/configs/radarr:/config'
              - '/home/${USER}/server:/data'
            restart: unless-stopped

# Lidarr - Music

        lidarr:
            container_name: lidarr
            image: ghcr.io/linuxserver/lidarr
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            volumes:
              - '/home/${USER}/server/configs/liadarr:/config'
              - '/home/${USER}/server:/data'
            ports:
              - '8686:8686'
            restart: unless-stopped

# Bazarr - Subtitles

        bazarr:
            container_name: bazarr
            image: ghcr.io/linuxserver/bazarr
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            volumes:
              - '/home/${USER}/server/configs/bazarr:/config'
              - '/home/${USER}/server:/data'
            ports:
              - '6767:6767'
            restart: unless-stopped

# Jellyfin

I personally only use jellyfin because it's completely free. I still have plex installed because overseerr which is used to request movies and tv shows require plex. But that's the only role plex has in my setup.

For the media volume you only need to provide access to the /data/media directory instead of /data as jellyfin doesn't need to know about the torrents.

        jellyfin:
            container_name: jellyfin
            image: ghcr.io/linuxserver/jellyfin
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            ports:
              - '8096:8096'
            devices:
              - '/dev/dri/renderD128:/dev/dri/renderD128'
              - '/dev/dri/card0:/dev/dri/card0'
            volumes:
              - '/home/${USER}/server/configs/jellyfin:/config'
              - '/home/${USER}/server/media:/data/media'
            restart: unless-stopped

        plex:
            container_name: plex
            image: ghcr.io/linuxserver/plex
            ports:
              - '32400:32400'
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
              - VERSION=docker
            volumes:
              - '/home/${USER}/server/configs/plex:/config'
              - '/home/${USER}/server/media:/data/media'
            devices:
              - '/dev/dri/renderD128:/dev/dri/renderD128'
              - '/dev/dri/card0:/dev/dri/card0'
            restart: unless-stopped
            
# Overseer/Ombi - Requesting Movies and TV shows

You can use ombi only if you don't plan to install plex.

        ombi:
        container_name: ombi
        image: ghcr.io/linuxserver/ombi
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/${USER}/server/configs/ombi:/config'
        ports:
          - '3579:3579'
        restart: unless-stopped

    overseerr:
        container_name: overseerr
        image: ghcr.io/linuxserver/overseerr
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/${USER}/server/configs/overseerr:/config'
        ports:
          - '5055:5055'
        restart: unless-stopped
        
Qbittorrent - Torrent downloader

I use qflood
container. Flood provides a nice UI and this image automatically manages the connection between qbittorrent and flood.

Qbittorrent only needs access to torrent directory, and not the complete data directory.

        qflood:
            container_name: qflood
            image: hotio/qflood
            ports:
              - "8080:8080"
              - "3005:3000"
            environment:
              - PUID=1000
              - PGID=1000
              - UMASK=002
              - TZ=Asia/Kolkata
              - FLOOD_AUTH=false
            volumes:
              - '/home/${USER}/server/configs/qflood:/config'
              - '/home/${USER}/server/torrents:/data/torrents'
            restart: unless-stopped

Heimdall - Dashboard

There are multiple dashboard applications but I use Heimdall.

        heimdall:
            container_name: heimdall
            image: ghcr.io/linuxserver/heimdall
            environment:
              - PUID=1000
              - PGID=1000
              - TZ=Asia/Kolkata
            volumes:
              - '/home/${USER}/server/configs/heimdall:/config'
            ports:
              - 8090:80
            restart: unless-stopped

# Transcoding

As I mentioned in the jellyfin section there is a section in the conmpose file as "devices". It is used for transcoding. If you don't include that section, whenever transcoding happens it will only use CPU. In order to utilise your gpu the devices must be passed on to the container.

I am using CPU-only transcoding as I the PC I am using does not have an independent GPU.

## CPU-only Transcoding

        plex:
            image: linxuserver/plex
            container_name: plex
            volumes:
                - /mnt/data/media:/media
                - ./config/plex:/config
            environment:
                - PUID=1000
                - PGID=1000
                - version=docker
            ports:
                - 32400:32400
            restart: unless-stopped


https://jellyfin.org/docs/general/administration/hardware-acceleration.html
Read up this guide to setup hardware acceleration for your gpu.

Generally, the devices are same for intel gpu transcoding.

## Intel GPU Transcoding

In order for Intel GPU transcoding to work, additionally install the intel-gpu-tools package, which will include both a command for monitoring our GPU's usage, and the underlying driver that makes it possible to use the GPU as a standalone device.

Install it with

        sudo apt-get install intel-gpu-tools

Afterwards, add the entry:

        plex:
            image: linuxserver/plex
            container_name: plex
            volumes:
                - /mnt/data/media:/media
                - ./config/plex:/config
            devices:
                - "/dev/dri:/dev/dri"
            environment:
                - PUID=1000
                - PGID=1000
                - version=docker
            ports:
                - 32400:32400
            restart: unless-stopped

## NVIDIA GPU Transcoding

NVIDIA is the most complicated process of the bunch, but is still doable in docker. First, download the Linux drivers for your GPU from the official NVIDIA drivers page. After clicking search and the first download button, when you get to the last page that contains the text "This download includes the NVIDIA graphics driver" right-click the "DOWNLOAD" button and copy the link. Then, in your Linux server machine, run the following commands:

        cd /tmp
        wget -O driver.run [paste your link here, but don't inlcude the brackets!]
        chmod +x driver.run
        sudo ./driver.run

This will bring up a pseudo-GUI. Follow the instructions and reboot if asked. To verify that your NVIDIA GPU has registered, run the command

        sudo nvidia-smi

It should output information about your GPU and current utilization. If it tells you it cannot detect an NVIDIA gpu, reinstall the drivers or try an earlier version.

Once the driver has been registered, the NVIDIA docker repository can be added with the following command (copy the whole thing)

        distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
        && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
        && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

And installed with this next command (copy the whole thing)

        sudo apt-get update \
        && sudo apt-get install nvidia-docker2

Finally, we can add the following to our docker-compose.yml file 

        plex:
            image: linuxserver/plex
            container_name: plex
            volumes:
                - /mnt/data/media:/media
                - ./config/plex:/config
            environment:
                - PUID=1000
                - PGID=1000
                - version=docker
                - NVIDIA_VISIBLE_DEVICES=all
            runtime: nvidia
            ports:
                - 32400:32400
            restart: unless-stopped
            
# Running and configuring the docker stack

Once the file has been built, we can start everything with one command while in the same folder as our docker-compose.yml file:

        docker-compose up -d
     

