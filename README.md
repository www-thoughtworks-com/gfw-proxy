# Peeping Over The Wall


## Requirements

 * Python >= 2.7
 * Boto >= 2.38
 * Ansible >= 1.9

*NOTE: You might get away with slightly older versions but it's as yet untested.*


## Setup

### AWS

There are a variety of ways to provide Boto with your AWS API credentials, but perhaps the simplest way is just to export them as environment variables...

```shell
$ export AWS_ACCESS_KEY_ID='AK...'
$ export AWS_SECRET_ACCESS_KEY='SECRET'
```

The `spinup.yml` playbook will...

 1. Create an SSH key pair if one doesn't already exist and write the private key to `./${PROXY_NAME}.pem` *(Git ignored for your safety)*.
 2. Create a security group allowing all egress traffic and SSH ingress limited to an IP whitelist, see `vars.yml`.
 3. Launch a EBS-SSD backed HVM EC2 instance from the latest official Ubuntu 14.04 LTS Marketplace AMI.
 4. Return the public DNS name (and IP address) of your new proxy instance.

Play the `spinup.yml` playbook...

```shell
$ ansible-playbook -i hosts spinup.yml
```

**WARNING: Please concider all the security implications of using this project and never commit your AWS access id/key or SSH private key to Git.**

### SSH

Establish an SSH tunnel...

```shell
$ ssh -i ${PROXY_NAME}.pem -vND 1080 -l ubuntu ${PROXY_PUBLIC_DNS_NAME}|${PROXY.PUBLIC_IP}
```

See `man 1 ssh` for details.

### Browser

Configure your browser to use the SOCKS5 proxy, see the documentation for your browser for details.

An example using Chromium...

```shell
$ chromium --proxy-server='socks5://127.0.0.1:1080' http://whatismyipaddress.com/
```

*NOTE: Make sure Chromium is not running before executing this command, otherwise the proxy settings will not take effect.*


## Teardown

In the interests of reducing costs, the proxy is a `t2.micro` instance, Amazon's cheapest instance type, which should be more than adequate.  However, you should also consider terminating the instance when not in use.  The `teardown.yml` playbook cleans up after `spinup.yml`.

Play the `teardown.yml` playbook...

```shell
$ ansible-playbook -i hosts teardown.yml
```


## Todo

**Start and stop instance via Ansible**  
There is currently a bug using Ansible's EC2 dynamic inventory with the `cn-north-1` (Beijing) region.  This bug, along with the fact that this story was timeboxed to one afternoon, blocked implementing this feature.  It should however be considered for future development.
