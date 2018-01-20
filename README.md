# haproxy

Ansible role for installing and configuring HAProxy for one or more load balanced applications.

This role takes the opinion that rather then caring about configuring frontend and backend servers in HAProxy the user should be careing about configuring load balanced applications. Therefor the use of this role is predicated on that idea where by the user specifies the applciations she or he would like to load balance and let the role take care of figuring out how to configure the frontend and backend servers of HAProxy.

## Requirements

Requirements for this role to funciton.

### Packages

Packages that must be available for the role to install.
* haproxy

### Services

Services that must be enabled in order for the role to configure them.
* firewalld

## Role Variables
Information about the expected role parameters.

| parameter             | required | default | choices         | comments 
| --------------------- |:--------:|:-------:| --------------- |:-------- 
| haproxy_applications  | yes      |         |                 | List of hashes defining applications to load balance

### haproxy_applications
The `haproxy_applications` parameter is a list of hashes defining the applications to load balance. Each item in the list may contain the following parameters.

| parameter              | required | default | choices     | comments 
| ---------------------- |:--------:|:-------:| ----------- |:-------- 
| name                   | yes      |         |             | Name of the application. Used in defining frontend and backend servers. Should be descriptive as will show up in HAProxy stats. Must match `/a-zA-Z0-9-_/`
| domain                 | yes      |         |             | FQDN which will resolve to the HAProxy server(s) to then load balance the `servers`. Can either just be a simple FQDN or a regex statement to match a domain.
| domain\_is\_regex      | no       | false   | true, false | `true` if the given `domain` is regex, `false` to treat as plane FQDN.
| expose_http            | no       | false   | true, false | `true` to expose this application on http, `false` to not expose on http.
| expose_https           | no       | false   | true, false | `true` to expose this application on https, `false` to not expose on https.
| redirect_http_to_https | no       | false   | true, false | `true` to automatically redirect http to https, `false` not to redirect.
| servers                | yes      |         |             | List of hashes defining servers to load balance.

#### servers
Each element in the `haproxy_applications` list must contain a `servers` key which is a list of hashes defining the servers to load balance for the respective application. Each item in the list may contain the following parameters.

| parameter  | required | default | choices         | comments 
| ---------- |:--------:|:-------:| --------------- |:-------- 
| name       | yes      |         |                 | Name used to reference the server. Will be displayed in the HAProxy stats. Must match `/a-zA-Z0-9-_/`
| address    | yes      |         |                 | FQDN or IP of server to load balance.
| port_http  | no       | 80      |                 | Port of server at `address` to load balance when `expose_http` is `true`.
| port_https | no       | 443     |                 | Port of server at `address` to load balance when `expose_https` is `true`.

## Example Playbooks

## Load balance Ansible Tower
    - name: HAProxy
      hosts: haproxy
      roles:
        - role: haproxy
          haproxy_applications:
            - name: ansible-tower
              domain: tower.example.com
              expose_https: True
              redirect_http_to_https: True
              servers:
                - name: tower0002
                  address: tower0002.example.com
                - name: tower0003
                  address: tower0003.example.com
                - name: tower0004
                  address: tower0004.example.com

## Load balance OpenShift Container Platform (OCP) masters and routers
    - name: HAProxy
      hosts: haproxy
      roles:
        - role: haproxy
          haproxy_applications:
            - name: ocp-admin
              domain: ocp.example.com
              expose_https: True
              redirect_http_to_https: True
              servers:
                - name: ocp0002-master
                  address: ocp0002.example.com
                - name: ocp0003-master
                  address: ocp0003.example.com
                - name: ocp0004-master
                  address: ocp0004.example.com
            - name: ocp-router
              domain: .*.apps.example.com
              domain_is_regex: True
              expose_https: True
              expose_http: True
              redirect_http_to_https: False
              servers:
                - name: ocp0005-infra
                  address: ocp0005.example.com
                - name: ocp0006-infra
                  address: ocp0006.example.com
                - name: ocp0007-infra
                  address: ocp0007.example.com


## Author Information

* Red Hat Consulting
  * Ian Tewksbury (<itewk@redhat.com>)
