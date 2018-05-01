[![Build Status](https://travis-ci.org/vegasbrianc/github-monitoring.svg?branch=master)](https://travis-ci.org/vegasbrianc/github-monitoring)

# A Docker Stack which Monitors your GitHub Repos
Here's a quick start to stand-up a Docker [Prometheus](http://prometheus.io/) stack containing Prometheus, Grafana and  [github-exporter](https://github.com/infinityworksltd/github-exporter) to collect and graph GitHub statistics.

## Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of docker and [docker-compose](https://docs.docker.com/compose/install/) on your Docker host machine. This has also been tested with Docker for Mac and it works well.

## Installation
Clone the project to your Docker host. 

If you would like to change which targets should be monitored or make configuration changes edit the [/prometheus/prometheus.yml](https://github.com/vegasbrianc/prometheus/blob/version-2/prometheus/prometheus.yml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yml` file. 

## Configuration
In order to pull GitHub stats consistently it is recommended you create a personal access token inside of GitHub. This token will allow you to query the GitHub API more frequently than a public user. [Create GitHub Token](https://github.com/settings/tokens). It is only necessary to give the repo scope to the token permission.

<center><img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/github_token.png" width="600" heighth="500"></center>

Copy the GitHub Token you created and paste into the bottom of the [docker-compose.yml](https://github.com/vegasbrianc/github-monitoring/blob/master/docker-compose.yml) file under the metrics service section replacing the `GITHUB_TOKEN` with your newly created token.

The REPOS variable can also be updated to point to the Repos that you wish to monitor. In my example I monitor freeCodeCamp and Docker.

     metrics:
      tty: true
      stdin_open: true
      expose:
        - 9171
      image: infinityworks/github-exporter:latest 
      environment:
        - REPOS=freeCodeCamp/freeCodeCamp, docker/docker
        - GITHUB_TOKEN=<GitHub API Token see README>
      networks:
        - back-tier


Once configurations are done let's start it up. From the /prometheus project directory run the following command:

    $ docker-compose up -d


That's it. docker-compose builds the entire Grafana and Prometheus stack automagically. 

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000

username - admin
password - foobar (Password is stored in the `config.monitoring` env file)

The DataSource and Dashboard for Grafana are automatically provisioned. You can still install the dashboard manually if you choose below.


<center><img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/Grafana_Add_Data_Source.png" width="400" heighth="400"></center>

## Manual Install Dashboard
I created a Dashboard template which is available on [GitHub Stats Dashboard](https://grafana.net/dashboards/1559). Simply download the dashboard and select from the Grafana menu -> Dashboards -> Import

This dashboard is intended to help you get started with graphing your GitHub Repos. If you have any changes you would like to see in the Dashboard let me know so I can update Grafana site as well.


<center><img src="https://github.com/vegasbrianc/github-monitoring/blob/master/images/dashboard.png" width="4600" heighth="500"></center>

## Troubleshooting
It appears some people have reported no data appearing in Grafana. If this is happening to you be sure to check the time range being queried within Grafana to ensure it is using Today's date with current time.
