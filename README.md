# nightscout-docker
Comprehensive Docker Setup for Nightscout CGM Monitor on Raspberry Pi 4 or any Linux Computer/Server

## Credit
I have gotten both the Dockerfile and the docker-compose.yml from liske/cgm-remote-monitor-docker, thank you!

## Backstory
My girlfriend is a type 1 diabetic and so I got into CGM Monitoring.
I wanted to create a guide of how I host my own Nightscout server under my own domain.
It is very easy for everybody that owns a Linux Server (for example a Raspberry Pi 4).
I will first explain how to setup Nightscout and later also how to setup Nginx so you can host Nightscout securely. If you don't own a domain you can use a free one (google is your friend). If you have any questions feel free to just write an e-mail to florian@schickel.me and I can try my best to help you.

## Requirements for this to work

* Any sort of Linux Computer/Server, have tested it both on ARM processors and AMD processors
* Docker and Docker-Compose installed (google it if you don't know how to do that)
* Administrative access to your router

## Setup Nightscout

* I have included the Dockerfile that I used, but it is for reference only
* First Clone this repository to your server and go into the folder
```sh
git clone https://github.com/pyrmon/nightscout-docker.git && cd nightscout-docker
```
* Edit the docker-compose-amd.yml file
    * Change the API_SECRET value to a password with the min length of 12, I don't know if all special characters are okay
    * Change the BASE_URL to the webadress you want to access Nightscout from later on 
    * If you don't use mg/dl then you can change DISPLAY_UNITS value to mmol
```yml
      # admin secret
      - API_SECRET=xxxxxxxxxxxx
      # the URL to this Nightscout instance
      - BASE_URL=https://ns.xxxxx.com
      ...
      # use SI units by default
      - DISPLAY_UNITS=mg/dl
```
* This is it. Now follow this command and your Nightscout is running on the webadress http://localhost:1337/ if you run it on the same machine that you are accessing the web browser with or http://IPOFSERVER:1337/
```sh
docker-compose up -d
```

## Setup Nginx

* This will only be the explanation on how to get Nginx running
* Just enter these commands and it will setup Nginx Proxy Mangager on http://localhost:81 / http://IPOFSERVER:1337/
```sh
cd nginx
docker-compose -f docker-compose-nginx.yml up -d
```
* After this you can sign in as admin@example.com with the password changeme

## Backup Nightscout
* It makes sense to backup your mongo db in case that something crashes. For this reason I have included the backupNightscout.sh file
* All you need to do is run this command:
```sh
./backupNightscout.sh
```
* If you are running it for the first time you might need to give the shell file the right to be executed, if required you do this with:
```sh
chmod +x backupNightscout.sh
```
* If you want this to run this on a specific schedule then I would recommend creating a cronjob for this
    * A cronjob is a task that your linux machine executes on a specific schedule [Cron](https://en.wikipedia.org/wiki/Cron)
    * To create a cronjob you need to execute ```crontab -e```
    * Then an entry like this need to be inserted ```00 2 * * 0-6 /bin/bash /path/to/backupNightscout.sh```
    * This would execute the backup script every day at 2 am, make sure to put in your path to the backupNightscout.sh
    * If you want to change the schedule you can lookup how to scheduling works here [Crontab Guru](https://crontab.guru/#0_2_*_*_0-6)
* The backup will put the backup zips to ~/nightscout-backups/
* To restore from a new mongodb you need to execute ```docker exec -d nightscout_mongo mongorestore /path/to/unzipped/restore-files```
* If you need help with a restore from the zip feel free to reach out
