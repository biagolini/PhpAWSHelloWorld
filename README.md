# Deploying a PHP Page on AWS EC2

This tutorial will guide you through the process of deploying a PHP page on an AWS EC2 instance using Ubuntu.

## Requirements

Before you begin, ensure you have the following requirements:

- An AWS account with access to launch EC2 instances and configure security groups.
- Basic knowledge of AWS services and EC2 instances.
- SSH access to your EC2 instance using a key pair.
- Basic knowledge of Git and GitHub.

## Setup

Step 1:Launch an Ubuntu EC2 instance in a public subnet.
Make sure that the security group attached to your instance allows public access (i.e., 0.0.0.0/0) to ports 443 and 80. Additionally, allow inbound access to port 22 from your IP address.

Use the following User Data while launching the instance:

```bash
#!/bin/bash
sudo apt update -y
sudo apt upgrade -y
sudo apt-get install -y php awscli git
```

Step 2: SSH to your Instance

```bash
ssh ec2-user@INSTANCE-IP -i YourKey.pem
```

Step 3: Check if your dependencies are correctelly installed

```bash
php -v
aws --v
git --version
```

Step 4: Create a Folder
Create a folder to store your project by running the following commands:

```bash
mkdir -p ~/myApps/public
cd ~/myApps
```

Step 5: Clone the Git Repository

Before cloning the repository, you may want to configure your Git credentials:

```bash
git config --global user.email "your_email@example.com"
git config --global user.name "Your Name"
```

Then, clone the repository by running the following command:

```bash
git clone git@github.com:biagolini/PhpAWSHelloWorld.git
sudo chmod -R 755 PhpAWSHelloWorld
```

Step 6: Configure the Web Server
Navigate to the project directory and obtain the path to your PHP project:

```bash
cd ~/myApps/PhpAWSHelloWorld
pwd # This will display the path to your PHP project, for example: /home/ubuntu/myApps/PhpAWSHelloWorld
```

Edit the Apache configuration file by running the following command:

```bash
sudo nano /etc/apache2/sites-available/myapp.conf
```

Paste the following content into the configuration file, replacing /home/your-username/myApps/public with the actual path to your PHP project:

```
<VirtualHost *:80>
    ServerName myapp.local
    ServerAdmin webmaster@localhost
    DocumentRoot /home/ubuntu/myApps/PhpAWSHelloWorld/public
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /home/your-username/myApps/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Next, edit the default Apache configuration file:

```ssh
sudo nano /etc/apache2/sites-available/000-default.conf
```

Update the `DocumentRoot` line to point to your application's public directory, and add the `<Directory>` block to allow access to the public directory. Replace `/home/your-username/myApps/public` with the actual path to your application's public directory:

```conf
DocumentRoot /home/ubuntu/myApps/PhpAWSHelloWorld/public
<Directory /home/ubuntu/myApps/PhpAWSHelloWorld/public>
    AllowOverride All
    Require all granted
</Directory>
```

Enable the new virtual host configuration:

```ssh
sudo a2ensite myapp.conf
```

To activate the new configuration, reload Apache:

```ssh
sudo systemctl reload apache2
```

Step 7: Test the Web Server
Open your internet browser and access your instance's public IP. You should see the "Hello World" page.
