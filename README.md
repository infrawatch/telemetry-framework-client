# Telemetry Ansible

Used during the deployment of a test bed. These files contain the configuration
and deployment strategies for getting a client side Barometer collectd
container and localized QDR setup for connection to the OpenShift cluster
running the telemtry platform.

## Prerequisites

It's assumed you're running CentOS 7.4, and that you've performed an update of
the system with `sudo yum update -y && sudo reboot`. Additionally, you should
install `epel-release` with `sudo yum install epel-release -y`.

After performing your system update and `epel-release` installation, you'll
need to install Ansible 2.5+ as well. You can do that with `sudo yum install
ansible -y`.

## Deployment

Deployment of a local executing barometer-collectd and QDR can be handled by
the playbook and roles and in the `ansible/` directory. To deploy, clone this
repo onto the machine you wish to deploy, and run the following command.

> **NOTE**
>
> Prior to deployment, you will likely need to modify the `vars/cloud_vars.yml`
> file to match your environment.

```
cd ansible/
ansible-galaxy install -r requirements.yml
ansible-playbook --ask-sudo-pass -e "@./vars/cloud_vars.yml" playbooks/cloud-monitor.yml
```
