# LAMP STACK IMPLEMENTATION ON AWS

## STEPS

- Create an AWS account
- Create an EC2 instance running ubuntu
- Update your ubuntu package database using `sudo apt update`
- Install apache using `sudo apt install apache2`
- Check the status of apache if it’s running using `sudo systemctl status apache2`
- Activate port 80 on AWS security group for inbound connections. Allow IP from Anywhere to access it. port 80 is the port used by most web servers. 
- You can check if you can access the web browser locally using curl http://localhost:80
- Use this to get your public ip address curl -s http://169.254.169.254/latest/meta-data/public-ipv4
- Now, to access the address, enter a web browser and type http://<Public-IP-Address>:80. example http://54.237.44.210:80
- Set up mysql for database management using `sudo apt install mysql-server`
- Use `sudo mysql` to login to the mysql database which logs you in as root
- Run the command to set a password for your database to make it more secure: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`
- This script will remove some insecure default settings and lock down access to your database system. `sudo mysql_secure_installation`. 
- VALIDATE PASSWORD PLUGIN let’s you set password strength policy. You can leave it disabled
- When you’re done, test if you’re able to log in to the MySQL console by typing `sudo mysql -p`
- In addition to the php package, you’ll need  php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed as dependencies.
- To install all, use `sudo apt install php libapache2-mod-php php-mysql`
- Test the installation by typing `php -v`
  
  ### Now, the complete LAMP framework is ready
 > In this project, we will set up a domain called projectlamp. Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory. We will leave this configuration as is and will add our own directory next next to the default one.

- Create the directory for projectlamp using ‘mkdir’ command as follows: `sudo mkdir /var/www/projectlamp`
- Next, assign ownership of the directory with your current system user: `sudo chown -R $USER:$USER /var/www/projectlamp`
- Then, create and open a new configuration file in Apache’s `sites-available` directory using your preferred command-line editor. Here, we’ll be using `vi or vim`
- Run the command `sudo vi /etc/apache2/sites-available/projectlamp.conf`
- This will create a new blank file. Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text: ` <VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost> `
- To save and close the file, simply follow the steps below:
> Hit the esc button on the keyboard
 Type :
 Type wq. w for write and q for quit
 Hit ENTER to save the file
- Type this to check if the file is available. `ls /etc/apache2/sites-available/`
- You can now use a2ensite command to enable the new virtual host: `sudo a2ensite projectlamp`
- You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this case Apache’s default configuration would overwrite your virtual host. To disable Apache’s default website use a2dissite command , type: `sudo a2dissite 000-default`
- To make sure your configuration file doesn’t contain syntax errors, run: `sudo apache2ctl configtest`
- Finally, reload Apache so these changes take effect: `sudo systemctl reload apache2`
> Your new website is now active, but the web root /var/www/projectlamp is still empty. Create an index.html file in that location so that we can test that the virtual host works as expected:
 ` sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html `
- Now go to your browser and try to open your website URL using IP address: http://54.237.44.210:80
- You can also use the public DNS name; ec2-54-237-44-210.compute-1.amazonaws.com. Result must be the same
  > You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

  
