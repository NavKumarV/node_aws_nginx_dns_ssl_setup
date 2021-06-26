# Node server hosted on AWS EC2 instance with dns and ssl configured. 

> Steps to deploy a Node.js app to AWS EC2 instance using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Create an EC2 instance from your AWS account.
Login to your AWS account, search for EC2 instance and create a EC2 instance with default VPC, storage and save the `key.pem` file. Then in the security groups add `Inbound rules` for port HTTP(80) and HTTPS(443) along with SSH(22).

## 2. Connect to the EC2 instance through SSH (there are other ways as well)
Open terminal and connect to the EC2 instance from the kep.pem file as Root user.

## 3. Install Node/NPM
```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install nodejs
node --version
```

## 4. Clone your project from Github
You can create your own node app. **OR** clone https://github.com/NavKumarV/node-express-typescript-sample-app , but with this one will have to install typescript `sudo npm i -g tsc` *Refer to the steps from that repo*
```
git clone yourproject.git 
OR
git clone https://github.com/NavKumarV/node-express-typescript-sample-app.git
```

### 5. Install dependencies and test app
```
cd yourproject
npm install
# start app
node ./dist/app.js (or whatever your start command)
# stop app
ctrl+C
```
## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start ./dist/app.js (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
#### check if your app is running with the below command if you have clone the project from my repo, should be able to a text *INDEX*
```
curl -v ec2-instance:3000  
```

## 7. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 3000) and see your app. Now let's add a domain

## 8. Add domain in AWS
In AWS console, go to Route53 and create a new host domain name (This costs a price) (OR **use the existing domain name if you have any**)

Add 2 A records with 
> www.yourdomain.com pointing to your EC2 public IP

> yourdomain.com pointing to your alias www.yourdomain.com


## 9. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```

Now visit https://yourdomain.com and you should see your Node app

## 10. Enjoy your Deployment
```
Say ya ya ya yaaaaaa and Shut down your system and sleep well happily...zzzZZ
```
