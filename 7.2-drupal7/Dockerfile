FROM ten7/flight-deck-web:develop
MAINTAINER tess@ten7.com

# This has to be re-run on each variant image, apparently.
USER root
RUN setcap cap_net_bind_service=+ep /usr/sbin/httpd

# Downgrade drush to 8.x for Drupal 7
USER apache
RUN rm -rf /var/www/.composer && \
    composer global require consolidation/cgr && \
    /var/www/.composer/vendor/bin/cgr drush/drush:~8.3.0 && \
    /var/www/.composer/vendor/bin/drush core:init -y && \
    rm -rf /var/www/.composer/cache
