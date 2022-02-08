# Steps/Resources to automate the creation of the ARM VM instance on the Oracle OCI

This repo helps to automate the creation of the ARM instance on the Oracle Cloud and handles `Out of host capacity.` error.

## Resources

- https://github.com/hitrov/oci-arm-host-capacity
- https://www.youtube.com/watch?v=uzAqgjElc64

___

## Steps to follow on the OCI dashboard

- Go to `cloud.oracle.com`
- Click `Profile` icon on the upper right corner
- Click `User Settings`
- Under `Resources` -> Click `API Keys`  -> Add API Key
- Download the public and private keys, downloaded files will have names like `oracleidentitycloudservice_kartikeyporwal-00-00-00-00_public.pem` `oracleidentitycloudservice_kartikeyporwal-00-00-00-00.pem` for public and private key
- Copy the configuration and save in a file named `config`
- Update the path of the downloaded private key from file like `oracleidentitycloudservice_kartikeyporwal-00-00-00-00.pem` in the `key_file=` section of the `config` file

___


## Steps to follow on the instance creation page

- Go to `https://cloud.oracle.com/compute/instances/`
- Under section `Image and shape`, click `Change Image` and select the image
- Under section `Image and shape`, click `Change Shape` and select the `Ampere`
- Toggle `Number of CPUs` to `4` and `Amount of memory (GB)` to `24`; click `Select Shape`
- Paste the public key from `ssh-key-2000-01-01_ubuntu@250.130.4.2.key.pub` to `Paste Public Keys` section in `Add SSH keys` and click create
- Track the network request for the url like `https://iaas.us-sanjose-1.oraclecloud.com/20160918/instances/` and `copy as CURL`; and save it in a file


___


## Steps to follow on the server where CRON job will be run


- Login to the machine where we have to run the CRON job

```bash


ssh -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key ubuntu@150.90.23.6


```


- Create a directory on the remote machine to install project

```bash

mkdir /home/ubuntu/.oci

```


- Copy private key file `oracleidentitycloudservice_kartikeyporwal-00-00-00-00.pem` from local machine to server's `/home/ubuntu/.oci`

```bash

scp \
    -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key \
    oracleidentitycloudservice_kartikeyporwal-00-00-00-00.pem \
    ubuntu@150.90.23.6:/home/ubuntu/.oci/

```


- Copy `config` file from local machine to server's `/home/ubuntu/.oci`

```bash

scp \
    -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key \
    config \
    ubuntu@150.90.23.6:/home/ubuntu/.oci/

```


- Clone the repo on the server

```bash

git clone https://github.com/hitrov/oci-arm-host-capacity.git


```

- Install PHP, composer and its dependencies

```bash

sudo apt install composer

sudo apt install php libapache2-mod-php
sudo systemctl restart apache2
sudo apt install php-fpm
systemctl status php7.4-fpm
sudo apt install php7.4-curl
sudo apt install php7.4-dom
sudo apt install php7.4-mbstring


```

- Install the dependencies of project `oci-arm-host-capacity`

```bash

composer install


```

- Copy the sample `.env.example` file from the project directory on server to the local machine as `.env` file

```bash

scp \
    -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key \
    ubuntu@150.90.23.6:/home/ubuntu/.oci/oci-arm-host-capacity/.env.example \
    .env


```

- Update the downloaded `.env` file on local machine

```bash

Copy OCI_SUBNET_ID, OCI_IMAGE_ID, OCI_SSH_PUBLIC_KEY from the tracked network request
Copy other params from the downloaded config file


```


- Copy the file to server

```bash

scp \
    -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key \
    .env \
    ubuntu@150.90.23.6:/home/ubuntu/.oci/oci-arm-host-capacity/.env


```


- Test run the app

```bash

php ./index.php


```

- Create a log file on the server where result of the PHP command will be saved

```bash


touch oci.log
chmod 777 /home/ubuntu/.oci/oci-arm-host-capacity/oci.log


```


- Check the PHP's binary path

```bash

# it should be something like /usr/bin/php
which php

```


- Create the CRON job, with the php path, that to be run each minute

```bash

EDITOR=nano crontab -e

Paste the below line and save the file
* * * * * /usr/bin/php /home/ubuntu/.oci/oci-arm-host-capacity/index.php >> /home/ubuntu/.oci/oci-arm-host-capacity/oci.log

```


- Regularly check the logs from file `/home/ubuntu/.oci/oci-arm-host-capacity/oci.log` for instance status

```bash

cat /home/ubuntu/.oci/oci-arm-host-capacity/oci.log 

```


- Copy the log from the server once the instance is created

```bash

scp -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key ubuntu@150.90.23.6:/home/ubuntu/.oci/oci-arm-host-capacity/oci.log oci.log


```


- Remove the CRON job entry

```bash

crontab -e

Add comment before the command

* * * * * /usr/bin/php /home/ubuntu/.oci/oci-arm-host-capacity/index.php >> /home/ubuntu/.oci/oci-arm-host-capacity/oci.log

to

# * * * * * /usr/bin/php /home/ubuntu/.oci/oci-arm-host-capacity/index.php >> /home/ubuntu/.oci/oci-arm-host-capacity/oci.log


```


## Steps to follow on the OCI dashboard once the instance is created

- Assign the public IP to the launched instance
    - Go to `instance page` on `OCI`
    - Go to `Resources` -> `Attached VNICs` -> Click `VNIC Name`
    - On `VNICs information` page, Go to `Resources` -> `IPv4 Addresses` -> For the private IP Address, click `Edit`
    - Toggle `Public IP Type` from `No Public IP` to `Ephemeral public IP` -> Click `Update`


## Access the created instance

- Login to machine

```bash

ssh -i ssh-key-2000-01-01_ubuntu@150.90.23.6.key ubuntu@250.130.4.2

```

- Copy `ssh-key-2000-01-01_ubuntu@150.90.23.6.key` to `ssh-key-2000-01-01_ubuntu@250.130.4.2.key`

```bash

cp ssh-key-2000-01-01_ubuntu@150.90.23.6.key ssh-key-2000-01-01_ubuntu@250.130.4.2.key


```

- Copy `ssh-key-2000-01-01_ubuntu@150.90.23.6.key.pub` to `ssh-key-2000-01-01_ubuntu@250.130.4.2.key.pub`

```bash

cp ssh-key-2000-01-01_ubuntu@150.90.23.6.key.pub ssh-key-2000-01-01_ubuntu@250.130.4.2.key.pub

```

- Login to machine using the new copied ssh file

```bash

ssh -i ssh-key-2000-01-01_ubuntu@250.130.4.2.key ubuntu@250.130.4.2


```