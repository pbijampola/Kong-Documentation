
# Kong API Gateway and Konga GUI setup

This README provides a step-by-step guide on how to set up Kong API Gateway and Konga GUI on a Ubuntu server.


## Table of Contents
1.System Requirements \
2.Step 1: Install Kong API Gateway \
3.Step 2: Install PostgreSQL \
4.Step 3: Configure Kong \
5.Step 4: Start Kong \
6.Step 5: Install Konga (Kong Admin GUI) \
7.Step 6: Configure Konga \
8.Step 7: Setting SSL Certificate on Kong API Gateway (OPENSSL)
9.Steo 8: Setting Free SSL certificate from Let's Encrypt on Kong API Gateway

## System Requirements

- Ubuntu 20.04+ (or any recent LTS version)
- PostgreSQL (For kong database)
- Node.js (For Konga GUI)
- Kong API Gateway (Latest stable version)



## Step 1: Install Kong API Gateway

1. Install Kong's dependencies:
```bash
sudo apt install -y openssl libpcre3 procps perl
```
2. Download and install Kong.
```bash
# Add kong repository

echo "deb [trusted=yes] https://download.konghq.com/gateway-3.x-ubuntu-focal/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/kong.list

# Update package list

sudo apt Update

# Install Kong

sudo apt install -y kong

```


## Step 2: Install PostgreSQL

Kong requires PostgreSQL as its database,but any database can be used basing on the preference

```bash
# Install PostgreSQL

sudo apt install postgresql postgresql-contrib -y
```

### Configure PostgreSQL

1. Create a PostgreSQL user and database for Kong

```bash
 
 sudo -u postgresql psql 

 # Create database 

 CREATE DATABASE kong;

 # Create user

 CREATE USER kong WITH PASSWORD 'your_password';

 # Grant privileges

 GRANT ALL PRIVILEGES ON DATABASE kong TO kong;

 # Exit PostgreSQL

 \q
 ```

 2. Edit the PostgreSQL configuration to allow connections.

 ```bash
 sudo nano /etc/postgresql/12/main/pg_hba.conf
```

  Change the `local` and `host` methods to `md5` for enhanced security. (Optional)

3. Restart PostgreSQL

```bash 
sudo systemctl restart postgresql 
```
 
## Step 3: Configure Kong
By default after installing kong, has a configuration `kong.conf.default` file whereby you are suppose to make a copy and edit it.


1. Go to this dir:
```bash

 cd /etc/kong
``` 

2. Make a copy:
```bash

cp kong.conf.default kong.conf

```
3. Edit the `kong.conf` file:
```bash
sudo nano kong.conf
```
### NOTE : 

