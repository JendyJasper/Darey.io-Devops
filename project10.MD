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
- 
