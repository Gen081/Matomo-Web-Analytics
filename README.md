# Matomo-Web-Analytics

## Deploy Matomo Web Analytics on cloud



### What is Matomo?

Matomo is the leading free, open-source analytics platform developed by a team of international developers, that runs on a PHP/MySQL webserver. This platform tracks online visits to one or more websites and displays reports on these visits for analysis. 

This project requires to install Matomo and a MariaDB database using Docker Compose, then install Nginx to act as a reverse proxy for the Matomo app. Finally, enable secure HTTPS connections by using Certbot to download and configure SSL certificates from the Let’s Encrypt Certificate Authority.

In order to successful complete this project, it is important to follow these instructions below:

### 1.  An Ubuntu 20.04 server, with the UFW firewall enabled.

Create a new Ubuntu 20.04 server, and perform some important configuration steps as part of the initial setup. These steps will increase the security and usability of the server, and will give a solid foundation for subsequent actions.

##### Step 1 — Logging in as root

For security purposes, log in as root can be done several ways: a password or – configuration of an SSH key authentication – the private key for the root user’s account is important. 

I inially connect to my EC2 server using the following:

![](pics/log-root.png) 


##### Step 2 — Creating a New User

In order to create a new user, it is important to log in as root. The root user is the administrative user in a Linux environment that has very broad privileges. Type the following command:

~~~
# adduser sammy
~~~

![](pics/adduser.png)

A few questions will be asking, starting with the account password. Enter a strong password, the additional information is not required, just hit `ENTER` to skip.


##### Step 3 — Granting Administrative Privileges

As the new user account is now created, let's add some privileges to it. These privileges will be needed to perform administratives tasks. The process to grant privileges is to add the user to the sudo group. By default, on Ubuntu 20.04, users who are members of the sudo group are allowed to use the sudo command.

~~~
usermod -aG sudo sammy
~~~

![](pics/grant-adm-P.png)


##### Step 4 — Setting Up a Basic Firewall

Ubuntu 20.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. Let's set up a basic firewall using this application.

OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW,

```
# ufw app list
```

Ensure that the firewall allows SSH connections so that we can log back in next time. Allow these connections by typing: 

```
# ufw allow OpenSSH
```

Enable the firewall by typing:

```
# ufw enable
```
Type `yes` and press `ENTER` to proceed. 


```
# ufw status
```

![](pics/firewall-set.png)

##### Step 5 — Enabling External Access for Your Regular User

Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly

![](pics/newuserroot-access.png)

Accessing that new user, there might be some permission issue. To fix that, type:

```
sudo nano /etc/ssh/sshd_config
```
Enter the password for the new user. Then it will open a nano text template as such:

![](pics/newuser-nano.png)
Scroll down and look for :
```
#PermitRootLogin yes
``` 

```
PasswordAuthentication yes
```

change the `no` that already assigned to `yes`
then: 
```
sudo service ssh restart
```


#### How to Set Up SSH Keys on your PC

##### Step 1 — Creating the Key Pair

```
$ ssh-keygen
```
![](pics/keypair.png)

#### Copying the Public Key Using ```ssh-copy-id```

The `ssh-copy-id` tool is included by default in many operating systems, so you may have it available on your local system. For this method to work, you must already have password-based SSH access to your server.

The syntax is:

```
$ ssh-copy-id username@remote_host
```
![](pics/copy-public-key.png)


#### Step 3 - Authenticating to Your Ubuntu Server Using SSH Keys

The basic process is the same:

```
$ ssh username@remote_host
```
![](pics/authenticate-ubuntuserver.png)

#### Step 4 - Disabling Password Authentication on Your Server

Before completing the steps in this section, make sure that you either have SSH-key-based authentication configured for the root account on this server, or preferably, that you have SSH-key-based authentication configured for a non-root account on this server with sudo privileges.

```
$ sudo nano /etc/ssh/sshd_config
```
![](pics/step4.png)

Inside the file, search for a directive called `PasswordAuthentication`. This will disable the ability to log in via SSH using account passwords:

```
. . .
PasswordAuthentication no
. . .
```


![](pics/pssauthenticate:no.png)

Save and close the file when you are finished by pressing `CTRL+X`, then `Y` to confirm saving the file, and finally `ENTER` to exit nano. To actually activate these changes, restart the `sshd` service:

```
$ sudo systemctl restart ssh
```

Open up a new terminal window and test that the SSH service is functioning correctly before closing the current session:

