# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

For this project I will be configuring Nginx load balancer solution and incorporating security into our deployed site. 
When data is moving between a client (browser) and a web server over the internet - it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-in-the-Middle (MIMT) attack.

## INFRASTRUCTURE
* AWS EC2 INSTANCE (NGINX WEBSERVER, WEBSERVER1, WEBSERVER 2, NFS SERVER, DB SERVER)
* ROUTE 53
* ELASTIC IP

## ARCHITECHTURE
The target architecture will look like this: 

[target_architecture](./images/architecture.PNG)


# CONFIGURE NGINX AS A LOAD BALANCER

1. Create an Nginx webserver which will be configured as a loadbalancer

[nginx](./images/nginx_server.jpg)

2. The hosts file for the local dns is updated with the webservers and their private ip addresses using the command `sudo vi etc/hosts`

[config_hosts](./images/updateetchosts.PNG)

3. configure nginx as a load balancer to point traffic to the resolvable dns names of the webservers. The nginx configuration is done using the below commands
```
sudo vi /etc/nginx/nginx.conf

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

[nginx_config](./images/confignginx.PNG)

4. Restart nginx server and verify

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

# REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Let's make necessary configurations to make connections to our tooling web solution secured. 

1. Obtain a new domain name by registering it.
2. Allocate an elastic IP to the nginx Load Balancer (LB) server. This prevents the public IP address from changing whenever the VM restarts.
[elasticip](./images/elasticipass.jpg)
3. Utilize Route 53 as the DNS web service to direct the webserver to the newly acquired domain name
[route53](./images/route53.PNG)
4. Configure nginx to recognize the new domain name within the server_name block in its configuration file

** Please note that it may take some time for the DNS to resolve and the nameservers to update, typically ranging from 1 to 48 hours.

To verify the successful loading of the website and the correctness of the setup, perform a test.
[testsite](./images/website.PNG)

We can see that the website is up and running but our work here is not done. We still need to ensure that the website is secure.

### SECURING THE WEBSITE

1. Certbot will be used to obtain the certificate for the domain name. First we need to make sure snapd is running on the server using the command 
```
sudo systemctl status snapd
```

[snapd](./images/snapdstatus.PNG)

2. Certbot is installed and a certificate is requested using the following commands
```
sudo apt install certbot -y
sudo apt install python3-certbot-nginx -y
sudo nginx -t && sudo nginx -s reload
sudo certbot --nginx -d <domain> -d www.<domain>
```
Now we have a valid certificate and we can test this. The site is reloaded.
[images](./images/securesite1.PNG)

3. Now we set up renewal for the ssl/tls certificate. First we edit the crontab file with the following command: 
`contab -e` 
Now we add the following line
`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

[crontabconf](./images/cronconfig.PNG)


