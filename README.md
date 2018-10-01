
# A Docker Stack which Monitors your home network
Here's a quick start to stand-up a Docker [Prometheus](http://prometheus.io/) stack containing Prometheus, Grafana with  [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) and [speedtest-exporter](https://github.com/stefanwalther/speedtest-exporter) to collect and graph home network connections and speed.

## Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of docker and [docker-compose](https://docs.docker.com/compose/install/) on your Docker host machine. This has also been tested with Docker for Mac and it works well.

## Installation
Clone the project to your Docker host. 

If you would like to change which targets should be monitored or make configuration changes edit the [/prometheus/prometheus.yml](./prometheus/prometheus.yml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yml` file. 

## Configuration
To change what hosts you ping you change the `targets` section in [/prometheus/pinghosts.yml](./prometheus/pinghosts.yml) file.

For speedtest the only relevant configuration is how often you want the check to happen. It is at 5 minutes by default which might be too much if you have limit on downloads. This is changed by editing `scrape_interval` under `speedtest` in [/prometheus/prometheus.yml](./prometheus/prometheus.yml).


Once configurations are done let's start it up. From the /prometheus project directory run the following command:

    $ docker-compose up -d

That's it. docker-compose builds the entire Grafana and Prometheus stack automagically. 

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3030` for example http://localhost:3000

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

## Interesting urls

http://localhost:9090/targets shows status of monitored targets as seen from prometheus - in this case which hosts being pinged and speedtest. note: speedtest will take a while before it shows as UP as it takes ~30s to respond.

http://localhost:9090/graph?g0.expr=probe_http_status_code&g0.tab=1 shows prometheus value for `probe_http_status_code` for each host. You can edit/play with additional values. Useful to check everything is okey in prometheus (in case Grafana is not showing the data you expect).

http://localhost:9115 blackbox exporter endpoint. Lets you see what have failed/succeded.

http://localhost:9696/metrics speedtest exporter endpoint. Does take ~30 seconds to shohw its result as it runs an actual speedtest when requested.


