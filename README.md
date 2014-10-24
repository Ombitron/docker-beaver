docker-beaver
=============

*A Docker image for shipping log to a logstash server*

How  It Works
-----------------
This image uses [python-beaver](https://github.com/josegonzalez/python-beaver) to ship local log files to a central Logstash server.

The image has one mountable volume `/app/logs` that can be used to provided access to the local log folder (such as `/var/logs`).  By default this will monitor changes to all `.log` files in `/app/logs` recursively and ship the events out to standard out.

Configuring the Beaver
------------------------------

There are three configuration parameters:

 1. The `/app/logs` volume for providing local logs to the system
 2. The environment variable `$BEAVERCONF_URL`. This provides a way to load a new beaver configuration file to the image.  For testing you can create a github gist and provide the raw file url.  A beaver configuration for sending syslogs out could look like:
```
[beaver]
logstash_version: 1
udp_host: 127.0.0.1
udp_port: 9000
 
[/app/log/**/*.log]
type: syslog
tags: sys,main 
```
3. The environment variable `$BEAVER_OPTS`.  This is where all beaver options are given besides the configuration file which is already set.  To send out the logs from the configuration listed above the string `'-t udp'` would be supplied.

Running this configuration would then become:
```
$ docker run -t -d -v /var/logs:/app/logs \
   -e BEAVERCONF_URL=https://gist.githubusercontent.com/btashton/0e43c1d5824915c57335/raw/f5d2526a7be2f5a7604055745d27f004068e7b4d/beaver.conf \
   -e BEAVER_OPTS='-t udp' \
   --net host \
   btashton/docker-beaver
```

*Note:* `--net host` is being used in this example because the Logstash server is at localhost.

Extended Example (run a local Logstash server)
----------------------------------------------------------------
This example will monitor your local system log files, send them to logstash, and let you view them at
the [prebuilt logstash dashboard](http://127.0.0.1:9292/index.html#/dashboard/file/logstash.json).
You can do a lot more with [docker-logstash](https://github.com/pblittle/docker-logstash)
```
$ docker run -d \
  -p 9292:9292 \
  -p 9200:9200 \
  -p 9000:9000 \
  -e LOGSTASH_CONFIG_URL=https://gist.githubusercontent.com/btashton/eb36d37e6cfc9800a63e/raw/logstash.conf \
  btashton/docker-logstash
 
$ docker run -t -d -v /var/logs:/app/logs \
   -e BEAVERCONF_URL=https://gist.githubusercontent.com/btashton/0e43c1d5824915c57335/raw/f5d2526a7be2f5a7604055745d27f004068e7b4d/beaver.conf \
   -e BEAVER_OPTS='-t udp' \
   --net host \
   btashton/docker-beaver
```
