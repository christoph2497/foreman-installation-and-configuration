# Playbook for installation and configuration of Foreman

This ansible playbook installs a full Foreman-Katello scenario. The added features are 
- TFTP
- DNS
- DHCP
- libvirt as Compute Resource
- Remote Execution

##
As part of the playbook there will also be created a test-vm with RHEL9 as operating system.
 
**A redhat manifest is needed to handle the subscription process.**

##

The playbook uses `host_vars` and `group_vars` and it always runs for the group `foremanserver`


All variables regarding the installation and configuration-process are set in `group_vars/foremanserver.yml`

## Prerequisites

To use this playbook make sure that the collections from the `requirements.yml` file are installed
The used collections are `theforeman.foreman`, `theforeman.operations` and  `ansible.posix`

Use this command to install them if they are not installed yet
```
ansible-galaxy install -r requirements.yml
```
##
Also make sure to replace `foreman.localdomain` with your own hostname in the `hosts.yml` 

```yaml
---
foremanserver:
  hosts:
    your.hostname
```
Keep in mind that you have to rename the file under `/host_vars` accordingly to the host name you set in `hosts.yml`


##
Don't forget to put your Red Hat manifest file under `files`. It's important that you name it `manifest.zip` otherwise the playbook won't recognize your manifest file.

```
├── files
│   ├── manifest.zip
```
##
You will also need to create a new ssh-key pair and place it in the `files` directory
Keep in mind that you name your public key `id_rsa.pub` and your private key `id_rsa` otherwise it won't work.

```
ssh-keygen -t rsa
```
If you want to use another encritpion algorithm, you need to adjust the playbook accordingly!

##
Your files directory should now look like this

```
├── files
│   ├── manifest.zip
│   ├── id_rsa
│   └── id_rsa.pub
```

## File Stucture 

The file structure of the project should look like this

```
├── files
│   ├── host.localdomain.pub
│   ├── id_rsa
│   └── id_rsa.pub
├── group_vars
│   └── foremanserver.yml
├── hosts.yml
├── host_vars
│   └── foreman.localdomain.yml
├── playbook.yml
├── README.md
└── requirements.yml

```


## Variables

All variables are set in either in the `group_vars` or in the `host_vars`

| Variable | Type  | Description |
| ---------| ----- | ---------|
| `foreman_repositories_version` | `String` | Version of Foreman to setup repositories for |
| `foreman_repositories_katello_version` | `String` | Version of Katello to setup repositories |
| `foreman_puppet_repositories_version` | `String` | Version of puppet to setup repositories |
| `username`      | `String` | Username accessing the Foreman server |
| `password`      | `String` | Password of the user accessing the Foreman server. |
| `server_url`    | `String` | URL of the Foreman server |
| `organization`  | `String` | The name of the organization |
| `root_password` | `String` | The root password that is set for the provisioned host |

`username`, `password` and `organization` need to match with the values set in `foreman_installer_options`

## Run the playbook

To run the playbook run the following command after you satisified all requirements

```bash
ansible-playbook -i hosts.yml playbook.yml
```

## Examples

### Example host inventory
```yaml
---
foremanserver:
  hosts:
    foreman.localdomain
```

### Example group_vars.yml

```yaml
username: "admin"
password: "admin"
server_url: "https://foreman.localdomain"
organization: "organization"
root_password: "changeme"

foreman_repositories_version: "3.12"
foreman_repositories_katello_version: "4.14"                                                                                                                                                  

foreman_puppet_repositories_version: 8
foreman_installer_scenario: katello
foreman_installer_package: foreman-installer-katello
foreman_installer_options:
  - '--foreman-proxy-tftp=true'
  - '--foreman-proxy-tftp-servername 10.0.0.2'
  - '--foreman-proxy-dns=true'
  - '--foreman-proxy-dns-interface=enp1s0'
  - '--foreman-proxy-dns-zone=localdomain'
  - '--foreman-proxy-dns-reverse=0.10.in-addr.arpa'
  - '--foreman-proxy-dns-forwarders=8.8.8.8'
  - '--foreman-proxy-dns-forwarders=8.8.4.4'
  - '--foreman-proxy-dhcp=true'
  - '--foreman-proxy-dhcp-interface=enp1s0'
  - '--foreman-proxy-dhcp-gateway=10.0.0.1'
  - '--foreman-proxy-dhcp-range="10.0.0.100 10.0.0.200"'
  - '--foreman-proxy-dhcp-nameservers="10.0.0.2"'
  - '--foreman-initial-organization "organization"'
  - '--foreman-initial-admin-password changeme'
  - '--tuning "development"'
  - '--enable-foreman-compute-libvirt'
  - '--enable-foreman-plugin-remote-execution'
  - '--enable-foreman-proxy-plugin-remote-execution-script'


```
