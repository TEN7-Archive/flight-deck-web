![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flight Deck Web

Flight Deck Web is a minimalist Apache/PHP container for Drupal and PHP-based sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* PHP optimized out of the box for Drupal sites, while supporting any PHP application.
* Drush, Drupal Console, Imagemagick, Node, SASS out of the box

## Tags and versions

There are several tags available for this container, each with different software and support:

| Tags | PHP version | Drupal versions | Drush |
| ---- | ----------- | --------------- | ----- |
| 7.4 | 7.4 | 8, 9 | 10.x |
| 7.4-drupal | 7.4 | 7 | 8.x |
| latest, 7, 7.3 | 7.3 | 8, 9 | 10.x |
| 7.3-drupal7 | 7.3 | 7 | 8.x |
| 7.2 | 7.2 | 8 | 9.x |
| 7.2-drupal7 | 7.2 | 7 | 8.x |
| 5.6 | 5.6 | 7, 6 | 8.x |

## Configuration

This container does not use environment variables for configuration. Instead, the `flight-deck-web.yml` file is used to handle all configuration.

```yaml
---
flightdeck_web: {}
```

All configuration is done as items under the `flightdeck_web` variable. See the following sections for details.

### Configuring virtual hosts

This container may have one or more virtual hosts configured to serve. Each is specified as an item under the `vhosts` key:

```yaml
---
flightdeck_web:
  vhosts:
    - name: "example.com"
      host: "*:80"
      docroot: "/var/www/html"
      aliases:
        - "docker.test"
        - "docker.localhost"
      env:
        - name: ANSWER_TO_EVERYTHING
          value: "42"
      https:  yes
      certFile: "/etc/ssl/apache2/server.pem"
      keyFile: "/etc/ssl/apache2/server.key"
      extraLines:
        - "SSLProxyEngine On"
```

Where:

* **name** is the fully-qualified domain name of the site. Required.
* **host** is the `hostname:port` on which to accept requests. Optional, defaults to `*:80`.
* **docroot** is the full path inside the container to the site docroot. Optional, defaults to `/var/www/html`.
* **aliases** is a list of domain aliases to treat as alternate domain names for the site. Optional.
* **env** is a list of environment variables to set when serving requests. Optional.
* **https** specifies that this vhost should serve requests through HTTPS. `yes` to enable, `no` to disable. Optional, defaults to `no`.
* **certFile** is the full path inside the container to the SSL certificate chain in `*.pem` format. Optional.
* **keyFile** is the full path inside the container to the SSL certificate private key. Optional.
* **extraLines** is a list of additional custom lines to add to the vhost. Optional.

### Configuring PHP

This container uses mod_php to provide PHP as part of the Apache process. While PHP is configured with sensible defaults out of the box, you may customize them by defining the `php` key:

```yaml
---
flightdeck_web:
  php:
    sendmail_path: "/usr/sbin/sendmail"
    max_execution_time: "120"
    max_input_vars: "3000"
    memory_limit: "{{ lookup('env', 'PHP_MAX_INPUT_VARS') | default('320M', true) }}"
    upload_max_filesize: "128M"
    post_max_size: "128M"
    error_reporting: "E_ALL & ~E_DEPRECATED & ~E_STRICT"
    display_errors: no
    display_startup_errors: no
    error_log: "/dev/stderr"
    variables_order: "GPCS"
```

Where:

* **sendmail_path** is the sendmail command to use when sending email. Optional, defaults the the sendmail binary included with the container.
* **max_execution_time** is the maximum time in seconds for PHP to serve a request. Optional, defaults to 120 seconds.
* **max_input_vars** specifies the maximum number of variables that may be submitted in a single request. Optional, defaults to 3000.
* **memory_limit** specifies the maximum amount of memory PHP may consume to serve a web request. Optional, defaults to `320M`.
* **upload_max_filesize** is the maximum file size that may be uploaded in a single request. Optional, defaults to `128M`.
* **post_max_size** is the maximum size a POST request may be and still be accepted. Optional, defaults to `128M`.
* **error_reporting** specifies the types or errors that may be displayed on screen. Optional, defaults to development values.
* **display_errors** specifies if errors should be written to the output of web requests. Optional, defaults to `no`.
* **display_startup_errors** specifies if startup errors should be written to the output of web requests. Optional, defaults to `no`.
* **error_log** is the full path inside the container to write the PHP error log. Optional, defaults to the docker log collector at `/dev/stderr`.
* **variables_order** specifies the order and types of variables to make available when serving web requests. Optional, defaults to production values.

### XDebug configuration

XDebug is included out of the box with Flight Deck Web. By default, it is **not** enabled. You can configure it using the `xdebug` key:

```yaml
---
flightdeck_web:
  xdebug:
    enabled: no
    autostart: no
    connectBack: no
    stdoutLogs: no
    idekey: "PHPSTORM"
    remoteHost: "10.254.254.254"
```

Where:

* **enabled** specifies if XDebug is enabled (`yes`), or disabled (`no`). Optional, default is `no`.
* **autostart** specifies if XDebug should always run on every request, even if the `idekey` isn't present. Optional, defaults to `no`.
* **connectBack* specifies if XDebug should attempt to connect back to a remote debugger. Optional, default is `no` as this is not functional on Docker for Mac or Windows.
* **stdoutLogs** specifies if XDebug's logs should be sent to STDOUT or not. Optional, default is `no` as this is only necessary when troubleshooting XDebug.
* **idekey** specifies the XDebug trigger value passed via a GET request to dynamically enable debugging on a per-request basis. Optional, defaults to `PHPSTORM`.
* **remoteHost** is the remote host at which to allow debugging. Optional, defaults to `10.254.254.254`.

### XDebug Profile configuration

The XDebug profiler is also included with the container out of the box for use in gathering PHP performance data for your application. You may configure it by using the `xdebugProfile` key:

```yaml
---
flightdeck_web:
  xdebugProfile:
    trigger: "XDEBUG_PROFILE"
    force: no
```

Where:

* **trigger** is the XDebug trigger value passed via a GET request to dynamically enable profilin gon a per-request basis. Optional, defaults to `XDEBUG_PROFILE`.
* **force** specifies if profiling should be run on every request (`yes`), or not (`no`). Optional, default is `no`.

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy the web container as a deployment. By default, it shares the same pod with [`ten7/flight-deck-varnish`](https://github.com/ten7/flight-deck-varnish):

```yaml
flightdeck_cluster:
  namespace: "example-com"
  configMaps:
    - name: "flight-deck-web"
      files:
        - name: "flight-deck-web.yml"
          content: |
            flightdeck_web:
  web:
    replicas: 1
    configMaps:
      - name: "flight-deck-web"
        path: "/config/web"
```

## Using with Docker Compose

Create the `flight-deck-web.yml` file relative to your `docker-compose.yml`. Define the `varnish` service mounting the file as a volume:

```yaml
version: '3'
services:
  varnish:
    image: ten7/flight-deck-web
    ports:
      - 80:80
      - 433:433
    volumes:
      - ./flight-deck-web.yml:/config/web/flight-deck-web.yml
```

### Using overrides to enable XDebug

XDebug creates a performance cost whenever enabled, even if the trigger value is not present in the request. For this reason, it's best to leave XDebug disabled unless you need to use the debugger. To do so, you can create a `docker-compose.overrides.yml` in the same directory as your `docker-compose.yml` file:

```yaml
version: '3'
services:
  web:
    environment:
      PHP_XDEBUG_ENABLED: 1
      PHP_XDEBUG_AUTOSTART: 1
      PHP_XDEBUG_PROFILE_TRIGGER: XDEBUG_PROFILE
```

It is highly recommended to add `docker-compose.overrides.yml` to your project's `.gitignore`.

## Extending

Often, you may wish to build a custom container on top of this one which includes your website. To see an example of this, see the [ten7/flight-deck-drupal](https://github.com/ten7/flight-deck-drupal) example container.

## Part of Flight Deck

This container is part of the [Flight Deck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).

## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

This container uses [Ansible](https://www.ansible.com/) to perform start-up tasks. To get even more verbose output from the start up scripts, set the `ANSIBLE_VERBOSITY` environment variable to `4`.

If the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` environment variable to `true` or `1`, then restart the container.

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
