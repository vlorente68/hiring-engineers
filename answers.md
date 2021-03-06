# Introduction
Companies are changing the way they access and run application using modern architectures, moving to cloud environments, public, priveate or hybrid. These architectures provide great capabilities in term of scalability and flexibility, but it creates a big challenge to monitor such complex architectures with multiples layers and dependencies. 

Here is when Datadog comes to help, providing a flexible monitoring and analytics cloud platform, able to monitor and analyze traditional and modern application architectures, including cloud environments, containrers, servers, databases, etc. If something is not ready you can create it...

# Prerequisites - Setup the environment

To avoid any compatibility or dependency issues I used an Ubuntu 16.04 VM running on Virtual Box using a Vagrant image. I already had a datadog trial account that was expired, and I requested to extend it in order to run the Challenge.

## Installing the VM environment

First step is to add the box from vagrant catalog to the local vagrant app
```
$ vagrant box add hashicorp-vagrant/ubuntu-16.04
```
Then we have to modify the configuration fiile **Vagrantfile**
```
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp-vagrant/ubuntu-16.04"
end
```
To start the environment we run the following command:
```
vagrant up
```
Now we have the environment up and running.

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Virtualbox.png?raw=true">

## Installing the agent

Now we have to install the datadog agent into our VM to start monitoring it. It is really easy to do it, just need run the specific command for your OS:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Agent platform.png?raw=true">


To install the Datadog agent into the Ubuntu server, is as easy as running from a shell the following command:

```
DD_API_KEY=c7c0572c87dc9c1295865e5fb4246307 bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"
```
This is the answer to the command that confirm that the agent is running

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Agent Installed.png?raw=true">

We can see in the [Datadog Host Map](https://app.datadoghq.com/infrastructure/map?fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&groupby=availability-zone&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=green_to_orange&paletteflip=false&node_type=host) that we are already monitoring it.

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Host Map Initial.png?raw=true">

# Collecting Metrics:

* Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

Datadog uses tags as a mechanism to filter, aggregate and compare metrics and infrastructure elements. Tagging is a very good mechanism to provide flexibility and multi-dimensioning to data.

It is possible to add tags to the datadog agent in the **datadog.yaml** file:

```
tags:
  - owner:vicente
  - project:technical_test
  - region:spain
```
We can see in the [Host Map](https://app.datadoghq.com/infrastructure/map?fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&groupby=availability-zone&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=green_to_orange&paletteflip=false&node_type=host) the new added tags to the agent:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Host Map with Tags.png?raw=true">


* Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

We are goint to install into our VM the database server MongoDB version 3.6.9 Community Edition by using the following commands:
```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5
$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org=3.6.9 mongodb-org-server=3.6.9 mongodb-org-shell=3.6.9 mongodb-org-mongos=3.6.9 mongodb-org-tools=3.6.9
$ echo "mongodb-org hold" | sudo dpkg --set-selections
$ echo "mongodb-org-server hold" | sudo dpkg --set-selections
$ echo "mongodb-org-shell hold" | sudo dpkg --set-selections
$ echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
$ echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```
Now we are going to install the datadog integration for MongoDB following the instruction at the datadog portal:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/MongoDB Integration.png?raw=true">

We run the commands at the MongoDB shell to create a user for datadog:
```
> use admin
switched to db admin
> db.createUser({"user":"datadog", "pwd": "er1RKRdSi10Xoq0Mac64xhAu", "roles" : [ {role: 'read', db: 'admin' }, {role: 'clusterMonitor', db: 'admin'}, {role: 'read', db: 'local' }]})
Successfully added user: {
	"user" : "datadog",
	"roles" : [
		{
			"role" : "read",
			"db" : "admin"
		},
		{
			"role" : "clusterMonitor",
			"db" : "admin"
		},
		{
			"role" : "read",
			"db" : "local"
		}
	]
}
```
To confirm that the user has been correctly created, we run the following:
```
$ echo "db.auth('datadog', 'LJjrd2A9Sdf5LVodMIUmabHe')" | mongo admin | grep -E "(Authentication failed)|(auth fails)" &&
> echo -e "\033[0;31mdatadog user - Missing\033[0m" || echo -e "\033[0;32mdatadog user - OK\033[0m"
datadog user - OK
```
Now we edit the **mongodb.yaml** file to instruct the agent to integrate with MongoDB:

```
instances:
  - server: mongodb://datadog:er1RKRdSi10Xoq0Mac64xhAu@localhost:27017

  tags:
    - database:mongodb
    - project:technical_test
```
And restart the agent

```
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

Now we can check that the integration is running correctly

```
sudo datadog-agent status
```
<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Mongo Check.png?raw=true">

Datadog will provide out of the box [dashboard for MongoDB](https://app.datadoghq.com/screen/integration/13/MongoDB%20-%20Overview?tpl_var_scope=host%3Avagrant&page=0&is_auto=false&from_ts=1543681380000&to_ts=1543684980000&live=true)

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Mongo Dashboard.png?raw=true">

* Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

Datadog has integrations for most of the commercial software platforms in the market, but sometimes it is needed to monitor something that is so unique or home grown that needs to build a custom monitoring. Datadog provides the flexibility to be extended with custom checks to create new metrics.

We need to create a configuration file for the custom check, in this case we will call it **myrandom.yaml**  

```
init_config:

instances: [{}]
```

We need also to create the script that will run to provide the custom metric **myrandom.py**
```python
import random

from datadog_checks.checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"


class CustomCheck(AgentCheck):
    def check(self, instance):
        self.gauge('custom.mycheck', random.randint(0,1000))

```

Now, let's confirm that the custom metric is correctly running at the agent

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Custom Check Agent.png?raw=true">

* Change your check's collection interval so that it only submits the metric once every 45 seconds.

It is possible to adjust the collection interval for checks, just by changing the configuration file, in this case **myrandom.yaml**
```
init_config:

instances:
  - min_collection_interval: 45
```

Automatically we will have the custom check available for the agent, as we can se at the [Host Map](https://app.datadoghq.com/infrastructure/map?fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&groupby=availability-zone&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=green_to_orange&paletteflip=false&node_type=host)

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Host Map with Custom.png?raw=true">

* **Bonus Question** Can you change the collection interval without modifying the Python check file you created?

To change the collection interval there is no need to modify the script created, just need to modify the configuration file.

# Visualizing Data:

Utilize the Datadog API to create a Timeboard that contains:

* Your custom metric scoped over your host.
* Any metric from the Integration on your Database with the anomaly function applied.
* Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

Datadog is an extremely flexible and open monitoring platform, it provides an API to easily integrate with any other external system and control programmaticaly all functionality included into the platform.

We are going to use that API to create a new Timeboard, we will use a python script that call the API **timeseries.py**
```python
from datadog import initialize, api

options = {
    'api_key': 'c7c0572c87dc9c1295865e5fb4246307',
    'app_key': 'b0ce75a211360b93300465f78abfc8ce82443c5d'
}

initialize(**options)

title = "Vicente's Dashboard"
description = "Dashboard created through the API"
graphs = [
{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:custom.mycheck{*}"}
        ],
        "viz": "timeseries"
    },
    "title": "Random Number"
},
{ 
    "definition": {
        "events": [],
        "requests": [
            {"q": "anomalies(avg:mongodb.uptime{*}, 'basic', 2)"}
        ],
        "viz": "timeseries"
    },
    "title": "Anomalies for MongoDB"
},
{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:custom.mycheck{*}.rollup(sum, 3600)"}
        ],
        "viz": "timeseries"
    },
    "title": "Random number 1 hour rollup"
}]

