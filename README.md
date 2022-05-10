OpenVPN role
=========

Role to manage OpenVPN server settings with TOTP MFA support.

Features:

* time based one time password support ("google" authenticator)
* one file client config file (just import)
* send instructions via email (welcome email with config and link to QR code)
* time limited access to MFA QR code (link expires in 48h by default)
* access control (push route and iptables rules)

Requirements
------------

Debian based distribution (tested on Debian Buster).
Nginx for time limited QR code access.

Example Playbook
----------------

Role for servers (without MFA, without sending config):

```yaml
    - role: openvpn
      vars:
        openvpn_name: servers
        openvpn_hostname: vpn.example.com
        openvpn_port: 1195
        openvpn_subnet: 172.16.0.0
        openvpn_netmask: 255.255.255.0
        openvpn_internal_address: 172.16.0.1
        openvpn_authenticator: false
        openvpn_send_config: false
        openvpn_domain: example.com
```

Roles for users (with MFA, sending config and QR code link via email)

```yaml
    - role: openvpn
      vars:
        openvpn_name: users
        openvpn_hostname: vpn.example.com
        openvpn_port: 1194
        openvpn_subnet: 172.16.1.0
        openvpn_netmask: 255.255.255.0
        openvpn_internal_address: 172.16.1.1
        openvpn_authenticator: true
        openvpn_send_config: true
        openvpn_domain: example.com
        openvpn_smtp_server: smtp.example.com
        openvpn_smtp_user: postmaster@example.com
        openvpn_smtp_pass: super-secret-password
```

VPN vars file for network definition and user access

```yaml
openvpn_networks:
  vpn-servers:
    - name: vpn-servers
      address: 172.16.0.0
      netmask: 255.255.255.0
      mask: 24
  vpn-users:
    - name: vpn-users
      address: 172.16.1.0
      netmask: 255.255.255.0
      mask: 24
  secret-area:
    - name: secret-area-a
      address: 10.255.0.0
      netmask: 255.255.0.0
      mask: 16

openvpn_clients:
  servers:
    - name: external-server-1
      address: 172.16.0.10
      networks:
        - vpn-servers
        - secret-area
    - name: external-server-2
      address: 172.16.0.11
      networks:
        - vpn-servers
  users:
    - name: jindrich.skupa
      email: jindrich.skupa
      address: 172.16.1.10
      networks:
        - vpn-servers
        - vpn-users
        - secret-area
    - name: name.surname
      address: 172.16.1.11
      networks:
        - vpn-servers
```

TODO
----

We would like to add following features:

* custom welcome email template
* move more hard coded values to variables
* configure nginx
* randomize QR code image name

License
-------

BSD

Author Information
------------------

Jindrich Skupa
