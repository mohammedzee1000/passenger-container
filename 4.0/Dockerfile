FROM openshift/base-centos7

# This image provides a Passenger 4.0 environment you can use to run your Passenger
# applications.

EXPOSE 8080

ENV PASSENGER_VERSION 4.0

LABEL summary="Passenger 4.0 environment you can use to run your Passenger" \
      io.k8s.description="Passenger 4.0 environment you can use to run your Passenger" \
      io.k8s.display-name="Passenger 4.0" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,passenger,ruby,nodejs"

RUN yum install -y centos-release-scl-rh && \
    INSTALL_PKGS="rh-ruby22 rh-ruby22-ruby-devel rh-ruby22-rubygem-rake v8314 rh-ruby22-rubygem-bundler rh-ror41-rubygem-rack nodejs010 rh-passenger40-mod_passenger rh-passenger40-ruby22 httpd24 nss_wrapper" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y

# Copy the STI scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root
COPY ./contrib/ /opt/openshift

RUN chown -R 1001:0 /opt/app-root

# In order to drop the root user, we have to make some directories world
# writeable as OpenShift default "security" model is to run the container under
# random UID.
#    sed -ri ' s!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g;' /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf && \
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf && \
    head -n151 /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit && \
    mkdir /tmp/sessions && \
    mkdir -p /opt/app-root/src/public && \
    chmod -R og+rwx /opt/rh/httpd24/root/var/run/httpd && \
    chmod -R og+rwx /var/run/rh-passenger40 && \
    chown -R 1001:0 /opt/app-root /tmp/sessions && \
    cat /opt/app-root/etc/passenger.conf >> /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf

USER 1001

# Set the default CMD to print the usage of the language image
CMD "$STI_SCRIPTS_PATH/usage"
