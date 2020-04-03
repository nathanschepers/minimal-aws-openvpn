# Minimal EC2 instance with OpenVPN

This repository contains an Ansible playbook and associated roles that support 
the dynamic creation of Amazon EC2 instances with OpenVPN. These instances also include dnsmasq and
allow for flexible DNS configuration.


## Prerequisites

This playbook requires the following:

 - an Amazon IAM user with the [SystemAdministrator](https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/job-function/SystemAdministrator$serviceLevelSummary) policy associated. This user will preform all of the creation and administration actions needed by the playbook. It will probably be useful to put their access credentials in `~/.aws/credentials` or somewhere else that the aws cli can get to them.
 - an Amazon EC2 [keypair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) used for accessing the instance via SSH. It would probably be useful to put this in `~/.ssh`, and addit as an identity to `~/.ssh/config`
 - an Amazon EC2 [security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html), allowing (at *least*) inbound TCP access on port 22 (for SSH) and inbound UDP acess on port 1194 (for OpenVPN)

## Roles

### Provisioning

The provisioning phase will create a new EC2 instance, using the paramters specified in `main.yml`:

```yaml
- name: Provision Resources
  hosts: localhost
  vars:
    # AWS-related variables
    aws_ec2_provision: True
    aws_ec2_keypair: minimal-openvpn-keypair
    aws_ec2_instance_type: t2.micro
    aws_ec2_image: ami-0b6f46ba4d94838a0
    aws_ec2_region: eu-central-1
    aws_ec2_security_group: minimal-openvpn-security-group
```

Important to note here are the keypair and security group, mentioned above.

Also of note here: the default ami is for Ubuntu 18.04LTS, as released by canonical. You can probably replace this with any debian-based ami, but be careful that the ami you choose will work in the specified EC2 region and will run on the specified instance type.

### Pre-Setup

The pre-setup phase installs `python`, waits for any automatic first-boot updates, and then explicitly updates its package repositories and upgrades any out-of-date packages.

### VPN and dnsmasq configuration

This phase installs OpenVPN, and creates a new Certificate Authority. It then generates appropriate server certificates and client keys.

dnsmasq is then installed and configured using the following sections from `main.yml`:

```yaml
    vpn_domain: sandbox.test
    vpn_host: www
```

Finally, a `.ovpn` file is created using the generated client keys. This file is suitable for use on any platform that supports Openvpn Client.

### Some notes

`dnsmasq` is set up as both a DNS and DHCP server. Once the OpenVPN connection is established, the client receives its IP address *and* DNS server from dnsmasq. This means, for example, that the newly set up machine in our example can be accessed simply by

1 - clicking on the .ovpn file  
2 - executing `ssh ubuntu@www.sandbox.test`

The domain and hostnames are completely configurable, and multiple users can use the same .ovpn file to connect simultaneously.
 
