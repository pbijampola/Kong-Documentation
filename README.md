
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
8.Step 7: Accessing Konga GUI \
9.Step 8: Testing Konga API Gateway

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

## Step 7: Accessing Konga GUI

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
## step 8: Setting up SSL certificate

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
 

