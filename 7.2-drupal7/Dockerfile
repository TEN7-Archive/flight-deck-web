FROM ten7/flight-deck-web:7.2
MAINTAINER tess@ten7.com

# This has to be re-run on each variant image, apparently.
USER root
RUN setcap cap_net_bind_service=+ep /usr/sbin/httpd

# Downgrade drush to 8.x for Drupal 7
RUN curl -L -o /usr/local/bin/drush https://github.com/drush-ops/drush/releases/download/8.3.5/drush.phar && \
    chown root:apache /usr/local/bin/drush && \
    chmod +x /usr/local/bin/drush

USER apache
RUN /bin/bash -c "drush init --bg -y"
