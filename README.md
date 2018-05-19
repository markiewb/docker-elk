# Post-mortem log file analysis with Docker ELK stack

The aim of this project is to use the ELK stack to do provide you tooling for analysing the log files in an offline-manner. 

## Use-case

After a crash you have been sent MBs of log files from a customer. At the customer side you don't have a log analyse tool or you aren't allowed to access it. So you are stuck with the plain log files. 

## How does it work?

LogStash parses the logfiles using a Grok filter and sends the entries to the ElasticSearch instance.
Thus you can create queries, visualisations and dashboards in Kibana to visualise the log file content. 

``` puml
hide footbox
Actor You 
Participant LogStash
Participant ElasticSearch
Participant Kibana


==Parsing==
LogStash -> LogStash : reads the log files
LogStash -> LogStash : parses the log files using 'Grok'
LogStash -> ElasticSearch : persists parsed logs

==Analysing==
You -> Kibana : work with 
Kibana -> ElasticSearch : reads entries
Kibana <- ElasticSearch : 
Kibana -> You : provides visualisations/dashboards

```


## Quickstart

1. Checkout the project
2. Place your log files at `./example-logs` (an example is provided)
3. Fine tune the  grok-pattern in  `./logstash/pipeline/logstash.conf` (an example for the example-log is provided)
4. Invoke `docker-compose build`
5. Invoke `docker-compose up -d` to start the Docker containers
6. Open Kibana at `http://localhost:5601`
7. Create a Kibana index manually at `Management/Index Patterns/`
8. Analyse your data 


## TODOs
* support multline-stacktraces in the logfiles
  * https://discuss.elastic.co/t/using-logstash-to-analyse-log4j-log-files/44914
  * https://blog.lanyonm.org/articles/2015/12/29/log-aggregation-log4j-spring-logstash.html
  * https://stackoverflow.com/questions/37931563/what-should-be-the-logstash-grok-filter-for-this-log4j-log
  * https://sematext.com/blog/handling-stack-traces-with-logstash/
  * https://stackoverflow.com/questions/19856316/logstash-multiline-filter-configuration-and-java-exceptions

