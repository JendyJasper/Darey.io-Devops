# LAMP STACK IMPLEMENTATION ON AWS

### STEPS

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
  
  #### Now, the complete LAMP framework is ready
 
