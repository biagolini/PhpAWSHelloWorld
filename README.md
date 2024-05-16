# Deploying a simple PHP Page on AWS EC2

This tutorial will guide you through the process of deploying a simple PHP page using two contexts:

1. AWS Ubuntu 22.04 EC2 instance using Apache;
2. AWS Amazon Linux EC2 instance using NGINX;

## Requirements

Before you begin, ensure you have the following requirements:

- An AWS account with access to launch EC2 instances and configure security groups.
- Basic knowledge of AWS services and EC2 instances.
- SSH access to your EC2 instance using a key pair.
- Basic knowledge of Git and GitHub.

## AWS Ubuntu 22.04 EC2 Instance Using Apache

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
git clone https://github.com/biagolini/PhpAWSHelloWorld.git
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

```
DocumentRoot /home/ubuntu/myApps/PhpAWSHelloWorld/public
<Directory /home/ubuntu/myApps/PhpAWSHelloWorld/public>
    AllowOverride All
    Require all granted
</Directory>
```

Enable the new virtual host configuration:

```bash
sudo a2ensite myapp.conf
```

To activate the new configuration, reload Apache:

```bash
sudo systemctl reload apache2
```

Step 7: Test the Web Server
Open your internet browser and access your instance's public IP. You should see the "Hello World" page.

## AWS Amazon Linux EC2 Instance Using NGINX

Step 1:Launch an Ubuntu EC2 instance in a public subnet.
Make sure that the security group attached to your instance allows public access (i.e., 0.0.0.0/0) to ports 443 and 80. Additionally, allow inbound access to port 22 from your IP address.

Use the following User Data while launching the instance:

```bash
#!/bin/bash
sudo yum upgrade -y
sudo yum install -y php git nginx php-fpm
```

Step 2: SSH to your Instance

```bash
ssh ec2-user@INSTANCE-IP -i YourKey.pem
```

Step 3: Check if your dependencies are correctelly installed

```bash
aws --v
php -v
git --version
nginx -v
php-fpm -
```

Step 4: Navigate to the NGINX default web root directory

```bash
cd /usr/share/nginx/html
```

Step 5: Remove the default files

```bash
sudo rm -rf *
```

Step 6: Clone the Git Repository

Before cloning the repository, you may want to configure your Git credentials:

```bash
git config --global user.email "your_email@example.com"
git config --global user.name "Your Name"
```

Then, clone the repository locally by running the following command:

```bash
sudo git clone https://github.com/biagolini/PhpAWSHelloWorld.git .
```

Step 7: Configure NGINX

Edit the NGINX configuration file:

```bash
sudo nano /etc/nginx/conf.d/default.conf
```

Modify the configuration to include the following PHP settings

```
server {
    listen 80;
    server_name YOUE_INSTANCE_IP;

    root /usr/share/nginx/html/public;
    index index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    error_page 404 /404.html;
    location = /40x.html {}

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {}
}
```

Validate an Nginx config file

```bash
sudo nginx -t
```

Step 8. Start and Enable PHP-FPM

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

Step 9. Restart NGINX

```bash
sudo systemctl restart nginx
```

Step 10: Test the Web Server

Open your internet browser and access your instance's public IP. You should see the "Hello World" page.

## Contributing

Feel free to submit issues, create pull requests, or fork the repository to help improve the project.

## License and Disclaimer

This project is open-source and available under the MIT License. You are free to copy, modify, and use the project as you wish. However, any responsibility for the use of the code is solely yours. Please use it at your own risk and discretion.
