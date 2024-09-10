Hello all, I created this page to share my knowledge related with computers and networks. Here you will find a description of my home network, the hardware and software I use and how it was setup.
I will put links to my blog page (https://qualquercoisa.eu/) related with these topics. In my blog I will describ how I configured everything. Hope you enjoy, and fell free to give me your feedback.

The Hardware:
- 6 x Raspberry Pi 4 4GB
- 2 x Raspberry Pi Zero WiFi
- 1 x Dell PowerEdge R610
- 1 x HP EliteDesk G800 G2 Mini

The OS's:
- Raspberry Pi OS
- VMware ESXi
- Ubuntu Server 20.04

The Software:
- OpenMediaVault
- Docker
- PiVPN
- PiHole
- UpTime Kuma
- NetAlertX
- Portainer
- HomeAssistant
- JellyFin
- Grafana
- Prometheus
- PhotoPrism


Monitoring on Raspberry Pi with Node Exporter, Prometheus, and Grafana with Docker – Part 1/3

This post is a partial copy of the post from Nishanth Nagendra in the site https://easycode.page/monitoring-on-raspberry-pi-with-node-exporter-prometheus-and-grafana/

In this post I made some small changes to the original one, and these were the setting and configuration I used in my network.

I will not explain what is a Raspberry Pi, Grafana, Prometheus or Node Exporter, because if you are here, it means that you are looking for this information.

Steps to Deploy Node Exporter on Docker
1. Setup folders to maintain the container data

# Create a directory for the project and each component
mkdir -p monitoring/node-exporter
mkdir -p monitoring/prometheus
mkdir -p monitoring/grafana
2. Navigate to the node-exporter folder and run the container using the docker command as shown

# Navigate to the node-exporter directory
cd monitoring/node-exporter
# Run the node-exporter docker container
sudo docker run -d \
--name="node-exporter" \
--net="host" \
--pid="host" \
-v "/:/host:ro,rslave" \
--restart=always \
quay.io/prometheus/node-exporter:latest --path.rootfs=/host
# Node exporter is installed on 9100 port by default
3. Verify node exporter is working by browsing to http://<IP>:9100 where “IP” is the IP address of the Pi/Machine on which the docker container is deployed

Monitoring
4. Click on the “Metrics” to view all the collected metrics as shown

Monitoring
Congratulations!! You’ve successfully deployed Node Exporter on Docker

Steps to Deploy Prometheus on Docker
1. Navigate to the Prometheus folder created previously and create a YAML configuration file for Prometheus

# Navigate to the prometheus directory
cd monitoring/prometheus
# Create a file called prometheus.yml
touch prometheus.yml
2. Create the Prometheus YAML configuration file as shown

# Edit the file using a file editor (nano is this case)
sudo nano prometheus.yml
# Add the below content to the file
global:
  scrape_interval: 5s
  external_labels:
    monitor: 'node'
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['192.168.0.128:9090'] ## IP Address of the localhost
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['192.168.0.128:9100'] ## IP Address of the localhost
# The Port with the prometheus job will be the port on which prometheus will be deployed (9090 in this case)
# The Port with the node-exporter job will be the port on which node-exporter is deployed (9100 in this case)
# Here a sample IP address has been used (192.168.0.128). Replace this with the IP of your Pi/Machine
3. Run the Prometheus docker container

# -d specifies the container to run in detached state
# --name specifies the name of the container
# -p specifies the port mapping where the left side indicates the host and the right side indicates to container
# -v specifies the volume mount and this must point to the location of the prometheus.yml file created in the previous step
# --restart always ensures the container restarts if it goes down due to any circumstances
sudo docker run -d \
--name prometheus \
-p 9090:9090 \
-v /home/pi/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
--restart always \
prom/prometheus
4. Verify Prometheus is working by browsing to http://<IP>:9090 where “IP” is the IP address of the Pi/Machine on which the docker container is deployed

Monitoring
5. Navigate to the Targets section to verify node-exporter and Prometheus is up

Monitoring
Monitoring
Congratulations!! You’ve successfully deployed Prometheus on Docker

Steps to Deploy Grafana on Docker
1. Navigate to the grafana folder created previously

# Navigate into the grafana folder
cd monitoring/grafana
2. Run the Grafana docker container as shown

# -d specifies the container to run in detached state
# --name specifies the name of the container
# -p specifies the port mapping where the left side indicates the host and the right side indicates to container
# --restart always ensures the container restarts if it goes down due to any circumstances
sudo docker run -d \
--name=grafana \
-p 9003:3000 \
--restart=always \
grafana/grafana
3. Verify grafana is deployed by browsing to http://<ip>:9003 where IP is the IP of the Pi/Machine. The Port is the port that has been specified during the deployment of the docker container (9003 in this case). You should see the login screen as shown below

Monitoring
4. Login to grafana using the following credentials:

Username: admin

Password: admin

You will be asked to change the password as shown

Monitoring
5. You should now be able to see the Grafana dashboard

Monitoring
Congratulations!! You’ve successfully deployed Grafana on Docker

Steps to Configure a Data Source in Grafana
1. Navigate to the Data Source Configuration as shown below

Monitoring
2. Click on “Add Data Source” and choose “Prometheus”

Monitoring
Monitoring
3. Configure the data source to the node-exporter url as setup previously and then click on “Save and Test”

Monitoring
Monitoring
4. Ensure the Data Source is working

Monitoring
Congratulations!! You’ve successfully setup the data source for grafana

Steps to add a Dashboard on Grafana
1. Search for a ready-made template dashboard here: https://grafana.com/grafana/dashboards/?orderBy=downloads&direction=desc&search=node+exporter

2. Copy the ID of the dashboard. For this tutorial, we will be using this dashboard: https://grafana.com/grafana/dashboards/1860. The ID of the dashboard is 1860 in this case

3. On the Grafana Home page click on the import option in the dashboard section

Monitoring
4. Enter the ID of your selected dashboard and click on Load

Monitoring
5. Select the Prometheus installation and click on Import

Monitoring
6. Once it is imported, you will be able to see the dashboard populated with the metrics of your Pi

Monitoring
Congratulations!! You’ve successfully configured a Dashboard to view the metrics of your Raspberry Pi
