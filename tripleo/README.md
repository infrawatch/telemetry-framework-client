# Telemetry Framework on OpenStack

In this directory contains a sample configuration that you can pass as
environment files to a TripleO / Director deployment to configure and setup
collectd and QDR.

## collectd.yaml

The `collectd.yaml` will configure the plugins for collectd, including the
AMQP1 module for connecting to the Apache QPID Dispatch Router.

## qdr.yaml

The `qdr.yaml` will configure the connection back to the telemetry framework
server where metrics (and eventually events) will be pushed across the message
bus for storage into Prometheus (future ElasticSearch for events).

# Deploying

You can deploy this with the `openstack deploy` command like so:

    openstack overcloud deploy \
    --timeout 100 \
    --templates /usr/share/openstack-tripleo-heat-templates \
    --stack overcloud \
    --libvirt-type kvm \
    --ntp-server 2.fedora.pool.ntp.org \
    -e /home/stack/virt/config_lvm.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml \
    -e /home/stack/virt/network/network-environment.yaml \
    -e /home/stack/virt/inject-trust-anchor.yaml \
    -e /home/stack/virt/hostnames.yml \
    -e /home/stack/virt/debug.yaml \
    -e /home/stack/virt/nodes_data.yaml \
    -e ~/containers-prepare-parameter.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/metrics-collectd-qdr.yaml \
    -e /home/stack/virt/qdr.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/collectd-environment.yaml \
    -e /home/stack/virt/collectd.yaml \
    --log-file overcloud_deployment_68.log

We leveraged [InfraRed](https://github.com/redhat-openstack/infrared) to build
out the configuration. Login to the undercloud and then modify the
`openstack_deploy.sh` script, and add the last 4 `-e` (environment file) lines
to deploy the `metrics-collectd-qdr.yaml` environment (configuration via
`qdr.yaml`) and the `collectd-environment.yaml` environment (configuration via
`collectd.yaml`).
