2019/05/09 by Damien L-G

# Install the NVIDIA Driver

0. Update all installed packages.

    sudo yum update

1. Disable Nouveau driver
Blacklist Nouveau open-source driver that ships with CentOS.

    sudo echo blacklist nouveau >> /etc/modprobe.d/blacklist.conf
    sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
    sudo dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
    sudo reboot

2. Install Linux Kernel headers.

    sudo yum groupinstall "Development Tools"
    sudo yum install kernel-devel kernel-headers dkms

3. Download and install the NVIDIA driver.

    wget http://us.download.nvidia.com/tesla/418.67/NVIDIA-Linux-x86_64-418.67.run
    sudo sh NVIDIA-Linux-x86_64-418.67.run --accept-license

NOTE: answered `No` to the question whether to install 32-bit compatibility libraries.

NOTE: got the following error message:

    An incomplete installation of libglvnd was found. All of the essential libglvnd libraries are present, but one or more optional components are missing. Do you want to install a full copy of libglvnd? This will overwrite any existing libglvnd libraries.

selected `Install and overwrite existing files`

4. Verify that the driver is installed properly.
Queries the GPU devices status by calling the NVIDIA System Management Interface to check that the NVIDIA driver is properly installed.

    nvidia-smi

    Thu May  9 16:14:32 2019       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 418.67       Driver Version: 418.67       CUDA Version: 10.1     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla V100-PCIE...  Off  | 00000000:00:05.0 Off |                    0 |
    | N/A   56C    P0    41W / 250W |      0MiB / 32480MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                   
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+

# Install Docker

Following instructions to install Docker CE (Community Edition) on https://docs.docker.com/install/linux/docker-ce/centos/

1. Set up Docker stable repository

    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

2. Install Docker CE

    sudo yum install docker-ce docker-ce-cli containerd.io

3. Start it and run `hello-world`

    sudo systemctl start docker
    sudo docker run --rm hello-world

    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    1b930d010525: Pull complete
    Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
    Status: Downloaded newer image for hello-world:latest
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.
    
    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash
    
    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/
    
    For more examples and ideas, visit:
     https://docs.docker.com/get-started/

Moving on to https://docs.docker.com/install/linux/linux-postinstall/
4. Create `docker` group and add the `cades` user to it

    sudo groupadd docker
    sudo usermod -aG docker $USER

NOTE: got the following message

    groupadd: group 'docker' already exists

NOTE: do not forget to log out and log back in so that the group membership is re-evaluated.

5. Configure Docker to start on boot

    sudo systemctl enable docker

6. Enable command-line completion

    sudo yum install -y bash-completion
    sudo curl -sL https://raw.githubusercontent.com/docker/docker-ce/$(docker version --format '{{.Server.Version}}' | cut -f1-2 -d".")/components/cli/contrib/completion/bash/docker -o /etc/bash_completion.d/docker

NOTE: need to log in and log back out before to be able to run docker whithout sudo

# Install Docker Compose

1. Download the latest version of Docker Compose

    sudo curl -sL "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

NOTE: make sure to check the [Compose repository release page on GitHub](https://github.com/docker/compose/releases) as this will become out-of-date.

2. Apply executable permissions to the binary.

    sudo chmod +x /usr/local/bin/docker-compose

3. Install bash completion.

    sudo curl -sL https://raw.githubusercontent.com/docker/compose/1.23.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

# Install nvidia-docker (version 2.0)

1. Setup the nvidia-docker repository.

    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -sL https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

2. Install the nvidia-docker2 package and reload the Docker daemon configuration.

    sudo yum install nvidia-docker2
    sudo pkill -SIGHUP dockerd

3. Verify that things are working properly.

    docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi

    Unable to find image 'nvidia/cuda:latest' locally
    latest: Pulling from nvidia/cuda
    38e2e6cd5626: Pull complete
    705054bc3f5b: Pull complete
    c7051e069564: Pull complete
    7308e914506c: Pull complete
    5260e5fce42c: Pull complete
    8e2b19e62adb: Pull complete
    9b3d4105edff: Pull complete
    4c87e0decc6a: Pull complete
    Digest: sha256:948ad13aa11d94cb42e7e193059fbd7460ef7268bfac97738925978e1336e3bd
    Status: Downloaded newer image for nvidia/cuda:latest
    Mon Mar  4 21:00:20 2019
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 410.104      Driver Version: 410.104      CUDA Version: 10.0     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla V100-PCIE...  Off  | 00000000:00:05.0 Off |                    0 |
    | N/A   79C    P0    50W / 250W |      0MiB / 32480MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+

2019/05/17

# Install Git

    sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
    sudo yum install gcc perl-ExtUtils-MakeMaker
    wget https://www.kernel.org/pub/software/scm/git/git-2.21.0.tar.gz
    tar xf git-2.21.0.tar.gz
    cd git-2.21.0/
    make prefix=/opt/local/git all
    sudo make prefix=/opt/local/git install
    sudo mkdir -p /opt/local/etc/bash_completion.d 
    sudo cp contrib/completion/git-completion.bash /opt/local/etc/bash_completion.d/
    sudo cp contrib/completion/git-prompt.sh /opt/local/etc/bash_completion.d/
