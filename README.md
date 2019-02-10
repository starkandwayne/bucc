# BUCC ([BOSH](http://bosh.io/), [UAA](https://github.com/cloudfoundry/uaa), [Credhub](https://github.com/cloudfoundry-incubator/credhub) and [Concourse](https://concourse-ci.org/)) [![BUCC CI](https://ci2.starkandwayne.com/api/v1/teams/starkandwayne/pipelines/bucc/jobs/integration-test/badge)](https://ci2.starkandwayne.com/teams/starkandwayne/pipelines/bucc)

The bucc command line utility allows for easy bootstrapping of the BUCC stack (Bosh Uaa Credhub and Concourse). Which is the starting point for many deployments.

## Install the bucc-cli

### Prepare the Environment

#### Ubuntu 18.04/18.10

1.  Make sure development environment is setup with proper utils and libraries:

```
sudo apt install libssl-dev libreadline-dev libgdbm-dev gcc make g++ ruby-dev zlibc zlib1g-dev zlib1g
```

### Next Steps

1. Install [BOSH CLI v2.0.1+](https://bosh.io/docs/cli-v2.html)

2. Optionall install [`direnv`](https://direnv.net/)

3. Clone this repository

```
git clone https://github.com/starkandwayne/bucc.git
cd bucc
source .envrc # if not using direnv
```

4. Install the credhub cli into the bin directory

Linux
```
wget -qO- https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.2.1/credhub-linux-2.2.1.tgz | tar xvz -C ./bin/
```

Mac
```
wget -qO- https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.2.1/credhub-darwin-2.2.1.tgz | tar xvz -C ./bin/
```

### Boot your BUCC VM

Choose your cpi:
```
bucc up --help
  --cpi      Cloud provider: [softlayer, virtualbox, gcp, vsphere, aws, docker, azure, 1-extra-ports.yml, openstack]
  --lite     Created bosh will use the warden cpi with garden runc
  --recreate Recreate VM in deployment, also when there are no changes
  --debug    Show arguments passed to 'bosh create-env'
  --concourse-ca-certs
  --concourse-lb
  --concourse-syslog
  --ldap
  --oauth-providers
  --proxy

  Optional cpi specific flags:
    softlayer: --cpi-dynamic
    virtualbox: --remote
    gcp: --ephemeral-external-ip --service-account --target-pool
    vsphere: --dns --resource-pool
    aws: --auto-assign-public-ip --lb-target-groups --security-groups --spot-instance
    docker: --unix-sock
    azure: --load-balancer --managed-disks
    openstack: --custom-ca --disk-az --dns --floating-ip --ignore-server-availability-zone --keystone-v2 --ntp --root-disk-size --trusted-certs
```

From the repo root run:
```
$ bucc up --lite
```

To delete your VM run:
```
$ bucc down
```

## Using BUCC

### Using BOSH

```
$ source <(bucc env) # should not be necessary when using direnv

$ bosh alias-env bucc
  Using environment '192.168.50.6' as client 'admin'

  Name               bosh  
  UUID               94e87b44-a7eb-4b67-a568-52553f87cd6e  
  Version            268.6.0 (00000000)  
  Director Stemcell  ubuntu-xenial/170.9  
  CPI                warden_cpi  
  Features           compiled_package_cache: disabled  
                     config_server: enabled  
                     local_dns: enabled  
                     power_dns: disabled  
                     snapshots: disabled  
  User               admin

  Succeeded

$ bosh vms
  Using environment '192.168.50.6' as client 'admin'

  Succeeded
```

### Using UAA

1. Install the cli

```
gem install cf-uaac
```

2. Use UAA

```
$ bucc uaac

  Target: https://192.168.50.6:8443
  Context: uaa_admin, from client uaa_admin


  Successfully fetched token via client credentials grant.
  Target: https://192.168.50.6:8443
  Context: uaa_admin, from client uaa_admin

$ uaac client get admin
  scope: uaa.none
  client_id: admin
  resource_ids: none
  authorized_grant_types: client_credentials
  autoapprove:
  authorities: bosh.admin
  lastmodified: 1490280436993
```

### Using Credhub

```
$ source <(bucc env) # should not be necessary when using direnv

$ bucc credhub
Setting the target url: https://192.168.50.6:8844
Login Successful

$ credhub api
https://192.168.50.6:8844

$ credhub generate -t password --name test
  id: 63947a28-ee47-4d3c-9320-7972c70ec431
  name: /test
  type: password
  value: <redacted>
  version_created_at: "2019-02-10T13:35:06Z"
```

### Using Concourse

#### Via the GUI

To get the login details for your concourse GUI run:

```
bucc info
```


#### With Fly

```
$ bucc fly

  logging in to team 'main'

  target saved
  Example fly commands:
    fly -t bucc pipelines
    fly -t bucc builds

$ fly -t bucc pipelines
  name  paused  public
```

## Backup & Restore
BUCC works with [BBR](https://github.com/cloudfoundry-incubator/bosh-backup-and-restore).

To make a backup of you deployed BUCC vm, run:

```
bucc bbr backup
```

To recreate your environment from a backup run:

```
cd bucc
last_backup=$(find . -type d -regex ".+_.+Z" | sort -r | head -n1)
tar -xf ${last_backup}/bosh-0-bucc-creds.tar -C state
bucc up # clean BUCC with credentials (creds.yml) from backup
bucc bbr restore --artifact-path=${last_backup}
```