template_variables = [{
    "name": "host1",
    "prefix": "host",
    "default": "host:my-host"
}]

read_only = True
api.Timeboard.create(title=title,
                     description=description,
                     graphs=graphs,
                     template_variables=template_variables,
                     read_only=read_only)

```
Now we can access our [Dashboard](https://app.datadoghq.com/dash/1006937/vicentes-dashboard?live=true&page=0&is_auto=false&from_ts=1543678463245&to_ts=1543692863245&tile_size=m&tpl_var_host1=vagrant) from the Dashboard list:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Custom Dashboard.png?raw=true">

* Set the Timeboard's timeframe to the past 5 minutes

It is possible to analyze in detail the metrics included into a dashboard by zooming in, selecting a shot period of time in one of the graphs:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Custom Dashboard 5min.png?raw=true">

* Take a snapshot of this graph and use the @ notation to send it to yourself.

Datadog provides collaboration functions to make easy share metrics and analytics with your team to improve teamwork and time to resolution.

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Graph Snapshot.png?raw=true">

* **Bonus Question**: What is the Anomaly graph displaying?

Datadog provides anomaly detection funtionality by comparing past data with current data to identify when metric is deviating from its normal behaviour. That functionality allows to prevent failures by anticipating when performance is degrading. 

# Monitoring Data

Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

* Warning threshold of 500
* Alerting threshold of 800
* And also ensure that it will notify you if there is No Data for this query over the past 10m.

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Alert Definition.png?raw=true">

Please configure the monitor’s message so that it will:

* Send you an email whenever the monitor triggers.
* Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
* Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

The message to be sent when a monitor triggers an alert can be customized using variables and team members can be added to receive the message, we will use this conditional message when the monitor exceeds thresholds:

```
{{#is_alert}}
## Alert!
Random metric has exceeded threshold. Value = {{value}} 
IP Address = {{host.ip}} 
{{/is_alert}}

{{#is_warning}}
## Warning
Random metric increasing value 
{{/is_warning}}

{{#is_no_data_recovery}} There is no data for last 10 minutes {{/is_no_data_recovery}}

@vlorente68@gmail.com
```
* When this monitor sends you an email notification, take a screenshot of the email that it sends you.

Here we have the messages sent by email when the alert is sent:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Warning Email.png?raw=true">
<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/No Data Email.png?raw=true">

* **Bonus Question**: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

Datadog provides a very easy to use and flexible interface to schedule Downtime and silence alerts

  * One that silences it from 7pm to 9am daily on M-F,

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Schedule Downtime 1.png?raw=true">
  
  * And one that silences it all day on Sat-Sun.
  
<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Schedule Downtime 2.png?raw=true">

  * Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.

A message can be sent when the downtime start or finish:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Downtime message.png?raw=true">

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/Downtime Email.png?raw=true">

# Collecting APM Data:

Given the following Flask app (or any Python/Ruby/Go app of your choice) instrument this using Datadog’s APM solution:

```python
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')
```

The first thing we have to do is to configure the Datadog agent to collect APM metrics and traces by modifiying the **datadog.yaml**
```
# Trace Agent Specific Settings
#
apm_config:
#   Whether or not the APM Agent should run
  enabled: true
#   The environment tag that Traces should be tagged with
#   Will inherit from "env" tag if none is applied here
  env: none
#   The port that the Receiver should listen on
#   receiver_port: 8126
#   Whether the Trace Agent should listen for non local traffic
#   Only enable if Traces are being sent to this Agent from another host/container
#   apm_non_local_traffic: false
#   Extra global sample rate to apply on all the traces
#   This sample rate is combined to the sample rate from the sampler logic, still promoting interesting traces
#   From 1 (no extra rate) to 0 (don't sample at all)
#   extra_sample_rate: 1.0
#   Maximum number of traces per second to sample.
#   The limit is applied over an average over a few minutes ; much bigger spikes are possible.
#   Set to 0 to disable the limit.
#   max_traces_per_second: 10
#   A blacklist of regular expressions can be provided to disable certain traces based on their resource name
#   all entries must be surrounded by double quotes and separated by commas
#   Example: ["(GET|POST) /healthcheck", "GET /V1"]
#   ignore_resources: []
```

To instrument the application, we will use in this case the manual mode by modifiying the application **myApp.py**, adding the first two lines:

```python
from ddtrace import patch_all
patch_all()

from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'
```

After some requests we can see the traces and the APM metrics in the [APM Dashboards](https://app.datadoghq.com/apm/service/flask/flask.request?env=environment&start=1543710832672&end=1543714432672&paused=false) that come out of the box with Datadog:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/APM Dashboard.png?raw=true">

* **Bonus Question**: What is the difference between a Service and a Resource?

Datadog APM differenciates between Services and Resources, a service is a more global concept and is a set of processes that provide a function, for example a database, app server, web framework, etc. A resource is an specific entry point or query that belongs to a service. We can say that the resource is more granular than a service.

Provide a link and a screenshot of a Dashboard with both APM and Infrastructure Metrics.

Please include your fully instrumented app in your submission, as well.

We can edit the [Dashboard](https://app.datadoghq.com/dash/1006937/vicentes-dashboard?live=true&page=0&is_auto=false&from_ts=1543700487308&to_ts=1543714887308&tile_size=m) created before to innclude APM metrics beside the infrastructure metrics:

<img src="https://github.com/vlorente68/hiring-engineers/blob/master/screenshots/APM and Infrastructure.png?raw=true">

# Final Question:

Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability!

Is there anything creative you would use Datadog for?

As soon as you can access to data series, even better if they are multi dimensional, you can make use of Datadog to gain insight and alert on specific conditions. Here we have some examples:

* **Smart Cities:** Most of the big cities in the world are making restriction to traffic. For example, in Madrid traffic is restricted for private cars, except if you are going to a public parking. We could use the API provided by the local government to check real time status of parking utilization and could decide if going to the city center by car or public transportation. Even, it could be used as a public service for all citizens.
* **Marketing Campaigns** Already exists software that is able to track marketing campaigns, but it could be interesting to put in the same dashboard the information coming from marketing about hits with the information from sales opportunities and from finance. Datadog could be used to gather data from many different systems and present that business data into a single dashboard.
* **Personal Finance** The European Union launched the PSD2 directive, so all banks have to provide a public API with information about the bank accounts, balances, etc. We can connect to those APIs and get the balance of our accounts, that information can be published in a dashboard to keep track of personal finances including all different banks. Also, We can connect to any of the multiple data streams for the stock market and monitor the evolution of the stock portfolio, add that information to the personal finance dashboard and even get alerts when thresholds are reached to buy or sell stocks.

Those are some uses cases, but as soon as you have access to data you can use Datadog advanced analytics and visualization to start getting more value out of that data. 
