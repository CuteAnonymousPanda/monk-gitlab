Gitlab meets Monk.io
===

This repository contains Monk.io template to deploy Gitlab and its runners either locally or on cloud of your choice (AWS, GCP, Azure, Digital Ocean).  

  - [Prerequisites](#prerequisites)
    - [Make sure monkd is running.](#make-sure-monkd-is-running)
    - [Clone Repository](#clone-repository)
    - [Load Template](#load-template)
    - [Verify if it's loaded correctly](#verify-if-its-loaded-correctly)
  - [Deploy Stack](#deploy-stack)
  - [Variables](#variables)
  - [Register a runner with your instance](#register-a-runner-with-your-instance)
  - [Stop, remove and clean up workloads and templates](#stop-remove-and-clean-up-workloads-and-templates)
  - [Create your own template](#create-your-own-template)

## Prerequisites
- [Install Monk](https://docs.monk.io/docs/get-monk)
- [Register and Login Monk](https://docs.monk.io/docs/acc-and-auth)
- [Add Cloud Provider](https://docs.monk.io/docs/cloud-provider)
- [Add Instance](https://docs.monk.io/docs/multi-cloud)

### Make sure monkd is running.

``` bash
$ monk status
daemon: ready
auth: logged in
not connected to cluster
```

### Clone Repository

``` bash
$ git clone git@github.com:CuteAnonymousPanda/monk-gitlab.git
```

### Load Template

``` bash
$ cd monk-gitlab
$ monk load manifest.yaml
```

### Verify if it's loaded correctly

```bash
$ monk list -l gitlab

✔ Got the list
Type      Template       Repository  Version  Tags  
runnable  gitlab/runner  local       -        -     
runnable  gitlab/server  local       -        -     
group     gitlab/stack   local       -        -     
```

## Deploy Stack

```bash
$ monk run gitlab/stack
✔ Starting the job: local/gitlab/stack... DONE
✔ Preparing nodes DONE
✔ Checking/pulling images...
✔ [================================================] 100% docker.io/gitlab/gitlab-ce:latest QmR5a3SXMKwhYHfcZjpVFWycrcy2TmUGAuEosFdsGL5oVn
✔ [================================================] 100% docker.io/gitlab/gitlab-runner:alpine QmR5a3SXMKwhYHfcZjpVFWycrcy2TmUGAuEosFdsGL5oVn
✔ Checking/pulling images DONE
✔ Starting containers DONE
✔ Starting containers DONE
✔ New container local-46a6f63de3a71470b28f440e45-local-gitlab-server-app created DONE
✔ Started local/gitlab/stack

🔩 templates/local/gitlab/stack             
 └─🧊 Peer QmR5a3SXMKwhYHfcZjpVFWycrcy2TmUGAuEosFdsGL5oVn
    ├─🔩 templates/local/gitlab/runner 
    │  └─📦 local-c616fa5ce83ef91e4b43cb5f27-local-gitlab-runner-runner 
    │     └─🧩 docker.io/gitlab/gitlab-runner:alpine
    └─🔩 templates/local/gitlab/server 
       └─📦 local-46a6f63de3a71470b28f440e45-local-gitlab-server-app 
          ├─🧩 docker.io/gitlab/gitlab-ce:latest                 
          ├─💾 /var/lib/monkd/volumes/gitlab-data/logs -> /var/log/gitlab
          ├─💾 /var/lib/monkd/volumes/gitlab-data/data -> /var/opt/gitlab
          ├─💾 /var/lib/monkd/volumes/gitlab-data/config -> /etc/gitlab
          ├─🔌 open 1.1.1.2:80 -> 80              
          ├─🔌 open 1.1.1.2:23 -> 22              
          ├─🔌 open 1.1.1.2:443 -> 443            
          └─🔌 open 1.1.1.2:5050 -> 5050          

💡 You can inspect and manage your above stack with these commands:
        monk logs (-f) local/gitlab/stack - Inspect logs
        monk shell     local/gitlab/stack - Connect to the container's shell
        monk do        local/gitlab/stack/action_name - Run defined action (if exists)
💡 Check monk help for more!
```

## Variables

The variables are stored in `manifest.yaml` file.  
You can quickly setup by editing the values there.

| Variable                     | Description                                 | Default                    |
|------------------------------|---------------------------------------------|----------------------------|
| ssh-port                     | Port to listen for incoming ssh connections | 22                         |
| gitlab-url                   | Your Gitlab instance address                | http://${your-external-ip} |
| gitlab-root-password         | Default root password                       | z123z123                   |
  

## Register a runner with your instance

To register a runner please:
1) Obtain a runner registration token from UI Admin/Runner section
2) Run following command with appropriate parameters:

``` bash
monk do gitlab/runner/register token=<Token> url=<Your_Gitlab_URL>
```

## Stop, remove and clean up workloads and templates

```bash
monk purge    gitlab/server gitlab/runner
monk purge -x gitlab/server gitlab/runner
```

## Persistency
If you're using any of the clouds available via Monk.you can use volume definition to spin a disk block device to make your Gitlab instance independent on the node it's running on.  
To do simply uncomment the `volume` block in `manifest.yaml`

## Create your own template

You can create your own template and inherit the defaults and add more changes that are needed by you. For example increase amount of workers.  
This is an example manifest that you could use:  

``` yaml
namespace: /gitlab

runner-1:
  defines: runnable
  inherits: /gitlab/runner

runner-2:
  defines: runnable
  inherits: /gitlab/runner

runner-3:
  defines: runnable
  inherits: /gitlab/runner

runner-stack:
  defines: process-group
  runnable-list:
    - /gitlab/runner-1
    - /gitlab/runner-2
    - /gitlab/runner-3
```