# Matomo-Web-Analytics
Deploy Matomo Web Analytics on cloud



### What is Matomo?

Matomo is the leading free, open-source analytics platform developed by a team of international developers, that runs on a PHP/MySQL webserver. This platform tracks online visits to one or more websites and displays reports on these visits for analysis. 

This project requires to install Matomo and a MariaDB database using Docker Compose, then install Nginx to act as a reverse proxy for the Matomo app. Finally, enable secure HTTPS connections by using Certbot to download and configure SSL certificates from the Let’s Encrypt Certificate Authority.

In order to successful complete this project, it is important to follow these instructions below:

~~~
$ ssh root@your_server_ip


