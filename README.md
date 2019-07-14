![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flight Deck Web

Flight Deck Web is a minimalist Apache/PHP container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* PHP 7.2, optimized out of the box for Drupal sites
* Drush 9, Drupal Console
* Node 10, Dart SASS
* Built in XDebug support

## Tags and versions

There are several tags available for this container, each with different software and support:

| PHP version | Tags | Drupal 8 | Drupal 7 | Drupal 6 |
| ----------- | ---- | -------- | -------- | -------- |
| 7.2 | 7.2, 7, latest | Yes | Mostly* | No |
| 7.2-imagemagick | 7.2-imagemagick | Yes | Mostly* | No |
| 7.1 | 7.1 | Yes | Yes | No |
| 5.6 | 5.6 | No | Yes | Yes |

* While the `7.2` container will run Drupal 7, the version of Drush included out of the box (9.x) is incompatible.

## Configuration

This container uses the following environment variables for configuration.

### Web server

`APACHE_DOCROOT_DIR`

Is the path inside the container to the web server docroot directory. Optional, defaults to `/var/www/html`.

`APACHE_SITE_NAME`

The expected hostname for the site. Optional, defaults to `docker.test`. Note that this container will handle any traffic sent to it irrespective of the name.

`APACHE_SITE_ALIAS`

An additional alias for the site being hosted. Optional.

`T7_SITE_ENVIRONMENT`

Specifies the operational environment in which the container runs such as production, stage, or test. Optional, defaults to `docker-dev`.

`HTTPS_CERT`

The SSL certificate -- with chain -- to use for HTTPS sites. Optional.

`HTTPS_KEY`

The private key for the SSL certificate. Optional.

### PHP

`PHP_INI_PATH`

The full path in the container to the PHP ini file. Optional, defaults to `/etc/php7/php.ini` which is prepopulated by the container.

`PHP_SENDMAIL_PATH`

The path to the sendmail executable inside the container. Optional.

`PHP_MAX_EXEC_TIME`

The maximum execution time for PHP in seconds. Optional, defaults to `120`.

`PHP_MAX_INPUT_VARS`

The maximum input variables PHP can handle for one request. Optional, defaults to `3000`.

`PHP_MEMORY_LIMIT`

The maximum amount of memory a web server request to PHP can allocate before being terminated. Optional, defaults to `320m`. CLI PHP has no memory limit.

`PHP_XDEBUG_INI_PATH`

The full path to the XDebug configuration file inside the container. Optional, defaults to `/etc/php7/conf.d/xdebug.ini` which is provided by the container.

### XDebug

`PHP_XDEBUG_ENABLED`

Specifies if XDebug is enabled or not. Optional, defaults to `0` to disable XDebug as it has performance implications.

`PHP_XDEBUG_AUTOSTART`

Specifies if XDebug should auto-start with any request when enabled. Optional, defaults to `0` for no.

`PHP_XDEBUG_REMOTE_CONNECT_BACK`

Specifies if connect-back should be used when debugging. Optional, defaults to `0` for no as this feature only works on Docker on Linux, or when using Docker Machine. Connect back does not work on Docker for Mac or Docker for Windows.

`PHP_XDEBUG_STDOUT_LOGS`

Specifies if XDebug logs should be sent to stdout. Optional, defaults to `0` for no as this feature is only needed to debug XDebug configuration.

### XDebug Profiler

`PHP_XDEBUG_PROFILE_FORCE`

Enable XDebug profiling for any request. Optional, defaults to `0` for no as this has **severe** performance implications.

`PHP_XDEBUG_PROFILE_TRIGGER`

Specifies the GET query key to use to trigger profiling. Optional.

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy the web container as a deployment. By default, it shares the same pod with [`ten7/flight-deck-varnish`](https://github.com/ten7/flight-deck-varnish):

```yaml
flightdeck_cluster:
  namespace: "example-com"
  web:
    replicas: 1
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
```

## Part of Flight Deck

This container is part of the [Flight Deck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
