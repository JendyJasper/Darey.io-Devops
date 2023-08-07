# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

> Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

- Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
- Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
```
sudo apt update
sudo apt install nginx
```

- Configure Nginx LB using Web Servers’ names defined in /etc/hosts
- Open the default nginx configuration file: `sudo vi /etc/nginx/nginx.conf`
```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```

- Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
sudo systemctl config nginx
```

> REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
- Register a new domain name with any registrar
- Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5742c89a-2f6a-4c9c-92dc-51cb1aa6ba73">

- Update A record in your registrar to point to Nginx LB using Elastic IP address
- Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>
- Configure Nginx to recognize your new domain name
- Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com
- remove nginx default welcome page and reload the nginx server:
```
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx reload
```

- Install certbot and request for an SSL/TLS certificate. Make sure snapd service is active and running.: `sudo systemctl status snapd`
and `sudo snap install --classic certbot`
- Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it )
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

- Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>
- You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
- Click on the padlock icon and you can see the details of the certificate issued for your website.   


- Set up periodical renewal of your SSL/TLS certificate
- By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.
- You can test renewal command in dry-run mode: `sudo certbot renew --dry-run`
- Best practice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day. To do so, lets edit the crontab file with the following command:

`crontab -e`
Add following line: `* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

## we just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.

<img width="561" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8ff061f0-88d7-40b7-a7ed-e76874c4b22f">

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e119f2b0-dbc0-47b7-9db8-5069dae9cda3">
