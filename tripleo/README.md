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

# Deploying When InternalTLS is enabled.

Currently cert files can be generated for metrics_qdr via enabling internal TLS, which can be
used to secure internal communication, # NOTE(vinaykns) at present this may not be useful but 
this could be useful when a mesh topology is established using interior and edge mode operation.
Since InternalApi network is not present in CephStorage role we need to have a custom Ceph role and 
pass that during overcloud deployment. The custom ceph role looks something like

- name: CephStorage
  description: |
    Ceph OSD Storage node role
  networks:
    - Storage
    - StorageMgmt
    - InternalApi
  uses_deprecated_params: False
  deprecated_nic_config_name: 'ceph-storage.yaml'
  ServicesDefault:
    - OS::TripleO::Services::Aide
    - OS::TripleO::Services::AuditD
    - OS::TripleO::Services::CACerts
    - OS::TripleO::Services::CephOSD
    - OS::TripleO::Services::CertmongerUser
    - OS::TripleO::Services::Collectd
    - OS::TripleO::Services::Docker
    - OS::TripleO::Services::Fluentd
    - OS::TripleO::Services::IpaClient
    - OS::TripleO::Services::Ipsec
    - OS::TripleO::Services::Kernel
    - OS::TripleO::Services::LoginDefs
    - OS::TripleO::Services::MetricsQdr
    - OS::TripleO::Services::MySQLClient
    - OS::TripleO::Services::Ntp
    - OS::TripleO::Services::ContainersLogrotateCrond
    - OS::TripleO::Services::Rhsm
    - OS::TripleO::Services::RsyslogSidecar
    - OS::TripleO::Services::Securetty
    - OS::TripleO::Services::SensuClient
    - OS::TripleO::Services::Snmp
    - OS::TripleO::Services::Sshd
    - OS::TripleO::Services::Timezone
    - OS::TripleO::Services::TripleoFirewall
    - OS::TripleO::Services::TripleoPackages
    - OS::TripleO::Services::Tuned
    - OS::TripleO::Services::Ptp

The info regarding creating a custom role is found in https://docs.openstack.org/tripleo-docs/latest/install/advanced_deployment/custom_roles.html

So when composable services MetricsQdr and Collectd is enabled along with internal_tls then overcloud should be deployed as 
    
    openstack overcloud deploy \
    ...
    ...
    -r ~/roles_data.yaml \
    ...
    ...
  
