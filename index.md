### How I created a Cloud VPN with Docker, DigitalOcean, and WireGuard.

## 1.
I used digital ocean to create an Ubuntu server application.
So the first step is to make a digital ocean account and make a new droplet with ubuntu as its OS.

## 2.
With your droplet created you can access your droplet's console by pressing the button with three dots on the far right and then select to access the console.
Now that you're in your droplet's console, we need to install docker before we can install wireguard, but to do that we have to install the necessary components for docker. Simply enter this command:
`sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`

## 3.
Now to add the docker key
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - `


## 4.
After adding the docker key we can easily add the docker repo, but there are many different docker repos depending on your droplet's sytem. I'm using a basic 32 / 64 bit OS so this is the command I used:
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

## 5.
Now I finally installed docker using this command:
`sudo apt install docker-ce -y`

## 6.
Then I installed docker compose using this command:
`sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

## 7.
Now to change the permission bit of docker compose to be easily executed:
`sudo chmod +x /usr/local/bin/docker-compose`

## 8.
Setup wireguard:
```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

## 9.
Now that we're editing the docker-compose.yml file, add these contents:
```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - SERVERURL=147.182.202.172
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

## 10.
There are several options here that need to be edited. The first is the timezone designoted by the line `TZ=`. 
You need to put in your specific time zone, which you can find a link to [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
Next you need to edit the serverurl value in the line that starts with `SERVERURL=`. You can find the URL on the DigitalOcean dashboard.
The final option you need to change is the user-config-files. You can specify the amount of user-files you want by either specifying an amount right next to the line beginning with `PEERS=`, or you can specify the user-config-files you'd like explictly like `pc1, pc1, phone1`

## 11.
Then, after saving that file, run these commands to start Wireguard:
```
  cd ~/wireguard/
  docker-compose up -d
```
## 12.
Finally, runt his command to pull up the QR codes and execution log of the Wireguard VPN:
`docker-compose logs -f wireguard`
Now we can connect to this wireguard server using our mobile phones if you install the Wireguard VPN application (it can be found on the iOS and Android appstore).
I visited [this](https://ipleak.net) link to get my current ip address before I used the VPN:
![MobileNoVPN](/MobileNoVPN.PNG)

Then I used used the wireguard mobile-app and scaned the QR code output of my terminal for the phone1 config file and was able to connect ot it and use it as my VPN. After using it my new ip on ipleak.net looked like this:
![MobileYesVPN](/MobileYesVPN.PNG)

## 13.
Next I used this wireguard VPN on my Mac. I simply installed the Wireguard app from the MacOS appstore and added the details in a new tunnel, which then asked if I wanted to use it as a VPN and I said yes.
I got the details by scanning the Pc1 QR code using my phone's camera instead of the Wireguard app, and it took me to a link that had all of the necessary details to create the tunnel:
![VPNDetails](/VPNDetails.png)
Here's what it looked like on my Mac when I had the VPN working:
![MacVPN](/MacVPN.png)
