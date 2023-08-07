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

- Activate your configuration by linking to the config file from Nginx’s sites-enabled directory: ```sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/```
- This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing: ```sudo nginx -t```
- We also need to disable default Nginx host that is currently configured to listen on port 80, for this run: ```sudo unlink /etc/nginx/sites-enabled/default```
- Reload NGINX to apply changes: ```sudo systemctl reload nginx```
- The new website at /var/www/projectLEMP  is active but still empty. Let's create an index file to make sure it's working as it should:
```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
- Access your website using ```http://<Public-IP-Address>:80``` or ```http://<Public-DNS-Name>:80``` to make sure you are able to see the html file you added.
- Open a new file called info.php within your document root in your text editor to test if your server is able to process php requests: ```sudo nano /var/www/projectLEMP/info.php```. Paste the code in the file: ```<?php
phpinfo();```
- You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php: http://`server_domain_or_IP`/info.php

> Always remember to delete the info.php file as it contains sensitive information about your server

## Retrieving data from mysql database with php
> We will create a database named example_database and a user named example_user.
- First, connect to the MySQL console using the root account: ```sudo mysql```. if it fails to connect as you have set your password earlier, use ```sudo mysql -u root -p``` to connect and enter the root password already set earlier
- Next, create a new database using the command: ```CREATE DATABASE `example_database`;```
- create a new user and grant it full privileges: ```CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';``` 
- grant full privileges over the example_database: ```GRANT ALL ON example_database.* TO 'example_user'@'%';```
> This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.
- exit the database: ```exit```
- You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials: ```mysql -u example_user -p```
- when you login, check if you can access the database by running the command: ```SHOW DATABASES;```
- Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement: 
```
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
- Insert a few rows of content in the test table: ```INSERT INTO example_database.todo_list (content) VALUES ("My first important item");```
- To confirm that the data was successfully saved to your table, run:  ```SELECT * FROM example_database.todo_list;```
- After confirming that you have valid data in your test table, you can exit the MySQL console: ```exit```
- Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor: ```nano /var/www/projectLEMP/todo_list.php```
- The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception. Copy this content into your todo_list.php script:
```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
- You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php: http://<Public_domain_or_IP>/todo_list.php
