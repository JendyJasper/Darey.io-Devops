# LEMP STACK IMPLEMENTATION


- Create an ec2 instance on AWS
- Run ```sudo apt update``` to update your packages
- run ```sudo apt install nginx``` to install nginx
- run ```sudo systemctl status nginx``` to confirm it's running as a service
- enable port 80 on your instance security group to allow web connections from anywhere
- test if the web server is running locally on your ubuntu seerver by typing ```curl http://localhost:80``` on your browser
- test if it can receive requests from the internet. Get your public ip address by running the command, ```curl -s http://169.254.169.254/latest/meta-data/public-ipv4```
- Install mysql database to manage data for the site by running ```sudo apt install mysql-server```
- login to mysql using ```sudo mysql```
- set a password for mysql root user ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';```
- ```exit``` mysql
- run a security script that will remove insecure default settings ```sudo mysql_secure_installation```
- Don't enable ```VALIDATE PASSWORD``` if you don't want to set a specified criteria for passwords to be accepted
- type yes and press enter for all the other prompts
- install PHP packages: ```sudo apt install php-fpm php-mysql```

## CONFIGURING NGINX TO USE PHP PROCESSOR
> #### just like in Apache server where you create virtual hosts to host more than one domain on same server, you can dop same on NGINX by creating server blocks. NGINX default server block is located in a directory named /var/www/html. Instead of editing this, we will create a different direcory to handle a new server block, and use the default server block to handle requests when known particular domain/site is specified
- Create the root web directory for your_domain as follows: ```sudo mkdir /var/www/projectLEMP```
- Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user: ```sudo chown -R $USER:$USER /var/www/projectLEMP```
- Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano: ```sudo nano /etc/nginx/sites-available/projectLEMP```
- paste the below code on the generated file: 

``` 
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

>
Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
root — Defines the document root where the files served by this website are stored.
index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.
>