```
$ ssh username@remote_host
```
![](pics/pssauthenticate1:no.png)





### 2. Docker installed. 

**Docker** is an application that simplifies the process of managing application processes in **containers** . They’re similar to virtual machines, but containers are more portable, more resource-friendly, and more dependent on the host operating system.

#### Prerequisites

To follow this tutorial, you will need the following:

- One Ubuntu 20.04 server set up  including a sudo non-root user and a firewall.
- An account on [Docker Hub](https://hub.docker.com/) if you wish to create your own images and push them to Docker Hub.

#### Step 1 — Installing Docker

The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. 

First, update your existing list of packages:

```
$ sudo apt update
```
If an error occurs, please type the following command:

![](pics/docker-sudo-apt.png)

```
sudo nano /etc/apt/sources.list
```

This command will lead to a nano script, then place a space between `focalstable` as indicated below:

![](pics/docker-focal.png)

Type this command again:

![](pics/docker-sudo-update.png)


Next, install a few prerequisite packages which let apt use packages over HTTPS:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
![](pics/docker-prereq.png)

Then add the GPG key for the official Docker repository to the system:

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

and Add the Docker repository to APT sources:

```
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal table"

```

![](pics/docker-gpgnrepo.png)

It is important to install from the Docker repo instead of the default Ubuntu repo:

```
apt-cache policy docker-ce
```

![](pics/docker-ce.png)


Now, let's install Docker.

![](pics/docker-install.png)


Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```
$ sudo systemctl status docker
```

![](pics/docker-daemon.png)


#### Step 2 — Executing the Docker Command Without Sudo (Optional)


By default, the `docker` command can only be run the **root** user or by a user in the **docker** group, which is automatically created during Docker’s installation process. To avoid typing sudo whenever you run the docker command, add your username to the docker group:

first run :

```
$ sudo username
```
then 

```
$ sudo usermod -aG docker ${USER}
```
![](pics/docker-sudouser.png)

To apply the new group membership, log out of the server and back in, or type the following:

```
$ su - ${USER}
```

It will be prompted to enter the user’s password to continue.
Confirm that your user is now added to the **docker** group by typing:

```
$ groups
```

![](pics/docker-su-user.png)

If need to add a user to the docker group that you’re not logged in as, declare that username explicitly using:

![](pics/docker-usermod.png)



#### Step 3 — Using the Docker Command

To view all available subcommands on docker, type:

```
$ docker
```


#### Step 4 — Working with Docker Images

Docker containers are built from Docker images. By default, Docker pulls these images from Docker Hub, a Docker registry managed by Docker, the company behind the Docker project. 

To check whether you can access and download images from Docker Hub, type:

```
$ docker run hello-world
```
![](pics/docker-img.png)

Search for images available on Docker Hub by using the docker command with the search subcommand. Type: 

```
$ docker search ubuntu
```

![](pics/docker-img1.png)

Execute the following command to download the official `ubuntu` image to your computer:

```
$ docker pull ubuntu
```

![](pics/docker-img2.png)

To see the images that have been downloaded to your computer, type:

```
$ docker images
```

#### Step 5 — Running a Docker Container

The hello-world container ran in the previous step is an example of a container that runs and exits after emitting a test message. Containers can be much more useful than that, and they can be interactive. Containers are similar to virtual machines, but more resource-friendly.

Let’s run a container using the latest image of Ubuntu. 

```
$ docker run -it ubuntu
```

![](pics/docker-container.png)

Let’s update the package database inside the container. 

```
$ sudo apt update
```

Then install any application in it. Let’s install Node.js:

```
$ sudo apt install nodejs
```
It is  also important to configurate geographic area and time zone while inside the docker container and installing `node.js`

![](pics/docker-container1.png)
![](pics/docker-container2.png)



#### Step 6 — Managing Docker Containers

After using Docker for a while, you’ll have many active (running) and inactive containers on your computer. To view the active ones, use:

```
docker ps
```

![](pics/docker-container3.png)


To view all containers — active and inactive, run:

![](pics/docker-container4.png)

To view the latest container you created, pass it the `-l` switch:

![](pics/docker-container5.png)

To start a stopped container, use docker start, followed by the container ID or the container’s name. Let’s start the Ubuntu-based container with the ID of : `7b32f846f649`

![](pics/docker-container6.png)


To stop a running container, use `docker stop`, followed by the container ID or name. This time, we’ll use the name that Docker assigned the container, which is `optimistic_agnesi`:

![](pics/docker-container6.png)

#### Step 7 — Committing Changes in a Container to a Docker Image

When start up a Docker image, you can create, modify, and delete files just like you can with a virtual machine. 
This section shows you how to save the state of a container as a new Docker image.


After installing Node.js inside the Ubuntu container, you now have a container running off an image, but the container is different from the image you used to create it. But you might want to reuse this Node.js container as the basis for new images later.

Then commit the changes to a new Docker image instance using the following command:

![](pics/docker-commit.png)

then typing:

```
$ docker images
```
![](pics/docker-commit1.png)


#### Step 8 — Pushing Docker Images to a Docker Repository

The next logical step after creating a new image from an existing image is to share it with a select few of your friends, the whole world on Docker Hub, or other Docker registry that you have access to. To push an image to Docker Hub or any other Docker registry, you must have an account there.


To push your image, first log into Docker Hub.

```
docker login -udocker-registry-username
```

![](pics/docker-push1.png)

then 

![](pics/docker-push2.png)

Push and commit docker image

![](pics/docker-push3.png)


### 3. Docker Compose installed.

Docker simplifies the process of managing application processes in containers. While containers are similar to virtual machines in certain ways, they are more lightweight and resource-friendly. This allows developers to break down an application environment into multiple isolated services.

Docker Compose is a tool that allows you to run multi-container application environments based on definitions set in a YAML file. It uses service definitions to build fully customizable environments with multiple containers that can share networks and data volumes.

#### Prerequisites

- Access to an Ubuntu 20.04 local machine or development server as a non-root user with sudo privileges. If you’re using a remote server, it’s advisable to have an active firewall installed.
- Docker installed on your server or local machine.

The following command will download the 1.27.4 release and save the executable file at /usr/local/bin/docker-compose, which will make this software globally accessible as docker-compose:

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

![](pics/docker-compose.png)

Next, set the correct permissions so that the docker-compose command is executable:

```
$ sudo chmod +x /usr/local/bin/docker-compose
```

To verify that the installation was successful, you can run:

```
$ docker-compose --version
```
![](pics/docker-compose1.png)


Now that the Docker Compose is successfully installed on your system. In the next section, let's set up a `docker-compose.yml` file and get a containerized environment up and running with this tool.


#### Step 2 — Setting Up a `docker-compose.yml` File

To demonstrate how to set up a docker-compose.yml file and work with Docker Compose, we’ll create a web server environment using the official Nginx image from Docker Hub, the public Docker registry. This containerized environment will serve a single static HTML file.

```
$ mkdir ~/compose-demo
cd ~/compose-demo
```

In this directory, set up an application folder to serve as the document root for your Nginx environment:

```
$ mkdir app
```

```
$ nano app/index.html
```

![](pics/docker-compose2.png)


```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Docker Compose Demo</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

	<h1>This is a Docker Compose Demo Page.</h1>
	<p>This content is being served by an Nginx container.</p>

</body>
</html>
```

```
$ nano docker-compose.yml
```

```
version: '3.7'
services:
  web:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./app:/usr/share/nginx/html
```
The docker-compose.yml file typically starts off with the version definition. This will tell Docker Compose which configuration version we’re using.


#### Step 3 — Running Docker Compose

With the docker-compose.yml file in place, let's now execute Docker Compose to bring our environment up.

With the docker-compose.yml file in place, we can now execute Docker Compose to bring our environment up.

The environment is now up and running in the background. To verify that the container is active, run:

Now access the demo application by pointing your browser to either `localhost:8000` or `your_server_domain_or_IP:8000`

![](pics/docker-demopage.png)




#### Step 4 — Getting Familiar with Docker Compose Commands

![](pics/docker-compose5.png)



### Step 1 — Running Matomo and MariaDB with Docker Compose
First step will be to create the Docker Compose configuration that will launch containers for both the Matomo app and a MariaDB database.

![](pics/matomo-run.png)

docker-compose.yml:

![](pics/nano-docker.png)



```
nano .env
```
![](pics/mariadb.png)

One way of generating a strong password is to use the openssl command, which should be available on most any operating system. The following command will print out a random 30 character hash that you can use as a password:

```
openssl rand 30 | base64 -w 0 ; echo
```

Now, ready to bring up the two containers with `docker-compose`:

![](pics/sudo-dock-compose.png)

![](pics/curl.png)


### Step 2 — Installing and Configuring Nginx