Edit the following sections/part at first by uncommenting (#)

```
1. proxy_listen

2. admin_listen

3. admin_gui_url (By specifing where KongAa GUI will be accessible)

Example: admin_gui_url <public IP:PORT>

Let on edit on the certificate (SSL)

# Database configuration

On the same file such for database conf

pg_host = 127.0.0.1
pg_port = 5432
pg_user = kong
pg_password = your_password
pg_database = kong
```

## Step 4: Start Kong

1. Run database migrations:
```bash
sudo kong migrations bootstrap
```
2. Start kong: 
```bash 
sudo kong start
```
3. Check Kong's status:
```bash
sudo kong health
```
4. Test kong api gateway
```bash
curl -i http://localhost:8001/

```

## Step 5: Install Konga (Kong Admin GUI)


1. Install `nvm`, a node version management tool. As of current i have faced issues with node modules nbe depreciated such `node-sass`

```bash
# Run nvm installer:

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

or

wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash


# Update profile configurations:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm

# Reload shell configuration:

source ~/.bashrc  # or source ~/.zshrc if you're using zsh

# To install the latest version of node  and npm

nvm install node

# Install node version 16 or above compatible with Konga

nvm install 16

# Use node 16 or above

nvm use 16

```

2. Clone this repository
```bash
$ git clone https://github.com/pantsel/konga.git
```
3. Navigate into konga directory

```bash
cd konga
```

4. Install dependencies
```bash
npm install
```
#### Note

If after installation errors arise about node-sass which is depreciated, install `node-sass` along with these recommended package `sass` and `sass-embedded`

```bash
npm i node-sass sass sass-embedded
```

5. Start Konga, by default, konga runs on port `1337`

```bash
npm start
```

## Step 6: Accessing Konga GUI

To access  Konga:
1. Visit the server's public IP on port `1337`
```bash
http://<server-public-ip>:1337
```
2. Create an admin account and configure your Kong instance by entering:

```bash
Kong Admin URL: http://<kong-public-ip>:8001
```
The kong public IP is http://localhost:8001


## Step 7: Setting  SSL certificate on the Gateway
The setup ensures that all API requests are secured over HTTPS.
1. Create OpenSSL Configuration File
   ```bash
   sudo nano /etc/ssl/openssl.cnf
   ```
2. Add the following content, replacing `<public-gateway-ip-address>` with actual IP address.
   ```bash
   [ req ]
   default_bits       = 2048
   distinguished_name = req_distinguished_name
   req_extensions     = req_ext
   
   [ req_distinguished_name ]
   countryName         = Country Name (2 letter code) e.g TZ
   stateOrProvinceName = State or Province Name e.g Dar es Salaam
   localityName        = Locality Name e.g Dar es Salaam
   organizationName    = Organization Name e.g Settlodev
   commonName          = `public-gateway-ip-address`
   
   [ req_ext ]
   subjectAltName = @alt_names
   
   [ alt_names ]
   IP.1 = `public-gateway-ip-address`
   ```
3. Generate the Certificate and Key
      Run the the following command to generate a private key and self-signed certification:
      ```bash
      sudo openssl req -new -x509 -nodes -days 365 -keyout /etc/ssl/private/ip.key -out /etc/ssl/certs/ip.crt -config /etc/ssl/openssl.cnf
      ```
      This command creates two files
      ```bash
      - `/etc/ssl/private/ip.key`
      - `/etc/ssl/certs/ip.crt`
      ```
4. Move SSL Certicate and key to the correct directory
   Create a directory for SSL certificate, key and move there:
    ```bash
       mkdir -p /etc/kong/ssl
       mv /etc/ssl/certs/ip.crt /etc/kong/ssl/certs
       mv /etc/ssl/private/ip.key /etc/kong/ssl/private
    ```
5. Configure Kong for SSL
   Locate and edit the `kong.conf` file
    ```bash
        sudo nano /etc/kong/kong.conf
        ```
        Upadate the following lines to specify the path to SSL certificate and key:
        ```bash
        ssl_cert = /etc/kong/ssl/certs/ip.crt
        ssl_cert_key = /etc/kong/ssl/private/ip.key
    ```
6. Restart Kong API Gateway:
   ```bash
         kong restart
   ```
7. Verify the Configuration
   To ensure that Kong is serving your API over HTTPS using the IP address, test it with curl. Replace `public-gateway-ip-address`
   ```bash
         curl -k https://192.168.1.1:8443/your-api-endpoint
         ```
         The -k option allows curl to connect without verifying the certificate (useful for self-signed certificates).
   ```

8. Troubleshooting Common Issues
   If you encounter issues such as `ETIMEDOUT` or `WRONG_VERSION_NUMBER`, consider the following troubleshooting steps:
   - Ensure that your server is running and accessible.
   - Check firewall settings to allow traffic on port specified. eg 8443 (CASE AWS CLOUD SERVICE)
   - Verify that Kong is correctly configured and listening on port 8443.
   - Test connectivity using tools like telnet or curl.
   - Review Kong's logs for any errors related to SSL/TLS connections.


## Step 8: Setting  SSL certificate on the Gateway (NOT INTERESTED ON USING OPENSSL)
The setup ensures that all API requests are secured over HTTPS using free ssl certificate from Lets Encrypt.

1. Stop services using port 80 if allowed and kong. For aws all the outbound for the instance on this port 80 to avoid the following issue
  Certbot failed to authenticate some domains (authenticator: standalone). The Certificate Authority reported these problems:
  Domain: gateway.settlo.co.tz
  Type:   connection
  Detail: <ip-address>: Fetching http://<domain>/.well-known/acme-challenge/2otjKPxPS3zMGy8qKY1-InLvrxPLq4dDEyzjXmglEUo: Timeout during connect (likely firewall problem)

Hint: The Certificate Authority failed to download the challenge files from the temporary standalone webserver started by Certbot on port 80. Ensure that the listed domains point to this machine and that it can accept inbound connections from the internet

```bash
  sudo systemclt stop ngnix
  sudo kong stop
```
 
2. install certbot
   ```bash
    sudo apt install python3-certbot-nginx 
   ```
3. Generate certbo. Do not put the port number at the end.
   ```bash
     sudo certbot certonly --standalone -d example.com 
   ```
4. Configure Kong for SSL
   Locate and edit the `kong.conf` file
    ```bash
        sudo nano /etc/kong/kong.conf
        ```
        Upadate the following lines to specify the path to SSL certificate and key:
        ```bash
        ssl_cert = /etc/letsencrypt/live/<your-domain>/fullchain.pem
        ssl_cert_key = /etc/letsencrypt/live/<your-domain>/privkey.pem
    ```
5. Set up automatic renewal. This will add a cron job to the default crontab.
```bash
  echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```
6. Restart the service and Good to Go 

## Documentation

1. [Kong Installation Documentation](https://docs.konghq.com/gateway/3.8.x/install/linux/ubuntu/?install=oss#package-install) 
2. [Kong Data Store Documentation](https://docs.konghq.com/gateway/3.8.x/install/post-install/set-up-data-store/)
2. [Kong Services and Routes Documentation](https://docs.konghq.com/gateway/3.8.x/get-started/services-and-routes/)
3. [Kong Gateway Tutorial: Running With a GUI](https://konghq.com/blog/engineering/kong-gateway-tutorial)


## Contributing

Contributions are always welcome!

See `contributing.md` for ways to get started.

Please adhere to this project's `code of conduct`.


## Authors

- [@PATRICK BIJAMPOLA](https://github.com/pbijampola)


## Appreciation

 - [Simplify API Management.Unlock AI Innovation.](https://konghq.com/)
 - [@WALTER PETER, CTO at Settlo](https://github.com/Walter-Peter)
 

