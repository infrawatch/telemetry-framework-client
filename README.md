# Telemetry Framework Client Automation

Used during the deployment of the telemetry framework client side test bed.
These files contain the configuration and deployment strategies for getting a
client side kolla/collectd container and interconnectedcloud/qdrouterd
(QDR) setup for connection to the OpenShift cluster running the telemetry
framework platform.

## Prerequisites

It's assumed you're running CentOS 7.6 (or RHEL 7.6), and that you've performed
an update of the system with `sudo yum update -y && sudo systemctl reboot`.
Additionally, you should install `epel-release` (CentOS) with `sudo yum install
epel-release -y`.

After performing your system update and `epel-release` installation, you'll
need to install Ansible 2.7+ as well. You can do that with `sudo yum install
ansible -y`.

## Configuration

You'll need to configure two files for deployment, but under the `ansible/`
subdirectory.

* inventory/virthost.inventory
* vars/qdr_certs.yml

### `inventory/virthost.inventory`

Here is an example inventory file that connects to a local libvirt hosted
virtual machine.

```
virthost ansible_host=192.168.122.238

[nodes]
virthost

[nodes:vars]
ansible_user=root
```

It's assumed you can login to the machine as root via passwordless (SSH keys)
SSH.

### `vars/qdr_certs.yml`

You also need to setup certificates to get through the OpenShift router on the
other side. The `vars/qdr_certs.yml` will need to variables filled in:

* `tls_certificate`
* `tls_key`

An example variable file looks like the following:

```
tls_certificate: |
    -----BEGIN CERTIFICATE-----
    <your_certificate_contents>
    -----END CERTIFICATE-----

tls_key: |
    -----BEGIN PRIVATE KEY-----
    <your_key_contents>
    -----END PRIVATE KEY-----

```

The certificates can be generated like so, but you'll need to make sure they
are loaded on the server side implementation as well. See
https://github.com/redhat-service-assurance/telemetry-framework for more
information.

```
openssl req -new -x509 -batch -nodes -days 11000 \
    -subj "/O=io.interconnectedcloud/CN=qdr-white-normal.sa-telemetry.svc.cluster.local " \
    -out qdr-server-certs/tls.crt \
    -keyout qdr-server-certs/tls.key
```

## Deployment

Deployment of the kolla/collectd and interconnectedcloud/qdrouterd can be
handled by the playbook and roles and in the `ansible/` directory. To deploy,
clone this repo onto the machine you wish to deploy, and run the following
command.

> **NOTE**
>
> Prior to deployment, you will likely need to modify the `vars/cloud_vars.yml`
> file to match your environment.

```
cd ansible/
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventory/virthost.inventory -e "@./vars/qdr_certs.yml" playbooks/cloud-monitor.yml
```
