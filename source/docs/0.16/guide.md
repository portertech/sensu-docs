---
version: "0.16"
category: "Installation"
title: "Guide"
next:
  url: adding_a_check
  text: "Adding a check"
---

# TODO

- Guide overview - explain what the installation guide is and isn't
  - Can't be too opinionated
  - Installation guide is an introduction to Sensu; deploying to production is a separate process

- Installation objectives - is there more than one?
  - Install dependencies & config
  - Install sensu services (sensu-server, etc) & config
  - Install client(s) & config
  - Next steps: how to deploy Sensu in production
  - How to secure your deployment (i.e. SSL)
  - Address scaling strategies (don't actually document how to scale)

- Installation guide / flow
  - /guide => /installation-overview

  - /install-rabbitmq
    - install rabbitmq
    - configure rabbitmq (link to /rabbit reference docs?)
    - start rabbitmq

  - /install-redis
    - install redis
    - configure redis (link to /redis reference docs?)
    - start redis

  - /install-respositories
    - Sensu Core
    - Sensu Enterprise (if appropriate)
    - apt-get update || yum update

  - /install-sensu
    - Sensu Core (server + api, etc)
    - Sensu Enterprise
    - configure sensu
      - add an example check (provide brief explanation of checks, link to /checks)
    - start the sensu services
    - observe sensu-server activity (e.g. logs)

  - /install-sensu-client
    - adding a machine
    - configure sensu-client
    - start sensu-client
    - observe sensu-client activity (e.g. logs)

  - /install-a-dashboard
    - explain that a dashboard is an optional component
    - installing Uchiwa or Sensu Enterprise Dashboard
    - refer to uchiwa docs for configuration
    - configure Sensu Enterprise Dashboard (if appropriate)
    - start sensu-enterprise-dashboard

  - /installation-summary
    - what have we accomplished?
    - how monitor actual applications and services
      - overview to explain the basic concepts of how to build comprehensive monitoring solution
    - deploying sensu in production (i.e. is different than the installation guide)
    - how to secure your deployment
    - scaling strategies

  - /getting-started
    - assumes sensu installation, shows how to start monitoring actual things

    - overview of sensu primitives / components (w/ 1-3 sentencese explaining what each of these things are)
      - checks
        - introduction to check results & events
          - checks generate results
          - some check results generate events
      - filters
      - mutators
      - handlers

  - /getting-started-with-checks
    - what it is yo!
    - basic explanation of check plugins & dependencies
    - example check plugin
      - example code
      - how/where to install
    - example check definition
      - example config
      - how/where to install
    - observe check results

  - /getting-started-with-handlers
    - what it is yo!
      - hanlders by themselves offer basic funtionality;
      - complex handler functionality is possible with filters & mutators (next)
    - example handler plugin
      - example code
      - how/where to install
    - example handler definition
      - example config
      - how/where to install
    - observe handler results

  - /getting-started-with-filters
    - what it is yo!
      - not mandatory; depends on handlers
      - can be inclusive/exclusive
    - example filter definitions
      - example inclusive filter config
      - example exclusive filter config
      - how/where to install
      - how to apply a filter (new/updated handler example)
    - observe filter results

  - /getting-started-with-mutators
    - what it is yo!
    - example mutator plugin
      - example code
      - how/where to install
    - example mutator definition
      - example config
      - how/where to install
      - how to apply a mutator (new/updated handler example)
    - observe mutator results


# Guide {#guide}

This guide provides instructions on how to manually install and
configure Sensu (and/or Sensu Enterprise) and its dependencies on Linux.

Sensu is typically (and best!) deployed by a configuration management
tool, such as Chef and Puppet.

* [Chef cookbook](https://github.com/sensu/sensu-chef)
* [Puppet module](https://github.com/sensu/sensu-puppet)

## Introduction {#introduction}

We will use two systems, one will run all of the Sensu components, and
the other will only run the Sensu client. You may choose to only
configure the system running all of the components, since it does run
a Sensu client. We'll name our systems "monitor" and "agent".

#### Monitor {#monitor}

- RabbitMQ
- Redis
- Sensu server
- Sensu client
- Sensu API

#### Agent {#agent}

- Sensu client

### Install Sensu server dependencies {#install-sensu-server-dependencies}

On the "monitor" system, generate SSL certificates for Sensu, using
the following instructions.

- [Generating SSL certificates](certificates)

On the "monitor" system, install RabbitMQ and Redis, using the
following instructions.

- [Installing RabbitMQ](rabbitmq)
- [Installing Redis](redis)

### Install Sensu {#install-sensu}

On both systems, install Sensu using the following instructions.

- [Installing Sensu](packages)


### Configure Sensu connections {#configure-sensu-connections}

Both systems run Sensu components that need to talk to RabbitMQ, so
we'll provide Sensu with connection information.

Earlier, you generated SSL certificates for Sensu on "monitor", and
used them when configuring RabbitMQ. The Sensu components will use the
client certificate that was generated.

On both systems, create an SSL directory for the Sensu components.

~~~ shell
mkdir -p /etc/sensu/ssl
~~~

Copy the following generated SSL files to the newly created SSL
directories.

* `client/cert.pem`
* `client/key.pem`

On both systems, create/edit `/etc/sensu/conf.d/rabbitmq.json`, you
will need to substitute a few values.

~~~ json
{
  "rabbitmq": {
    "ssl": {
      "cert_chain_file": "/etc/sensu/ssl/cert.pem",
      "private_key_file": "/etc/sensu/ssl/key.pem"
    },
    "host": "SUBSTITUTE_ME",
    "port": 5671,
    "vhost": "/sensu",
    "user": "sensu",
    "password": "SUBSTITUTE_ME"
  }
}
~~~

The "monitor" system runs the Sensu server and API, which need to talk
to Redis, so we'll provide Sensu with connection information.

On the "monitor" system, create/edit `/etc/sensu/conf.d/redis.json`.

~~~ json
{
  "redis": {
    "host": "localhost",
    "port": 6379
  }
}
~~~

### Configure the Sensu API {#configure-the-sensu-api}

The "monitor" system runs the Sensu API, so we'll need to configure
them.

On the "monitor" system, create/edit `/etc/sensu/conf.d/api.json`.

~~~ json
{
  "api": {
    "host": "localhost",
    "port": 4567,
    "user": "admin",
    "password": "secret"
  }
}
~~~

### Configure the Sensu clients {#configure-the-sensu-clients}

On both systems, configure the Sensu client, providing them with
information about themselves.

The Sensu client name is commonly the system's hostname, or another
unique identifier (such as a VM ID).

On both systems, create/edit `/etc/sensu/conf.d/client.json`.

~~~ json
{
  "client": {
    "name": "SUBSTITUTE_ME",
    "address": "SUBSTITUTE_ME",
    "subscriptions": [ "all" ]
  }
}
~~~

### Enable Sensu services {#enable-sensu-services}

The Sensu packages install sysvinit (init.d) scripts directly to
`/etc/init.d/`. All services are disabled by default.

Alternative supervisor scripts (such as upstart) are available in
`/usr/share/sensu` for those that may want them.

#### "Monitor" system {#enable-monitor-system-services}

On the "monitor" system, enable all of the Sensu components.

##### Debian and Ubuntu {#monitor-system-services-debian-and-ubuntu}

~~~ shell
update-rc.d sensu-server defaults
update-rc.d sensu-client defaults
update-rc.d sensu-api defaults
~~~

##### CentOS (RHEL) {#monitor-system-services-centos-and-rhel}

~~~ shell
chkconfig sensu-server on
chkconfig sensu-client on
chkconfig sensu-api on
~~~

#### "Agent" system {#enable-agent-system-services}

On the "agent" system, enable the Sensu client.

##### Debian and Ubuntu {#agent-system-services-debian-and-ubuntu}

~~~ shell
update-rc.d sensu-client defaults
~~~

##### CentOS (RHEL) {#agent-system-services-centos-and-rhel}

~~~ shell
chkconfig sensu-client on
~~~

### Start Sensu services {#start-sensu-services}

#### "Monitor" system {#start-monitor-system-services}

On the "monitor" system, start all of the Sensu components.

~~~ shell
/etc/init.d/sensu-server start
/etc/init.d/sensu-client start
/etc/init.d/sensu-api start
~~~

#### "Agent" system {#start-agent-system-services}

On the "agent" system, start the Sensu client.

~~~ shell
/etc/init.d/sensu-client start
~~~

## Next Steps {#next-steps}

Now that you have a running Sensu installation, the next steps are to
add monitoring checks and event handlers.

You may also want to [install a Sensu dashboard](install_a_dashboard)!

If you have further questions please visit the `#sensu` IRC channel on
Freenode or send an email to the `sensu-users` mailing list.
