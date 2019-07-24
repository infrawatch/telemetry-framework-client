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

You'll need to configure an inventory file for deployment within
`ansible/inventory/` subdirectory.

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

### `inventory/localhost.inventory`

Here is an example inventory file that does an installation on the local
machine that Ansible is being run from (i.e. not a remote installation).

Additionally, it is assumed that Docker and the docker-py modules have been
pre-installed, so we skip the Docker installation roles with the
`docker_install` and `docker_modules_prereqs` being set to `false`.

```
localhost ansible_host=localhost ansible_connection=local

[nodes]
localhost

[nodes:vars]
docker_install=false
docker_modules_prereqs=false
```
### Controlling custom connectors

If you need to connect the AMQP bus to another location other than the default
`qdr-white-port-5671-sa-telemetry.127.0.0.1.nip.io` route, you can override it
with a variables file. Create a new file within the `vars/` subdirectory and
place contents like the following into that file:

```
cat > vars/connectors.yaml <<EOF
connectors:
  - name: minishift_saf
    role: edge
    ssl_profile: sslProfile
    host: qdr-white-port-5671-sa-telemetry.192.168.42.85.nip.io
    port: 443
    verify_hostname: false
EOF
```

When performing the deployment, add `-e "@./vars/connectors.yaml"` to the
`ansible-playbook` command. See the [Deployment](#deployment) section for the
full deployment command.

## Deployment

Deployment of the kolla/collectd and interconnectedcloud/qdrouterd can be
handled by the playbook and roles and in the `ansible/` directory. To deploy,
clone this repo onto the machine you wish to deploy, and run the following
command.

```
cd ansible/
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventory/localhost.inventory playbooks/cloud-monitor.yml
```

If you created a custom set of connectors (or other variable overrides) you
would pass them to the `ansible-playbook` command.

```
ansible-playbook -i inventory/localhost.inventory -e "@./vars/connectors.yaml" playbooks/cloud-monitor.yaml
```