* create the Kibana-index automatically - [see](#via-the-kibana-web-ui)
* support multiple patterns using [Logstash-multiple pipelines](https://www.elastic.co/blog/logstash-multiple-pipelines) and [type filters](https://sematext.com/blog/getting-started-with-logstash/)
* provide some default filters and visualisations for Kibana
* allow to persist user-created filters/visualisations

## Further resources

### Grok-Pattern
* [Grok filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)
* [Default patterns for grok](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns)
* Grok debugger: [Grok constructor](grokconstructor.appspot.com), [Grok debugger](http://grokdebug.herokuapp.com)



## Troubleshooting
* if an entry has the `_grokparsefailure` tag the pattern did not match correctly. Use grok debuggers to check your pattern! 
* or have a look at the log files of logstash `docker logs dockerelk_logstash_1 -f` or
`docker exec -t -i dockerelk_logstash_1 /bin/bash`


## Trademark

This setup of ELK 6.2.3 is based on https://github.com/deviantony/docker-elk/


# Docker ELK stack

[![Join the chat at https://gitter.im/deviantony/docker-elk](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/deviantony/docker-elk?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Elastic Stack version](https://img.shields.io/badge/ELK-6.2.4-blue.svg?style=flat)](https://github.com/deviantony/docker-elk/issues/266)
[![Build Status](https://api.travis-ci.org/deviantony/docker-elk.svg?branch=master)](https://travis-ci.org/deviantony/docker-elk)

Run the latest version of the ELK (Elasticsearch, Logstash, Kibana) stack with Docker and Docker Compose.

It will give you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch
and the visualization power of Kibana.

Based on the official Docker images:

* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [logstash](https://github.com/elastic/logstash-docker)
* [kibana](https://github.com/elastic/kibana-docker)

**Note**: Other branches in this project are available:

* ELK 6 with X-Pack support: https://github.com/deviantony/docker-elk/tree/x-pack
* ELK 6 in Vagrant: https://github.com/deviantony/docker-elk/tree/vagrant
* ELK 6 with Search Guard: https://github.com/deviantony/docker-elk/tree/searchguard

## Contents

1. [Requirements](#requirements)
   * [Host setup](#host-setup)
   * [SELinux](#selinux)
   * [DockerForWindows](#dockerforwindows)
2. [Getting started](#getting-started)
   * [Bringing up the stack](#bringing-up-the-stack)
   * [Initial setup](#initial-setup)
3. [Configuration](#configuration)
   * [How can I tune the Kibana configuration?](#how-can-i-tune-the-kibana-configuration)
   * [How can I tune the Logstash configuration?](#how-can-i-tune-the-logstash-configuration)
   * [How can I tune the Elasticsearch configuration?](#how-can-i-tune-the-elasticsearch-configuration)
   * [How can I scale out the Elasticsearch cluster?](#how-can-i-scale-up-the-elasticsearch-cluster)
4. [Storage](#storage)
   * [How can I persist Elasticsearch data?](#how-can-i-persist-elasticsearch-data)
5. [Extensibility](#extensibility)
   * [How can I add plugins?](#how-can-i-add-plugins)
   * [How can I enable the provided extensions?](#how-can-i-enable-the-provided-extensions)
6. [JVM tuning](#jvm-tuning)
   * [How can I specify the amount of memory used by a service?](#how-can-i-specify-the-amount-of-memory-used-by-a-service)
   * [How can I enable a remote JMX connection to a service?](#how-can-i-enable-a-remote-jmx-connection-to-a-service)

## Requirements

### Host setup

1. Install [Docker](https://www.docker.com/community-edition#/download) version **1.10.0+**
2. Install [Docker Compose](https://docs.docker.com/compose/install/) version **1.6.0+**
3. Clone this repository

### SELinux

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux
into Permissive mode in order for docker-elk to start properly. For example on Redhat and CentOS, the following will
apply the proper context:

```console
$ chcon -R system_u:object_r:admin_home_t:s0 docker-elk/
```

### DockerForWindows

If you're using Docker for Windows, ensure the 'Shared Drives' feature is enabled for the C: drive (Docker for Windows > Settings > Shared Drives). [MSDN article detailing Shared Drives config](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/).

## Usage

### Bringing up the stack

**Note**: In case you switched branch or updated a base image - you may need to run `docker-compose build` first

Start the ELK stack using `docker-compose`:

```console
$ docker-compose up
```

You can also choose to run it in background (detached mode):

```console
$ docker-compose up -d
```

Give Kibana a few seconds to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser.

By default, the stack exposes the following ports:
* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

**WARNING**: If you're using `boot2docker`, you must access it via the `boot2docker` IP address instead of `localhost`.

**WARNING**: If you're using *Docker Toolbox*, you must access it via the `docker-machine` IP address instead of
`localhost`.

Now that the stack is running, you will want to inject some log entries. The shipped Logstash configuration allows you
to send content via TCP:

```console
$ nc localhost 5000 < /path/to/logfile.log
```

## Initial setup

### Default Kibana index pattern creation

When Kibana launches for the first time, it is not configured with any index pattern.

#### Via the Kibana web UI

**NOTE**: You need to inject data into Logstash before being able to configure a Logstash index pattern via the Kibana web
UI. Then all you have to do is hit the *Create* button.

Refer to [Connect Kibana with
Elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html) for detailed instructions
about the index pattern configuration.

#### On the command line

Create an index pattern via the Kibana API:

```console
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 6.2.4' \
    -d '{"attributes":{"title":"logstash-*","timeFieldName":"@timestamp"}}'
```

The created pattern will automatically be marked as the default index pattern as soon as the Kibana UI is opened for the first time.

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

It is also possible to map the entire `config` directory instead of a single file.

### How can I tune the Logstash configuration?

The Logstash configuration is stored in `logstash/config/logstash.yml`.

It is also possible to map the entire `config` directory instead of a single file, however you must be aware that
Logstash will be expecting a
[`log4j2.properties`](https://github.com/elastic/logstash-docker/tree/master/build/logstash/config) file for its own
logging.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override directly via environment variables:

```yml
elasticsearch:

  environment:
    network.host: "_non_loopback_"
    cluster.name: "my-cluster"
```

### How can I scale out the Elasticsearch cluster?

Follow the instructions from the Wiki: [Scaling out
Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

## Storage

### How can I persist Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on
your Docker host. Update the `elasticsearch` service declaration to:

```yml
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

This will store Elasticsearch data inside `/path/to/storage`.

**NOTE:** beware of these OS-specific considerations:
* **Linux:** the [unprivileged `elasticsearch` user][esuser] is used within the Elasticsearch image, therefore the
  mounted data directory must be owned by the uid `1000`.
* **macOS:** the default Docker for Mac configuration allows mounting files from `/Users/`, `/Volumes/`, `/private/`,
  and `/tmp` exclusively. Follow the instructions from the [documentation][macmounts] to add more locations.

[esuser]: https://github.com/elastic/elasticsearch-docker/blob/016bcc9db1dd97ecd0ff60c1290e7fa9142f8ddd/templates/Dockerfile.j2#L22
[macmounts]: https://docs.docker.com/docker-for-mac/osxfs/

## Extensibility

### How can I add plugins?

To add plugins to any ELK component you have to:

1. Add a `RUN` statement to the corresponding `Dockerfile` (eg. `RUN logstash-plugin install logstash-filter-json`)
2. Add the associated plugin code configuration to the service configuration (eg. Logstash input/output)
3. Rebuild the images using the `docker-compose build` command

### How can I enable the provided extensions?

A few extensions are available inside the [`extensions`](extensions) directory. These extensions provide features which
are not part of the standard Elastic stack, but can be used to enrich it with extra integrations.

The documentation for these extensions is provided inside each individual subdirectory, on a per-extension basis. Some
of them require manual changes to the default ELK configuration.

## JVM tuning

### How can I specify the amount of memory used by a service?

By default, both Elasticsearch and Logstash start with [1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) allocated to
the JVM Heap Size.

The startup scripts for Elasticsearch and Logstash can append extra JVM options from the value of an environment
variable, allowing the user to adjust the amount of memory that can be used by each component:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |
| Logstash      | LS_JAVA_OPTS         |

To accomodate environments where memory is scarce (Docker for Mac has only 2 GB available by default), the Heap Size
allocation is capped by default to 256MB per service in the `docker-compose.yml` file. If you want to override the
default JVM configuration, edit the matching environment variable(s) in the `docker-compose.yml` file.

For example, to increase the maximum JVM Heap Size for Logstash:

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Xmx1g -Xms1g"
```

### How can I enable a remote JMX connection to a service?

As for the Java Heap memory (see above), you can specify JVM options to enable JMX and map the JMX port on the docker
host.

Update the `{ES,LS}_JAVA_OPTS` environment variable with the following content (I've mapped the JMX service on the port
18080, you can change that). Do not forget to update the `-Djava.rmi.server.hostname` option with the IP address of your
Docker host (replace **DOCKER_HOST_IP**):

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=18080 -Dcom.sun.management.jmxremote.rmi.port=18080 -Djava.rmi.server.hostname=DOCKER_HOST_IP -Dcom.sun.management.jmxremote.local.only=false"
```
