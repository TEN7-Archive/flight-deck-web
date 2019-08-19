FROM alpine:3.8
MAINTAINER tess@ten7.com
# Update the package list and install Ansible.
RUN apk -U upgrade &&\
    apk add --update --no-cache ansible && \
    rm -rf /tmp/* \
           /var/cache/apk/* \
           /usr/lib/python3.6/site-packages/ansible/modules/network/ \
           /usr/lib/python3.6/site-packages/ansible/modules/cloud/ \
           /usr/lib/python3.6/site-packages/ansible/modules/windows/

 # Copy the Ansible configuration files
 COPY ansible-hosts /etc/ansible/hosts
 COPY ansible.cfg /etc/ansible/ansible.cfg
 COPY ansible /ansible

 # Run the build.
 RUN ansible-galaxy install -fr /ansible/requirements.yml && \
     ansible-playbook -i /ansible/inventories/all.ini /ansible/build.yml

 # Allow Apache to allocate ports as non-root.
 RUN setcap cap_net_bind_service=+ep /usr/sbin/httpd

 # Configure the runtime environment of the container.
 EXPOSE 80
 WORKDIR /var/www

 # Switch to the service account.
 USER apache

 # Set the entrypoint and default command.
 ENTRYPOINT ["docker-entrypoint.sh"]
 CMD ["/usr/sbin/httpd", "-D", "FOREGROUND", "-f", "/etc/apache2/httpd.conf"]
