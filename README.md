![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flight Deck Web

Flight Deck Web is a minimalist Apache/PHP container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* PHP 7.2, optimized out of the box for Drupal sites
* Drush 9, Drupal Console
* Node 10, Dart SASS

## Tags and versions

There are several tags available for this container, each with different software and support:

| PHP version | Tags | Drupal 8 | Drupal 7 | Drupal 6 |
| ----------- | ---- | -------- | -------- | -------- |
| 7.2 | 7.2, 7, latest | Yes | Mostly* | No |
| 7.2-image | 7.2-imagemagick | Yes | Mostly* | No |
| 5.6 | 5.6 | No | Yes | Yes |

* While the `7.2` container will run Drupal 7, the version of Drush included out of the box (9.x) is incompatible.

## Configuration

This container does not use environment variables for configuration. Instead, the `flight-deck-web.yml` file is used to handle all configuration.

```yaml
---
flightdeck_web:

```

Where:
* **secret** is the Varnish secret used to access the control terminal. Optional.

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
