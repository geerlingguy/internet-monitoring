# A Docker Stack which Monitors your GitHub Repos
Here's a quick start to stand-up a Docker [Prometheus](http://prometheus.io/) stack containing Prometheus, Grafana and  [github-exporter](https://github.com/infinityworksltd/github-exporter) to collect and graph GitHub statistics.

##Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of docker and [docker-compose](https://docs.docker.com/compose/install/) on your Docker host machine. This has also been tested with Docker for Mac and it works well.

##Installation
Clone the project to your Docker host. 

If you would like to change which targets should be monitored or make configuration changes edit the [/prom/prometheus.yml](https://github.com/vegasbrianc/prometheus/blob/version-2/prometheus/prometheus.yml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yml` file. 

## Configuration
In order to pull GitHub stats consistently it is recommended you create a personal access token inside of GitHub. This token will allow you to query the GitHub API more frequently than a public user. [Create GitHub Token](https://github.com/settings/tokens). 

<img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/github_token.png" width="400" heighth="400">

Copy the GitHub Token you created and paste into the bottom of the [docker-compose.yml](https://github.com/vegasbrianc/github-monitoring/blob/master/docker-compose.yml) file under the metrics service section replacing the `GITHUB_TOKEN` with your newly created token.

The REPOS variable can also be updated to point to the Repos that you wish to monitor. In my example I monitor freeCodeCamp and Docker.

     metrics:
      tty: true
      stdin_open: true
      expose:
        - 9171
      image: infinityworks/github-exporter:latest 
      environment:
        - REPOS=freeCodeCamp/freeCodeCamp,docker/docker
        - GITHUB_TOKEN=<GitHub API Token see README>
      networks:
        - back-tier


Once configurations are done let's start it up. From the /prometheus project directory run the following command:

    $ docker-compose up -d


That's it. docker-compose builds the entire Grafa and Prometheus stack automagically. 

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000

username - admin
password - foobar (Password is stored in the `config.monitoring` env file)

## Post Configuration
Now we need to create the Prometheus Datasource in order to connect Grafana to Prometheues 
* Click the `Grafana` Menu at the top left corner (looks like a fireball)
* Click `Data Sources`
* Click the green button `Add Data Source`.

<img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/Add_Data_Source.png" width="400" heighth="400">

## Alerting
Alerting has been added to the stack with Slack integration. 2 Alerts have been added and are managed 

Alerts              - `prometheus/alert.rules`
Slack configuration - `alertmanager/config.yml`

The Slack configuration requires to build a custom integration.
* Open your slack team in your browser https://<your-slack-team>.slack.com/apps
* Click build in the upper right corner
* Make Custom integration 
* Choose Incomming Web Hooks
* Select which channel
* Copy the Webhook URL into the `alertmanager/config.yml` URL section
* Fill in Slack username and channel

View Prometheus alerts `http://<Host IP Address>:9090/alerts`
View Alert Manager `http://<Host IP Address>:9093`

### Test Alerts
A quick test for your alerts is to stop a service. Stop the node_exporter container and you should notice shortly the alert arrive in Slack. Also check the alerts in both the Alert Manager and Prometheus Alerts just to understand how they flow through the system.

High load test alert - `docker run --rm -it busybox sh -c "while true; do :; done"``

Let this run for a few minutes and you will notice the load alert appear.

## Install Dashboard
I created a Dashboard template which is available on [GitHub Stats Dashboard](https://grafana.net/dashboards/1559). Simply download the dashboard and select from the Grafana menu -> Dashboards -> Import

This dashboard is intended to help you get started with graphing your GitHub Repos. If you have any changes you would like to see in the Dashboard let me know so I can update Grafana site as well.


<img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/dashboard.png" width="400" heighth="400">

## Troubleshooting
It appears some people have reported no data appearing in Grafana. If this is happening to you be sure to check the time range being queried within Grafana to ensure it is using Today's date with current time.
