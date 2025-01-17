# StackStorm

- [StackStorm](#stackstorm)
  - [Introduction](#introduction)
  - [Documentation](#documentation)
  - [Architecture](#architecture)
  - [Installation](#installation)
    - [One-line Install](#one-line-install)
  - [Environment Setup](#environment-setup)
  - [Basic Configuration](#basic-configuration)
    - [Login Methods](#login-methods)
    - [User Management](#user-management)
  - [Packs](#packs)
    - [List packs](#list-packs)
    - [Install and Uninstall packs](#install-and-uninstall-packs)
    - [Pack information](#pack-information)
  - [Actions](#actions)
    - [List actions](#list-actions)
    - [Execute actions](#execute-actions)
      - [Web UI](#web-ui)
      - [CLI](#cli)
      - [Rest API](#rest-api)
    - [List execution](#list-execution)
  - [Pack development](#pack-development)
    - [YAML](#yaml)
    - [Jinja2](#jinja2)
      - [String operations](#string-operations)
      - [Arithmetic operations](#arithmetic-operations)
      - [Comparison operations](#comparison-operations)
      - [Membership operations](#membership-operations)
      - [Define a variable in Jinja](#define-a-variable-in-jinja)
      - [Array (List) and Map (Dictionary) operations](#array-list-and-map-dictionary-operations)
      - [Conditional and loop operations](#conditional-and-loop-operations)
    - [Creating custom pack](#creating-custom-pack)
      - [Folder structure](#folder-structure)
      - [Actions](#actions-1)
  - [Workflows](#workflows)
    - [Orquesta Graph Editor](#orquesta-graph-editor)
    - [Creating workflow](#creating-workflow)
    - [Workflow Functions](#workflow-functions)
    - [Workflow Summary](#workflow-summary)

## Introduction

StackStorm is an open source event-driven platform for automating your tools and infrastructure. It is licensed under the Apache 2.0 License.

There is also a commercial version of StackStorm called Extreme Workflow Composer (EWC) which adds additional features like priority support, workflow designer and RBAC.


## Documentation

- [StackStorm Documentation](https://docs.stackstorm.com/index.html)
- [StackStorm Exchange](https://exchange.stackstorm.org/)
- [StackStorm API Reference](https://api.stackstorm.com/)
- [Ansible Template Tester](https://ansible.sivel.net/test/)
- [Create and Contribute a Pack](https://docs.stackstorm.com/reference/packs.html)
- [Action Runners](https://docs.stackstorm.com/actions.html#action-runners)
- [Curl to Request](https://curl.trillworks.com/)
- [Orquesta workflow](https://docs.stackstorm.com/orquesta/index.html)
- [Rehearsal Fork - Orquesta Graph Editor](https://github.com/maroskukan/rehearsal)

## Architecture

Stackstorm is composed of the following main components:

- Sensors - monitor when an event happens
- Rules - check what needs to be done
- Workflows - run a set of instructions
- Actions - execute relevant commands
- Results - process results for analysus or additional triggers

When deployed for single system the reference architecture looks as follows:

![Architecture](https://docs.stackstorm.com/_images/st2-deployment-big-picture.png)


StackStorm provides great level of extensibility by using content packs. Packs combination of *Sensors*, *Rules*, *Workflows* and *Actions*. They are available for many different services and applications, visit [StackStorm Exchange](https://exchange.stackstorm.org/) for full lisf of maintaned packs. You can also create a custom pack for your own specific needs.

Tasks can be also executed manually by using one of the following methods:
- Command Line Interface - powerful and easy for administrators and developers
- Web User Interface - suited for operation team members
- REST API - excellent for integration with any upstream custom applications

Connectivity to managed nodes is handled through SSH or REST API.


## Installation

StackStorm is supported on Ubuntu, RHEL and CentOS Linux. The system requirements for testing and production are different, therefore always refer to [documentation](https://docs.stackstorm.com/install/system_requirements.html) when defining the underlying platform. At the time of writing they are:

| Testing      | Production   |
| ------------ | ------------ |
| 2 vCPU       | 4 vCPU       |
| 2GB RAM      | >16GB RAM    |
| 10GB HDD     | 40GB HDD     |
| t2.medium    | m4.xlarge    |

StackStorm is distributed as an RPMs and Debs packages as well as Docker images, therefore there ware multiple ways how to perform instalation, for example.

- **One-line Install** - using provided shell script for an opionated install for all components
- **Manual Installation** - great for custom needs
- **Vagrant / Virtual Appliance** - pre-installed, tested and shipped as virtual image, great for testing, pack develoment or demo
- **Docker** - quickiest way to get StackStorm running, useful for trying platform and development
- **Ansible Playbooks** - great for repeatable, consistent, and idempotent installation using Ansible
- **Puppet Module** - greate for repeatable, consistent and idempotent installation using Puppet
- **High Availability** - great for business critital automation by using StackStorm HA Cluster in Kubernetes

### One-line Install

Provision a clean installation of `Ubuntu2004` and install StackStorm using one-line script. It will automatically install required components.

```bash
# Provision and open an SSH session to VM
cd install/one-line && vagrant up && vagrant ssh

# Install StackStorm
bash <(curl -sSL https://stackstorm.com/packages/install.sh) --user=st2admin --password=Ch@ngeMe
```

One the installation completed the StackStorm components should be up and running. You can use the following methods to verify the installation.

Verify st2 and python version.

```bash
st2 --version
st2 3.5.0, on Python 3.8.10

/opt/stackstorm/st2/bin/python3 --version
Python 3.8.10
```

Verify `nginx`, `rabbitmq`, `mongod`, `st2api` service

```bash
systemctl status nginx
● nginx.service - nginx - high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-09-22 11:42:34 UTC; 11min ago
       Docs: https://nginx.org/en/docs/
   Main PID: 17659 (nginx)
      Tasks: 3 (limit: 4617)
     Memory: 3.3M
     CGroup: /system.slice/nginx.service
             ├─17659 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
             ├─17660 nginx: worker process
             └─17661 nginx: worker process

systemctl status rabbitmq-server.service 
● rabbitmq-server.service - RabbitMQ Messaging Server
     Loaded: loaded (/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-09-22 11:39:20 UTC; 15min ago
   Main PID: 5728 (beam.smp)
     Status: "Initialized"
      Tasks: 87 (limit: 4617)
     Memory: 76.0M
     CGroup: /system.slice/rabbitmq-server.service
             ├─5711 /bin/sh /usr/sbin/rabbitmq-server
             ├─5728 /usr/lib/erlang/erts-10.6.4/bin/beam.smp -W w -A 64 -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs>
             ├─5992 erl_child_setup 65536
             ├─6017 inet_gethost 4
             └─6018 inet_gethost 4

systemctl status mongod.service 
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-09-22 11:40:01 UTC; 15min ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 7358 (mongod)
     Memory: 191.6M
     CGroup: /system.slice/mongod.service
             └─7358 /usr/bin/mongod --config /etc/mongod.conf

Sep 22 11:40:01 stackstorm systemd[1]: Started MongoDB Database Server.

systemctl status st2api
● st2api.service - StackStorm service st2api
     Loaded: loaded (/lib/systemd/system/st2api.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-09-22 11:41:53 UTC; 26min ago
TriggeredBy: ● st2api.socket
   Main PID: 15881 (gunicorn)
      Tasks: 2 (limit: 4617)
     Memory: 91.2M
     CGroup: /system.slice/st2api.service
             ├─15881 /opt/stackstorm/st2/bin/python /opt/stackstorm/st2/bin/gunicorn st2api.wsgi:application -k eventlet -b>
             └─15892 /opt/stackstorm/st2/bin/python /opt/stackstorm/st2/bin/gunicorn st2api.wsgi:application -k eventlet -b>
```

You can use show sockets `ss` to verify which ports are listening.

```bash
ss -tlp
State      Recv-Q     Send-Q         Local Address:Port               Peer Address:Port    Process    
LISTEN     0          511                127.0.0.1:6379                    0.0.0.0:*                  
LISTEN     0          2048               127.0.0.1:9100                    0.0.0.0:*                  
LISTEN     0          2048               127.0.0.1:bacula-dir              0.0.0.0:*                  
LISTEN     0          2048               127.0.0.1:bacula-fd               0.0.0.0:*                  
LISTEN     0          511                  0.0.0.0:http                    0.0.0.0:*                  
LISTEN     0          4096           127.0.0.53%lo:domain                  0.0.0.0:*                  
LISTEN     0          128                  0.0.0.0:ssh                     0.0.0.0:*                  
LISTEN     0          511                  0.0.0.0:https                   0.0.0.0:*                  
LISTEN     0          128                127.0.0.1:amqp                    0.0.0.0:*                  
LISTEN     0          128                  0.0.0.0:25672                   0.0.0.0:*                  
LISTEN     0          4096               127.0.0.1:27017                   0.0.0.0:*                  
LISTEN     0          4096                       *:epmd                          *:*                  
LISTEN     0          128                     [::]:ssh                        [::]:* 
```

To verify st2 component status use the `st2ctl status` command:

```bash
st2ctl status
##### st2 components status #####
st2actionrunner PID: 15955
st2actionrunner PID: 15957
st2actionrunner PID: 15959
st2actionrunner PID: 15961
st2actionrunner PID: 15963
st2actionrunner PID: 15965
st2actionrunner PID: 15967
st2actionrunner PID: 15969
st2actionrunner PID: 15971
st2actionrunner PID: 15973
st2api PID: 15881
st2api PID: 15892
st2stream PID: 15830
st2stream PID: 15874
st2auth PID: 15775
st2auth PID: 15802
st2garbagecollector PID: 14548
st2notifier PID: 14549
st2rulesengine PID: 14550
st2sensorcontainer PID: 15898
st2chatops is not running.
st2timersengine PID: 14553
st2workflowengine PID: 15909
st2scheduler PID: 14552
```

Some important paths and files to remeber are:
- `/opt/stackstorm`
- `/etc/st2`
- `/etc/st2/st2.conf`
- `/var/log/st2`

With quick installation StackStorm Automation tasks will run by default with `stanley` system user on the OS. This use has sudo privileges. This use will be also used to connect to remote managed system by default.

```bash
id stanley
uid=1001(stanley) gid=1001(stanley) groups=1001(stanley)

cat /etc/sudoers.d/st2
Defaults env_keep += "http_proxy https_proxy no_proxy proxy_ca_bundle_path DEBIAN_FRONTEND"
stanley    ALL=(ALL)       NOPASSWD: SETENV: ALL

grep -A2 system_user /etc/st2/st2.conf 
[system_user]
user = stanley
ssh_key_file = /home/stanley/.ssh/stanley_rsa
```

When you modify the st2.conf file you need to restart the service with `st2ctl restart` command to apply the new configuration.


## Environment Setup

To execute actions on remote hosts, StackStorm uses SSH connection. It is recommended to use password-less authentication using SSH keys.

One way to setup password-less authentication is to generate public key (if not available already) for `stanley` user on StackStorm Engine. And then create a `stanley` user on each managed node and copy the public key to `authorized_keys` file on each managed node. [documentation](https://docs.stackstorm.com/install/config/config.html#configure-ssh).


Start by creating a RSA key pair on StackStorm host if it is not already available.

```bash
sudo su stanley
ssh-keygen -f /home/stanley/.ssh/stanley_rsa -P ""
```

Next, on each managed system, create new user, update authorized_keys file and update sudoers.

```bash
useradd stanley
mkdir -p /home/stanley/.ssh
chmod 0700 /home/stanley/.ssh

cat <<EOF > /home/stanley/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDsMdrhVmDHIdfEDQvXDZCLbaY3e1E/wPz4q8s+XZAgg94aWK6i4HAWlT9jfxaCQIOAXWnIkYVrn5sgAbutCEPYE2uS0shymB4S1gO0YYIxuSP9OOMSR+zMA1SeYvPU4QfSEtYeYG9iMlwNmzsyq48Td40SMvdBMBOcMn9KyxW+IpPmbuXyk+fs//ulXlWEH744BoUzqz4jTzGR7yglplTL7QTxdxLfaAGXRyCkUUBK6x7qACXobim0YVBFk35/dIjW4gVPKvZhrBehhzBCTWSvCPNdWXGQQm4OQzExNhPndrY0+dKuhXTRemwn9bn3BEGZauLY7DDcWPxAiy45r9WeqkBF9n9SKM5AVjBP8LxzfHTHNNvDLh8eoUOlaAaRCQflEQWN5H0r4HNhcoWuVx7RtUNzFjMvnGUekhADIEdVjquOgSIo0TFyk+6mNUnujGjO283eY1Ba7zRdLlNU8fvwJgjZYb4btTwI4lyBlFHgMqI9eu1JcDCT5SNABNSpu88= root@stackstorm
EOF

chmod 0600 /home/stanley/.ssh/authorized_keys
chown -R stanley:stanley /home/stanley
echo "stanley    ALL=(ALL)       NOPASSWD: SETENV: ALL" >> /etc/sudoers.d/st2

sudo sed -i -r "s/^Defaults\s+\+requiretty/# Defaults +requiretty/g" /etc/sudoers
```

Finally, verify key-based authentication and sudo privileges from StackStorm host.

```bash
# ssh should not require a password since the key is already provided
ssh -i /home/stanley/.ssh/stanley_rsa stanley@host.example.com "sudo id"
uid=0(root) gid=0(root) groups=0(root)
```

## Basic Configuration

ST2 provides number of CLI commands for managing the instance. To get full list of them, just use `st2` command with tab completion and you will get the following result back.

```bash
st2                                st2-generate-symmetric-crypto-key  st2-self-check
st2-apply-rbac-definitions         st2-register-content               st2-track-result
st2-bootstrap-rmq                  st2-rule-tester                    st2-trigger-refire
st2ctl                             st2-run-pack-tests                 st2-validate-pack-config
```

From this list, the most used ones are `st2` and `st2ctl`.

The `st2ctl` is used for StackStorm administration. For example to start, stop, restart service or reload a component. This is required when you change the configuration.

The `st2` is used to login and execute automation tasks

### Login Methods

THere are two primary ways how to login into StackStorm:
- Web UI using https://<st2-ip-address>/
- CLI `st2 login st2admin`

```bash
st2 login st2admin
Password:
Logged in as st2admin

Note: You didn't use --write-password option so the password hasn't been stored in the client config and you will need to login again in 24 hours when the auth token expires.
As an alternative, you can run st2 login command with the "--write-password" flag, but keep it mind this will cause it to store the password in plain-text in the client config file (~/.st2/config).
```

When you login `.st2` directory will be created. It will contain the st2 user configuration with the authentication token.

```bash
ls -al .st2
-rw-r--r-- 1 vagrant vagrant   35 Sep 23 12:56 config
-rw-rw---- 1 vagrant vagrant   77 Sep 23 12:56 token-st2admin
```

### User Management

The default admin user `st2admin` credentials are stored in `/etc/st2/htpasswd` file.

```bash
ls -la /etc/st2/htpasswd
-rw------- 1 st2 st2 47 Sep 22 11:41 /etc/st2/htpasswd
```

As you can see this file is owned by `st2` service account with nologin. In order to update this password, login as `root` user and change the password.

```bash
sudo su
htpasswd /etc/st2/htpasswd st2admin
New password:
Re-type new password:
Updating password for user st2admin
```

## Packs

As mentioned in the [Architecture](#architecture) section a `pack` is a group of integration and automation tasks that are related to some service or infrastructure provider.

Imagine that you need to automate a docker implementation. You need to define tasks which you want to automate such as:

- Status of Docker
- Start Docker
- Stop Docker
- Restart Docker
- Clean up /var/log

You can aggregate the above tasks into `Actions` and execute them manuall or based on events such as:

- Monitor the Status of Docker (if down start it)
- Monitor the disk space (if low clean up logs)

You can classify these two monitor tasks as `Sensors` as they are monitoring the managed hosts.

Finally, you can implement additional logic based on conditions by using `workflows`.

- Start Docker if it is not running
- Stop Docker if it is running

In summary a pack may contain the following folders and files.

- actions
- workflows
- sensors
- rules
- pack.yml

### List packs

StackStorm contains some default packs such as:

```bash
ls -l /opt/stackstorm/packs/
total 24
drwxrwxr-x 5 root st2packs 4096 Sep 22 11:40 chatops
drwxrwxr-x 5 root st2packs 4096 Sep 22 11:40 core
drwxrwxr-x 5 root st2packs 4096 Sep 22 11:40 default
drwxrwxr-x 5 root st2packs 4096 Sep 22 11:40 linux
drwxrwxr-x 5 root st2packs 4096 Sep 22 11:40 packs
drwxrwxr-x 8 root st2packs 4096 Sep 22 11:42 st2

st2 pack list
+---------+---------+-----------------------------+---------+------------------+
| ref     | name    | description                 | version | author           |
+---------+---------+-----------------------------+---------+------------------+
| chatops | chatops | ChatOps integration pack    | 3.5.0   | StackStorm, Inc. |
| core    | core    | Basic core actions.         | 3.5.0   | StackStorm, Inc. |
| default | default | Default pack where all      | 3.5.0   | StackStorm, Inc. |
|         |         | resources created using the |         |                  |
|         |         | API with no pack specified  |         |                  |
|         |         | get saved.                  |         |                  |
| linux   | linux   | Generic Linux actions       | 3.5.0   | StackStorm, Inc. |
| packs   | packs   | Pack management             | 3.5.0   | StackStorm, Inc. |
|         |         | functionality.              |         |                  |
| st2     | st2     | StackStorm utility actions  | 2.0.1   | StackStorm, Inc. |
|         |         | and aliases                 |         |                  |
+---------+---------+-----------------------------+---------+------------------+
```

By default output is formatted as table, you have option to view it as json or yaml with `--json` or `--yaml` arguments.

### Install and Uninstall packs

To install a pack.

```bash
st2 pack search aws
+-----------+------------------------------------+---------+------------------+
| name      | description                        | version | author           |
+-----------+------------------------------------+---------+------------------+
| aws       | st2 content pack containing Amazon | 2.0.1   | StackStorm, Inc. |
|           | Web Services integrations.         |         |                  |
| libcloud  | st2 content pack containing        | 0.6.0   | StackStorm, Inc. |
|           | libcloud integrations              |         |                  |
| aws_boto3 | AWS actions using boto3            | 1.0.0   | StackStorm, Inc. |
| aws_s3    | AWS S3-specific actions            | 2.0.3   | StackStorm, Inc. |
+-----------+------------------------------------+---------+------------------+

st2 pack install aws
For the "aws" pack, the following content will be registered:

actions   |  3581
rules     |  0
sensors   |  2
aliases   |  3
triggers  |  0

Installation may take a while for packs with many items.

        [ succeeded ] init_task
        [ succeeded ] download_pack
        [ succeeded ] make_a_prerun
        [ succeeded ] get_pack_dependencies
        [ succeeded ] check_dependency_and_conflict_list
        [ succeeded ] install_pack_requirements
        [ succeeded ] get_pack_warnings
        [ succeeded ] register_pack

+-------------+--------------------------------------------------------------+
| Property    | Value                                                        |
+-------------+--------------------------------------------------------------+
| ref         | aws                                                          |
| name        | aws                                                          |
| description | st2 content pack containing Amazon Web Services              |
|             | integrations.                                                |
| version     | 2.0.1                                                        |
| author      | StackStorm, Inc.                                             |
+-------------+--------------------------------------------------------------+
```

To uninstall a pack.

```bash
st2 pack remove aws

        [ succeeded ] unregister packs
        [ succeeded ] delete packs

+-------------+--------------------------------------------------------------+
| Property    | Value                                                        |
+-------------+--------------------------------------------------------------+
| ref         | aws                                                          |
| name        | aws                                                          |
| description | st2 content pack containing Amazon Web Services              |
|             | integrations.                                                |
| version     | 2.0.1                                                        |
| author      | StackStorm, Inc.                                             |
+-------------+--------------------------------------------------------------+
```


### Pack information

Information about an installed pack.

```bash
st2 pack get aws
+-------------+--------------------------------------------------------------+
| Property    | Value                                                        |
+-------------+--------------------------------------------------------------+
| name        | aws                                                          |
| version     | 2.0.1                                                        |
| author      | StackStorm, Inc.                                             |
| email       | info@stackstorm.com                                          |
| keywords    | [                                                            |
|             |     "aws",                                                   |
|             |     "amazon web services",                                   |
|             |     "amazon",                                                |
|             |     "ec2",                                                   |
|             |     "sqs",                                                   |
|             |     "sns",                                                   |
|             |     "route53",                                               |
|             |     "cloud",                                                 |
|             |     "iam",                                                   |
|             |     "vpc",                                                   |
|             |     "s3",                                                    |
|             |     "CloudFormation",                                        |
|             |     "RDS",                                                   |
|             |     "SQS",                                                   |
|             |     "lambda",                                                |
|             |     "kinesis"                                                |
|             | ]                                                            |
| description | st2 content pack containing Amazon Web Services              |
|             | integrations.                                                |
+-------------+--------------------------------------------------------------+
```

Information about an installed or available pack at exchange.

```bash
st2 pack show terraform
+-----------------+-------------------------------------------------------------+
| Property        | Value                                                       |
+-----------------+-------------------------------------------------------------+
| name            | terraform                                                   |
| description     | Terraform integrations                                      |
| author          | Martez Reed                                                 |
| content         | {                                                           |
|                 |     "actions": {                                            |
|                 |         "count": 13,                                        |
|                 |         "resources": [                                      |
|                 |             "apply",                                        |
|                 |             "create_workspace",                             |
|                 |             "delete_workspace",                             |
|                 |             "destroy",                                      |
|                 |             "get_version",                                  |
|                 |             "import_object",                                |
|                 |             "init",                                         |
|                 |             "list_workspaces",                              |
|                 |             "output",                                       |
|                 |             "pipeline",                                     |
|                 |             "plan",                                         |
|                 |             "select_workspace",                             |
|                 |             "show"                                          |
|                 |         ]                                                   |
|                 |     },                                                      |
...
[ Output omitted  ]
...
```


## Actions

Actions are pieces of code or script that can perform automation. They can be written in any programming language. 

Usually actions are part of a pack, but there may be cases that they are deployed without pack, therefore they will be part of `default` pack.

### List actions

In order to list available actions use the `st2 action list` command. As you can see each action has a prefix of pack name and short description.

```bash
st2 action list
+---------------------------------+---------+------------------------------------------------+
| ref                             | pack    | description                                    |
+---------------------------------+---------+------------------------------------------------+
| chatops.format_execution_result | chatops | Format an execution result for chatops         |
| chatops.match                   | chatops | Match a string to an action alias              |
| chatops.match_and_execute       | chatops | Execute a chatops string to an action alias    |
| chatops.post_message            | chatops | Post a message to stream for chatops           |
| chatops.post_result             | chatops | Post an execution result to stream for chatops |
| chatops.run                     | chatops | Match a text chatops command, execute it and   |
|                                 |         | post the result                                |
| core.announcement               | core    | Action that broadcasts the announcement to all |
|                                 |         | stream consumers.                              |
| core.ask                        | core    | Action for initiating an Inquiry (usually in a |
...
[ Output omitted ]
...
```

To get more information about a particular action, use the `get` argument. This is useful for retrieving the required parameters.

```bash
st2 action get core.echo
+---------------+--------------------------------------------------------------+
| Property      | Value                                                        |
+---------------+--------------------------------------------------------------+
| id            | 614b1676586f7adc0a29e849                                     |
| uid           | action:core:echo                                             |
| ref           | core.echo                                                    |
| pack          | core                                                         |
| name          | echo                                                         |
| description   | Action that executes the Linux echo command on the           |
|               | localhost.                                                   |
| enabled       | True                                                         |
| entry_point   |                                                              |
| runner_type   | local-shell-cmd                                              |
| parameters    | {                                                            |
|               |     "message": {                                             |
|               |         "description": "The message that the command will    |
|               | echo.",                                                      |
|               |         "type": "string",                                    |
|               |         "required": true                                     |
|               |     },                                                       |
|               |     "cmd": {                                                 |
|               |         "description": "Arbitrary Linux command to be        |
|               | executed on the local host.",                                |
|               |         "required": true,                                    |
|               |         "type": "string",                                    |
|               |         "default": "echo "{{message}}"",                     |
|               |         "immutable": true                                    |
|               |     },                                                       |
|               |     "kwarg_op": {                                            |
|               |         "immutable": true                                    |
|               |     },                                                       |
|               |     "sudo": {                                                |
|               |         "default": false,                                    |
|               |         "immutable": true                                    |
|               |     },                                                       |
|               |     "sudo_password": {                                       |
|               |         "immutable": true                                    |
|               |     }                                                        |
|               | }                                                            |
| metadata_file | actions/echo.yaml                                            |
| notify        |                                                              |
| output_schema |                                                              |
| tags          |                                                              |
+---------------+--------------------------------------------------------------+
```

### Execute actions

In order to execute action manually, you can use the follwing methods:
- Web UI 
- CLI
- API

#### Web UI

Executing actions from Web UI is easy. Just navigate to **Actions** page, expand the required pack, select and action. Fill out required parameters and hit **Run**.


#### CLI

You can also execute action using CLI using `st2` with `action execute` or `run` arguments as shown in examples below which executes command on local server.

```bash
#
# Execute an action
#
st2 action execute core.local cmd="st2 --version"
To get the results, execute:
 st2 execution get 614d8b87f843af4f37850851

To view output in real-time, execute:
 st2 execution tail 614d8b87f843af4f37850851

#
# View result from execution
#
st2 execution get 614d8b87f843af4f37850851
id: 614d8b87f843af4f37850851
action.ref: core.local
context.user: st2admin
parameters:
  cmd: st2 --version
status: succeeded (0s elapsed)
start_timestamp: Fri, 24 Sep 2021 08:25:43 UTC
end_timestamp: Fri, 24 Sep 2021 08:25:43 UTC
log:
  - status: requested
    timestamp: '2021-09-24T08:25:43.409000Z'
  - status: scheduled
    timestamp: '2021-09-24T08:25:43.538000Z'
  - status: running
    timestamp: '2021-09-24T08:25:43.569000Z'
  - status: succeeded
    timestamp: '2021-09-24T08:25:43.846000Z'
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: st2 3.5.0, on Python 3.8.10
  succeeded: true 
```

This is useful when you have long running executions. You can simulate this using `sleep` utility.

```bash
st2 action execute core.local cmd="sleep 30"
To get the results, execute:
 st2 execution get 614d8f3cf843af4f37850857

To view output in real-time, execute:
 st2 execution tail 614d8f3cf843af4f37850857

#
# The action is still running
#
st2 execution get 614d8f3cf843af4f37850857
id: 614d8f3cf843af4f37850857
action.ref: core.local
context.user: st2admin
parameters:
  cmd: sleep 30
status: running (4s elapsed)
start_timestamp: Fri, 24 Sep 2021 08:41:32 UTC
end_timestamp:
log:
  - status: requested
    timestamp: '2021-09-24T08:41:32.023000Z'
  - status: scheduled
    timestamp: '2021-09-24T08:41:32.133000Z'
  - status: running
    timestamp: '2021-09-24T08:41:32.176000Z'
result: None 
```

```bash
#
# Execute an action
#
st2 run core.local cmd="st2 --version"
.
id: 614d8bacf843af4f37850854
action.ref: core.local
context.user: st2admin
parameters:
  cmd: st2 --version
status: succeeded
start_timestamp: Fri, 24 Sep 2021 08:26:20 UTC
end_timestamp: Fri, 24 Sep 2021 08:26:21 UTC
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: st2 3.5.0, on Python 3.8.10
  succeeded: true
```

In some case a sudo privilege is required in order to execute a command. Observe the difference between these two outputs.

```
st2 run core.local cmd='id'
.
id: 614d92fbf843af4f37850872
action.ref: core.local
context.user: st2admin
parameters:
  cmd: id
status: succeeded
start_timestamp: Fri, 24 Sep 2021 08:57:31 UTC
end_timestamp: Fri, 24 Sep 2021 08:57:31 UTC
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: uid=1001(stanley) gid=1001(stanley) groups=1001(stanley)
  succeeded: true


st2 run core.local_sudo cmd='id'
.
id: 614d92daf843af4f3785086f
action.ref: core.local_sudo
context.user: st2admin
parameters:
  cmd: id
status: succeeded
start_timestamp: Fri, 24 Sep 2021 08:56:58 UTC
end_timestamp: Fri, 24 Sep 2021 08:56:58 UTC
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: uid=0(root) gid=0(root) groups=0(root)
  succeeded: true
```

#### Rest API

To execute actions using REST API available at `https://<stackstorm-host>:443`. The endpoints are available at `/api/v1/<service_to_work>`. Before you can interact with these endpoints you need to authenticate using either `St2-Api-Key` or `X-Auth-Token`.

To genearate an API key, you need to execute the following command from StackStorm Engine.

```bash
st2 apikey create -k -m '{"used_by": "my integration"}'
OKO04N2VkOWU5ZlMzdhMTNMWYyNGUzODRjTM0ZDVmmQxMzdZmUyMzZlMmVjYZjN2VkYzNiZmjN2VkYzNiZmUys
```

To execute action from remote host using API.

```bash
curl -k -X POST http://<STACKSTORM-HOST>:443/api/v1/executions \
     -H "St2-Api-Key:<API-KEY-VALUE>" \
     -H "content-type:application/json" \
     --data-binary '{"action": "core.local", "parameters": {"cmd": "uptime"}, "user": null}'

{"action":{"tags":[],"uid":"action:core:local","metadata_file":"actions/local.yaml","name":"local","ref":"core.local","description":"Action that executes an arbitrary Linux command on the localhost.","enabled":true,"entry_point":"","pack":"core","runner_type":"local-shell-cmd","parameters":{"cmd":{"description":"Arbitrary Linux command to be executed on the local host.","required":true,"type":"string"},"sudo":{"immutable":true}},"output_schema":{},"notify":{},"id":"614b1677586f7adc0a29e84d"},"runner":{"name":"local-shell-cmd","description":"A runner to execute local actions as a fixed user.","uid":"runner_type:local-shell-cmd","enabled":true,"runner_package":"local_runner","runner_module":"local_shell_command_runner","runner_parameters":{"cmd":{"description":"Arbitrary Linux command to be executed on the host.","type":"string"},"cwd":{"description":"Working directory where the command will be executed in","type":"string"},"env":{"description":"Environment variables which will be available to the command(e.g. key1=val1,key2=val2)","type":"object"},"kwarg_op":{"default":"--","description":"Operator to use in front of keyword args i.e. \"--\" or \"-\".","type":"string"},"sudo":{"default":false,"description":"The command will be executed with sudo.","type":"boolean"},"sudo_password":{"default":null,"description":"Sudo password. To be used when passwordless sudo is not allowed.","type":"string","secret":true,"required":false},"timeout":{"default":60,"description":"Action timeout in seconds. Action will get killed if it doesn't finish in timeout seconds.","type":"integer"}},"output_schema":{},"id":"614b166ecbde329ef4ac8eb8"},"liveaction":{"action":"core.local","action_is_workflow":false,"parameters":{"cmd":"uptime"},"callback":{},"runner_info":{},"id":"614dbe14a500afc4b5a4ab80"},"status":"requested","start_timestamp":"2021-09-24T12:01:24.013495Z","parameters":{"cmd":"uptime"},"context":{"user":"st2admin","pack":"core"},"log":[{"timestamp":"2021-09-24T12:01:24.019400Z","status":"requested"}],"web_url":"https://stackstorm/#/history/614dbe14a500afc4b5a4ab81/general","id":"614dbe14a500afc4b5a4ab81"}
```

To retrieve task execution status using execution id.

```bash
curl -k -s -X GET http://<STACKSTORM-HOST>:443/api/v1/executions/614dbe14a500afc4b5a4ab81 \
     -H "St2-Api-Key:<API-KEY-VALUE>" \
     -H "content-type:application/json" \
     | jq -r '.result.stdout'
12:01:24 up 1 day,  7:06,  1 user,  load average: 0.14, 0.16, 0.16
```

### List execution

In order to retrieve the list of past executions use the `st2` command with `execution list` argument.

```bash
st2 execution list
+--------------------------+---------------+--------------+-------------------------+-----------------+---------------+
| id                       | action.ref    | context.user | status                  | start_timestamp | end_timestamp |
+--------------------------+---------------+--------------+-------------------------+-----------------+---------------+
| 614b168f18f0dd6e8c03e048 | core.local    | st2admin     | succeeded (1s elapsed)  | Wed, 22 Sep     | Wed, 22 Sep   |
|                          |               |              |                         | 2021 11:42:07   | 2021 11:42:08 |
|                          |               |              |                         | UTC             | UTC           |
| 614b169218f0dd6e8c03e04b | core.remote   | st2admin     | succeeded (1s elapsed)  | Wed, 22 Sep     | Wed, 22 Sep   |
|                          |               |              |                         | 2021 11:42:10   | 2021 11:42:11 |
|                          |               |              |                         | UTC             | UTC           |
```


## Pack development

Before developing a custom pack, there are minimum requiremnets which include knowledge of:
- YAML
- Jinja2
- Shell commands and its exit status
- Shell scripting
- Python scripting
- YAQL

### YAML

- YAML stands for 'Ain't Markup Language
- YAML is a human-readable data serialization language
- It is commonly used for configuration files and in applications where data is being stored or transmitted
- Its very simple and easy to learn
- Common applications that leverage this language is Ansible, Kubernetes, Salt and StackStorm
- YAML files are made up of three components:
  - Scalars (e.g. file: config)
  - Arrays/Lists
  - Mappings
- Every YAML file should start with `---` and by default each YAML file can be represented as dictionary in Python

### Jinja2

- Jinja2 is a template engine used to produce data dynamically from data serialization language like YAML or JSON
- You can create a template engine, where you define static and dynamic data
- During execution the dynamic data is rendered with required login during execution (vars, conditions, loops, filters)
- Jinja2 supports two ways to implement templates:
  - As a file (.j2)
  - As a data (inside a yaml file)
- Jinja2 supports the following dilimeters:
  - `{{ }}` - used for expressions (variables)
  - `{% %}` - used for control statements (conditionals, loops)
  - `{# .. #}` - used for comments
  - `# ##` - used for line statements

For example, you have a variable defined in `yaml` file as follows:

```yml
name: stackstorm is an event driven automation tool
```

To render the content of variable in j2 you would referenced it as follows.

```j2
{{ name }}
```

#### String operations

You can also use number of string operations such as `upper`, `lower`, `title` using pipeline with following syntax.

```j2
{{ name | upper }}
{{ name | lower }}
{{ name | title }}
```

These operations are equivalent to following:

```j2
{{ name.upper() }}
{{ name.lower() }}
{{ name.title() }}
```

If you try to render a value for nonexisting variable, you will receive an error. You can however define a default value.

```j2
{{ license | default('Apache 2.0') }}
```

#### Arithmetic operations

Jinja2 supports arithmetic operations such as additions, subtraction, multiplication, division.

```yml
---
x: 5
y: 10
```

```j2
# The following statement renders as 15
{{ x + y }}
```

#### Comparison operations

```yml
---
x: 5
y: 10
```

```j2
# The following statement renders False
{{ x == y }}

# The following statement renders True
{{ x != y}}
```

#### Membership operations

```yml
---
x: stackstorm is an event driven automation tool
y: tool
mylist:
  - 6
  - 9
  - 10
l: 9
n: 78
```

```j2
# The following statement renders True
{{ y in x }}

# The following statement renders False
{{ n in mylist }}
```

#### Define a variable in Jinja

Use the `set` statement to define a variable in Jinja/Jinja Template.

```yml
---
x: 5
y: 10
```

```j2
# Create a new variable result
{% set result=x + y %}

# Print the content of result
{{ result }}
```

#### Array (List) and Map (Dictionary) operations

Define sample arrays and maps:

```yml
---
myfirstlist: [1, 2, 3, 7]
mysecondlist:
  - 1
  - 2
  - 3
  - 7
myfirstmap: {one: 1, two: 2, three: 3, seven: 7}
mysecondmap:
  one: 1
  two: 2
  three: 3
  sevent: 7
```


```j2
# The following statement renders content of myfirstlist
# [1, 2, 3, 7]
{{ myfirstlist }}

# The following statement renders content of mysecondlist
# [1, 2, 3, 7]
{{ mysecondlist }}

# The following statement renders first value from myfirstlist
# 1
{{ myfirstlist[0] }}

# The following statement renders lenght of myfirstlist
# 4
{{ myfirstlist | length }}
```

```j2
# The following statement renders content of myfirstmap
# {'one': 1, 'two': 2, 'three': 3, 'seven': 7}
{{ myfirstmap }}

# The following statement renders content of mysecondmap
# {'one': 1, 'two': 2, 'three': 3, 'seven': 7}
{{ mysecondmap }}

# The following statement renders all keys from myfirstmap
# ['one', 'two', 'three', 'seven']
{{ myfirstmap.keys() }}

# The following statement renders all values from myfirstmap
# [1, 2, 3, 7]
{{ myfirstmap.values() }}

# The following statement renders all items from myfirstmap
# [('one', 1), ('two', 2), ('three', 3), ('seven', 7)]
{{ myfirstmap.items() }}
```

#### Conditional and loop operations

Syntax for simple `if` condition looks as follows:

```j2
{% if condition %}
 block for logic
{% endif %}
```

Syntax for simple `if-else` condition looks like follows:
```j2
{% if condition %}
 block for logic-1
{% else %}
 block for logic-2
{% endif %}
```

Syntax for inline `if-else` statement:

```j2
{{ "block for logic-1" if condition esle "block for logic-2" }}
```

Syntax for `if-elif-else` condition:

```j2
{% if condition1 %}
 block for logic-1
{% elif condition2 %}
 block for logic-2
{% else %}
 block for logic-3
{% endif %}
```

Syntax for `for loop` with list:

```j2
{% for each in mylist %}
{% endfor %}
```

Syntax for `for loop` with dictionary items:

```j2
{% for key, value in mydict.items() %}
{% endfor %}
```

### Creating custom pack

As mentioned in [Packs](#packs) section, each pack has a predifined directory structure, such as:

- pack.yml
- actions
  - workflows
- sensors
- rules
- requirements.txt etc...

#### Folder structure 

Start by creating an empty pack folder `dummy`:

```bash
# Create pack folder
mkdir -p ~/packs/dummy && cd $_

# Create folder structure and required files
mkdir actions rules sensors aliases policies
touch pack.yaml requirements.txt

# Update content of pack.yaml
cat <<EOF >>pack.yaml
---
ref: dummy
name: dummy
description: Simple dummy pack without sensor, rule, and action
keywords:
    - example
    - test
version: 3.5.0
python_versions:
  - "2"
  - "3"
author: Maros Kukan
email: maros.kukan@example.com
contributors:
  - "John Doe1 <john.doe1@gmail.com>"
  - "John Doe2 <john.doe2@gmail.com>"
EOF
```

Initialize git repository and install the pack.

```bash
# Initliaze git repository
git init && git add ./* && git commit -m "Initial commit"

# Install local pack
st2 pack install file://$PWD

        [ succeeded ] init_task
        [ succeeded ] download_pack
        [ succeeded ] make_a_prerun
        [ succeeded ] get_pack_dependencies
        [ succeeded ] check_dependency_and_conflict_list
        [ succeeded ] install_pack_requirements
        [ succeeded ] get_pack_warnings
        [ succeeded ] register_pack

+-------------+----------------------------------------------------+
| Property    | Value                                              |
+-------------+----------------------------------------------------+
| ref         | dummy                                              |
| name        | dummy                                              |
| description | Simple dummy pack without sensor, rule, and action |
| version     | 3.5.0                                              |
| author      | Maros Kukan                                        |
+-------------+----------------------------------------------------+
```

Verify that the dummy pack is listed in available packs:

```bash
st2 pack get dummy
+-------------+----------------------------------------------------+
| Property    | Value                                              |
+-------------+----------------------------------------------------+
| name        | dummy                                              |
| version     | 3.5.0                                              |
| author      | Maros Kukan                                        |
| email       | maros.kukan@example.com                            |
| keywords    | [                                                  |
|             |     "example",                                     |
|             |     "test"                                         |
|             | ]                                                  |
| description | Simple dummy pack without sensor, rule, and action |
+-------------+----------------------------------------------------+
```

#### Actions

Actions are used to execute code (shell script, python script, shell command) that can perform automation in our environment. The leverage action runners on backend.

Action Runners are the environment to execute user-implemented actions. There are multiple action runners available in default installation, such as:
- local-shell-cmd
- local-shell-script
- remote-shell-cmd
- remote-shell-script
- python-script
- [...](https://docs.stackstorm.com/actions.html#available-runners)

Normally, the exit code of a runner is defined by the exit code of the script or command executed. All runners return timeout exit code (-9) if the command or script did not complete its execution within the specified timeout.

Actions are made up from:
- Action Meta Data file or yaml file - mandatory
- Script (Shell, Python, or any other) - optional

Script will have our logic and meta data file will have the info about an action. Script should return valid stdout and stderr.

In order to demostrate usage of actions, create a new `math` pack.

```bash
# Create pack folder
mkdir -p ~/packs/math && cd $_

# Create folder structure and required files
mkdir actions rules sensors aliases policies
touch pack.yaml requirements.txt

# Update content of pack.yaml
cat <<EOF >>pack.yaml
---
ref: math
name: math
description: This pack is used to find the add and sub of two numbers.
keywords:
    - calculator
    - addition
    - subtraction
version: 3.5.0
python_versions:
  - "2"
  - "3"
author: Maros Kukan
email: maros.kukan@example.com
contributors:
  - "John Doe1 <john.doe1@gmail.com>"
  - "John Doe2 <john.doe2@gmail.com>"
EOF
```

Next, create action metadata and script for `addition` task.

```bash
# Addition action medatada
cat <<EOF >>./actions/add.yaml
---
name: add
description: This is aan addition action.
runner_type: local-shell-script
enabled: true
entry_point: "add.sh"
parameters:
  num1:
    type: integer
  num2:
    type: integer
EOF

# Addition action script
cat <<\EOF >>./actions/add.sh
#!/usr/bin/env bash

num1=$1
num2=$2
sum=$((num1+num2))
echo "The addition of $num1 and $num2 is: $sum"
EOF
```

Next, create action metadata nad script for `subtraction` task.

```bash
# Subtraction action medatada
cat <<EOF >>./actions/sub.yaml
---
name: sub
description: This is a subtraction action.
runner_type: local-shell-script
enabled: true
entry_point: "sub.sh"
parameters:
  num1:
    type: integer
  num2:
    type: integer
EOF

# Subtraction action script
cat <<\EOF >>./actions/sub.sh
#!/usr/bin/env bash

num1=$1
num2=$2
sum=$((num1-num2))
echo "The addition of $num1 and $num2 is: $sum"
EOF
```

Initialize git repository and install the pack.

```bash
# Initliaze git repository
git init && git add ./* && git commit -m "Initial commit"

# Install local pack
st2 pack install file://$PWD
```

Verify newly deployed pack.

```bash
st2 action list -p math
+----------+------+-------------------------------+
| ref      | pack | description                   |
+----------+------+-------------------------------+
| math.add | math | This is an addition action.   |
| math.sub | math | This is a subtraction action. |
+----------+------+-------------------------------+
```

Every time you perform a change in your metadata or script file, you need to redeploy the pack.

To run your action, invoke it with `st2 run` or `st2 action execute`.

```bash
st2 run math.add
.
id: 6151be08959864546745ac9c
action.ref: math.add
context.user: st2admin
parameters: None
status: succeeded
start_timestamp: Mon, 27 Sep 2021 12:50:16 UTC
end_timestamp: Mon, 27 Sep 2021 12:50:16 UTC
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: 'The addition of  and  is: 0'
  succeeded: true
```

As you can see, the action execute successfully, eventhough we did not provided any parameters. We need to update the action metadata to require `num1` and `num2` in `add.yaml`.

```yml
---
name: add
description: This is an addition action.
runner_type: local-shell-script
enabled: true
entry_point: "add.sh"
parameters:
  num1:
    type: integer
    required: true
  num2:
    type: integer
    required: true
```

Remember, when using `git` always make sure you have latest changes commited before reinstall the pack.

```bash
git add ./* && git commit -m "Add required: true for both parameters"
st2 pack install file://$PWD
```

Now, when you run the `math.add` action without any arguments, you will receive an error.

```bash
# Running action without argument will fail
st2 run math.add
ERROR: 400 Client Error: Bad Request
MESSAGE: 'num1' is a required property for url: http://127.0.0.1:9101/v1/executions

# Running action with arguments
st2 run math.add num1=5 num2=7
.
id: 6151c2de959864546745acab
action.ref: math.add
context.user: st2admin
parameters:
  num1: 5
  num2: 7
status: succeeded
start_timestamp: Mon, 27 Sep 2021 13:10:54 UTC
end_timestamp: Mon, 27 Sep 2021 13:10:54 UTC
result:
  failed: false
  return_code: 0
  stderr: '/opt/stackstorm/packs/math/actions/add.sh: line 5: --num2=7: expression recursion level exceeded (error token is "num2=7")'
  stdout: 'The addition of --num2=7 and --num1=5 is: '
  succeeded: true
```

Better, but this is still now what we want to see as result. You need to define the position of required arguments in metadata file.

```yml
---
name: add
description: This is an addition action.
runner_type: local-shell-script
enabled: true
entry_point: "add.sh"
parameters:
  num1:
    type: integer
    required: true
    position: 0
  num2:
    type: integer
    required: true
    position: 1
```

Commit the new change, reinstall and test the action once again.

```bash
# Commit change and reinstall the pack
git add ./* && git commit -m "Add position for both parameters"
st2 pack install file://$PWD

# Test the additional action
st2 run math.add num1=5 num2=10
.
id: 6151c4ab959864546745acb1
action.ref: math.add
context.user: st2admin
parameters:
  num1: 5
  num2: 10
status: succeeded
start_timestamp: Mon, 27 Sep 2021 13:18:35 UTC
end_timestamp: Mon, 27 Sep 2021 13:18:35 UTC
result:
  failed: false
  return_code: 0
  stderr: ''
  stdout: 'The addition of 5 and 10 is: 15'
  succeeded: true
```

More pack examples are available at `packs` directory.


## Workflows

Workflow is a way to chain multiple actions based on your requirements and environment. It supports if, then, else logic. For example in order to orchestrate nginx service:

- Start Nginx Service
  - Task1: is nginx installed?
  - Task2: is it already running?
  - Task3: if not then start it

StackStorm uses the Orquesta language to define a workflow.

```bash
st2 run core.remote cmd='which nginx' hosts='u1'
..
id: 61556e4ae010e55432fa1b40
action.ref: core.remote
context.user: st2admin
parameters:
  cmd: which nginx
  hosts: u1
status: failed
start_timestamp: Thu, 30 Sep 2021 07:59:06 UTC
end_timestamp: Thu, 30 Sep 2021 07:59:09 UTC
result:
  u1:
    failed: true
    return_code: 1
    stderr: ''
    stdout: ''
    succeeded: false
```

### Orquesta Graph Editor

```bash
git clone git@github.com:maroskukan/rehearsal.git
cd rehearsal
git submodule init && git submodule update
make up
```

One your browser and navigate to `http://localhost:5000/`.

### Creating workflow

Workflow is also an action, therefore you need to create a meta data file and a script file or workflow file under `workflow` directory.

Each Workflow file consists of version, description, input, vars, tasks, output.

### Workflow Functions

There are three runtime functions of Orquesta Workflow.
- `ctx()`
- `task()`
- `result()`

The `ctx()` function is used to retrieve variable value during execution. For example
`<% ctx(xyz) %>` or `<% ctx().xyz %>`.

The `task()` function is used to retrieve result from tasks. For example `<% task(ExecuteCommand) %>`.

Once you run this workflow action the output will look like following:

```yaml
result:
      end_timestamp: '2021-09-30 11:20:29.906046+00:00'
      result:
        failed: false
        return_code: 0
        stderr: ''
        stdout: Thu 30 Sep 2021 11:20:29 AM UTC
        succeeded: true
      route: 0
      start_timestamp: '2021-09-30 11:20:29.589346+00:00'
      status: succeeded
      task_execution_id: 61559d7d5cb2f10bec65b765
      task_id: ExecuteCommand
      task_name: ExecuteCommand
      workflow_execution_id: 61559d7dc553d9751089e3c8
```
The `result()` function is used within an action to capture output inside a variable for later processing.

```yaml
[ Text omitted ]
    next:
      - publish:
          - outputtask1: <% result() %>
[ Text omitted ]
output:
  - outputtask1: <% ctx(outputtask1) %>
```

Once you run this workflow action the output will look like following:

```yaml
outputtask1:
      failed: false
      return_code: 0
      stderr: ''
      stdout: Thu 30 Sep 2021 11:20:29 AM UTC
      succeeded: true
```

### Workflow Summary

- Orquesta Workflow is also an action but runner type is Orquesta
- Therefore we have meta data file and script file
- Writing meta data file with Orquesta runner is same as writing meta data file for other runners
- Next Important Part is: writing a script / workflow file
- Key points to remember for workflow file are:
  - input, vars, publish and output are the places to define variables and parameters
    - input section is to define variables and parametes which are in meta data file
    - vars section is to define intermediate variables
    - publish section is to define variables in tasks
    - output section is to dipslay variable values


