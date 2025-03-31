<h1 align="center">
    <img src="./images/logo.png" alt="MediaMTX / rtsp-simple-server" width='75%'>
</h1>

# Introduction

The application *SweetCam* is a honeypot for IP camera. It can simulate a real IP camera vividly, including the interaction with user for rotating and zooming.

# Components

The SweetCam honeypot application is composed by four parts:

1. The MySQL service for data storage.
2. The RTSP service, this is used to provide the RTSP service for the attackers.
3. The Web service, this is used to provide the web service for the attackers, including viewing the camera page, logging etc.
4. The Cowrie service, this is used as the SSH honeypot for providing the SSH service for the attackers.

# How to run the application

To run launch the application, just enter the root directory of the application and launch the application with the following command (remember to add a .env file with required environment variables):

```sh
docker compose up -d
```

Thereafter, there should be four containers that are running as shown follows:

1. web_service
2. cowrie_service
3. rtsp_service
4. mysql_service

Once the four services are lunched, there are several configurations should be made within the rtsp_service:

1. Enter the rtsp_service container with the following command:

   ```sh
   docker exec -it rtsp_service /bin/sh
   ```

2. Revise the mediamtx.yml file to configure the logging function of the Mediamtx application.

3. Then use the FFmpeg tool to push the video to mediamtx server (RTSP server)

# Deploy on cloud

See [Azure Cloud](https://azure.microsoft.com/en-us/products/cloud-services) as example.

## Configure SSH

First we need to change the used SSH port since the default one 22 should be used by Cowrie honeypot.
1. sudo vim /etc/ssh/sshd_config.
2. Change the port to another one, 2404 for example.
3. Restart ssh service: sudo service ssh restart
4. Reconnect with new port: ssh -i ./sweetcam_key.pem azureuser@20.197.231.249 -p 2404
5. Revise the virtual machine network policy to allow 2404 traffic.

## Install Docker tool chain

To install docker, refer to the original script provided by `docker`.
```sh
# Up to date script for install.
curl -fsSL https://raw.githubusercontent.com/docker/docker-install/master/install.sh -o install.sh
# Make it executable.
sudo chmod +x install.sh
# Run script with sudo.
sudo ./install.sh
# Add user to the docker user group to run docker without sudo. Logout and login with the user.
sudo usermod -aG docker $USER
```

## Deploy application

The deployement goes as follows:
1. Revise the virtual machine network policy to allow 80, 554, 2404 (customized port for SSH), 22 traffic.
2. Clone the repository with `git clone https://github.com/Agachily/sweetcam.git`.
3. Create the .env from template file and populate the variables `cp .template.env .env`.
4. Run the application in the background: `docker compose up -d` (stop with `docker compose down -v`)
5. Enter the container of rtsp service: `docker exec -it rtsp_service /bin/sh` and configure the logging function.
6. Use FFmpeg to push to video to RTSP server. `ffmpeg -nostdin -re -stream_loop -1 -i ./videos/fake-video.mp4 -c copy -f rtsp rtsp://localhost:8554/mystream`.
7. View it at `rtsp://public_ip:554/mystream`.
8. Check the volumes: `docker volume ls`.
9. Inspect volume: `docker volume inspect sweetcam_rtsp-resource`.
10. Get the logs from the volume.

## Example of getting the logs from remote honeypot

```sh
scp -r -i ./key.pem -P 2404 azureuser@public_ip:/home/azureuser/logs/sweetcam_cowrie-log ./attack-logs/machine-a/
```