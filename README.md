newrelic_plugin_agent
=====================

An agent that polls supported backend systems and submits the results to the
NewRelic platform. Currently supported backend systems are:

- Apache HTTP Server
- CouchDB
- Edgecast CDN "Realtime" API
- Memcached
- MongoDB
- Nginx
- pgBouncer
- PostgreSQL
- RabbitMQ
- Redis
- Riak

Base Requirements
-----------------
The agent requires Python 2.6 or 2.7 and pip for installation. Individual plugin backends may require additional libraries and are detailed below.

Installation Instructions
-------------------------
1. Install via pip*:

    pip install newrelic-plugin-agent

* See pip installation instructions at http://www.pip-installer.org/en/latest/installing.html

2. Copy the configuration file example from /opt/newrelic_plugin_agent/etc/newrelic/newrelic_plugin_agent.cfg to /etc/newrelic/newrelic_plugin_agent.cfg and edit the configuration in that file.

3. If you use the default "newrelic" user from the sample configuration, ensure that the user exists and that the /var/run/newrelic and /var/log/newrelic directories are writable or owned by that user.

4. Run the app:

    newrelic_plugin_agent -c PATH-TO-CONF-FILE [-f]

Where -f is to run it in the foreground instead of as a daemon.

Sample configuration and init.d scripts are installed in /opt/newrelic_plugin_agent

Installing Additional Requirements
----------------------------------

To use the MongoDB, pgBouncer or PostgreSQL plugin you must install the psycopg2 library. To easily do
this, make sure you have the latest version of pip installed (http://www.pip-installer.org/). This should be done after installing the agent itself.

Once installed, from inside the source directory run the following command:

    pip install -e .[mongodb]

or

    pip install -e .[pgbouncer]

or

    pip install -e .[postgresql]

Apache HTTPd Installation Nodes
-------------------------------
Enable the HTTPd server status page in the default virtual host. The following example configuration snippet for Apache HTTPd demonstrates how to do this:

    <Location /server-status>
        SetHandler server-status
        Order deny,allow
        Deny from all
        Allow from all
    </Location>

The agent requires the exteneded information to parse metrics.  If you are not seeing any metrics on your graphs for Apache verify that you have enable ExtendedStatus, the default is off so you must enable it.  In your global Apache HTTP configuration you need to enable exetended status using:

    ExtendedStatus On

MongoDB Installation Nodes
-------------------------
You need to install the pymongo driver, either by running "pip install pymongo" or by following the "Installing Additional Requirements" above. Each database you wish to collect metrics for must be enumerated in the configuration.

To use authentication, include the username and password configuration values, otherwise omit them.

Nginx Installation Notes
------------------------
Enable the nginx stub_status setting on the default site in your configuration. The following example configuration snippet for Nginx demonstates how to do this:

      location /nginx_stub_status {
        stub_status on;
      }

pgBouncer Installation Notes
----------------------------
The user specified must be a stats user.

PostgreSQL Installation Notes
-----------------------------
By default user specified must superuser to get PostgreSQL directory listings. If you want to
use this plugin without need of superuser permissions use `superuser: False` setting in
configuration file e.g.:

    postgresql:
      host: localhost
      port: 5432
      user: newrelic
      dbname: postgres
      password: newrelic
      superuser: False


RabbitMQ Installation Notes
---------------------------
The user specified must have access to all virtual hosts you wish to monitor and should have either the Administrator tag or the Monitor tag.

Redis Installation Notes
-----------------------------
For Redis daemons that are password protected, add the password configuration value, otherwise omit it. The Redis configuration section allows for multiple redis servers. The syntax to poll multiple servers is in the example below.

Configuration Example
---------------------

    %YAML 1.2
    ---
    Application:
      license_key: REPLACE_WITH_REAL_KEY
      poll_interval: 60
      #proxy: http://localhost:8080

      apache_httpd:
        name: hostname
        host: localhost
        port: 80
        path: /server-status

      couchdb:
        name: localhost
        host: localhost
        port: 5984

      edgecast:
        name: My Edgecase Account
        account: ACCOUNT_NUMBER
        token: API_TOKEN

      mongodb:
        name: hostname
        host: localhost
        port: 27017
        username: foo
        password: bar
        databases:
          - database_name_1
          - database_name_2
          - etc

      memcached:
        name: localhost
        host: localhost
        port: 11211

      nginx:
        name: hostname
        host: localhost
        port: 80
        path: /nginx_stub_status

      pgbouncer:
        host: localhost
        port: 6000
        user: stats

      postgresql:
        host: localhost
        port: 5432
        user: postgres
        dbname: postgres
        superuser: True

      rabbitmq:
        name: rabbitmq@localhost
        host: localhost
        port: 15672
        username: guest
        password: guest

      redis:
        - name: localhost
          host: localhost
          port: 6379
          db_count: 16
          password: foobar
        - name: localhost
          host: localhost
          port: 6380
          db_count: 16
          password: foobar

      riak:
        name: localhost
        host: localhost
        port: 8098

    Daemon:
      user: newrelic
      pidfile: /var/run/newrelic/newrelic_plugin_agent.pid

    Logging:
      formatters:
        verbose:
          format: '%(levelname) -10s %(asctime)s %(process)-6d %(processName) -15s %(threadName)-10s %(name) -25s %(funcName) -25s L%(lineno)-6d: %(message)s'
      handlers:
        file:
          class : logging.handlers.RotatingFileHandler
          formatter: verbose
          filename: /var/log/newrelic/newrelic_plugin_agent.log
          maxBytes: 10485760
          backupCount: 3
      loggers:
        newrelic_plugin_agent:
          level: INFO
          propagate: True
          handlers: [console, file]
        requests:
          level: ERROR
          propagate: True
          handlers: [console, file]

Troubleshooting
---------------
- If the installation does not install the "newrelic_plugin_agent" application in /usr/bin then it is likely that setuptools or distribute is not up to date. The following commands can be run to install distribute and pip for installing the application:

    curl http://python-distribute.org/distribute_setup.py | python
    curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python

- If the application installs but doesn't seem to be submitting status, check the logfile which at /tmp/newrelic_plugin_agent.log if the default example logging configuration is used.
- If the agent starts but dies shortly after ensure that /var/log/newrelic and /var/run/newrelic are writable by the same user specified in the daemon section of the configuration file.
- If the agent has died and won't restart, remove any files found in /var/run/newrelic/
- If using the Apache HTTP plugin and your stats are blank, ensure the ExtendedStatus directive is on.
