# dokku-neo4j

A neo4j plugin for [Dokku](https://dokku.com/).

Based off of the [official postgres plugin for dokku](https://github.com/dokku/dokku-postgres). Currently defaults to installing [neo4j 4.4.8 community edition](https://hub.docker.com/_/neo4j/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/maayanlab/dokku-neo4j.git neo4j
```

## Commands

```
neo4j:app-links <app>                           # list all neo4j service links for a given app
neo4j:backup <service> <bucket-name> [--use-iam] # create a backup of the neo4j service to an existing s3 bucket
neo4j:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # set up authentication for backups on the neo4j service
neo4j:backup-deauth <service>                   # remove backup authentication for the neo4j service
neo4j:backup-schedule <service> <schedule> <bucket-name> [--use-iam] # schedule a backup of the neo4j service
neo4j:backup-schedule-cat <service>             # cat the contents of the configured backup cronfile for the service
neo4j:backup-set-encryption <service> <passphrase> # set encryption for all future backups of neo4j service
neo4j:backup-unschedule <service>               # unschedule the backup of the neo4j service
neo4j:backup-unset-encryption <service>         # unset encryption for future backups of the neo4j service
neo4j:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
neo4j:connect <service>                         # connect to the service via the neo4j connection tool
neo4j:create <service> [--create-flags...]      # create a neo4j service
neo4j:destroy <service> [-f|--force]            # delete the neo4j service/data/container if there are no links left
neo4j:enter <service>                           # enter or run a command in a running neo4j service container
neo4j:exists <service>                          # check if the neo4j service exists
neo4j:export <service>                          # export a dump of the neo4j service database
neo4j:expose <service> <ports...>               # expose a neo4j service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
neo4j:import <service>                          # import a dump into the neo4j service database
neo4j:info <service> [--single-info-flag]       # print the service information
neo4j:link <service> <app> [--link-flags...]    # link the neo4j service to the app
neo4j:linked <service> <app>                    # check if the neo4j service is linked to an app
neo4j:links <service>                           # list all apps linked to the neo4j service
neo4j:list                                      # list all neo4j services
neo4j:logs <service> [-t|--tail] <tail-num-optional> # print the most recent log(s) for this service
neo4j:promote <service> <app>                   # promote service <service> as DATABASE_URL in <app>
neo4j:restart <service>                         # graceful shutdown and restart of the neo4j service container
neo4j:start <service>                           # start a previously stopped neo4j service
neo4j:stop <service>                            # stop a running neo4j service
neo4j:unexpose <service>                        # unexpose a previously exposed neo4j service
neo4j:unlink <service> <app>                    # unlink the neo4j service from the app
neo4j:upgrade <service> [--upgrade-flags...]    # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to neo4j:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `neo4j:help` command for any undocumented commands.

### Basic Usage

### create a neo4j service

```shell
# usage
dokku neo4j:create <service> [--create-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password
- `-s|--shm-size SHM_SIZE`: override shared memory size for neo4j docker container

Create a neo4j service named lollipop:

```shell
dokku neo4j:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the neo4j image.

```shell
export NEO4J_IMAGE="neo4j"
export NEO4J_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku neo4j:create lollipop
```

You can also specify custom environment variables to start the neo4j service in semi-colon separated form.

```shell
export NEO4J_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku neo4j:create lollipop
```

### print the service information

```shell
# usage
dokku neo4j:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku neo4j:info lollipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku neo4j:info lollipop --config-dir
dokku neo4j:info lollipop --data-dir
dokku neo4j:info lollipop --dsn
dokku neo4j:info lollipop --exposed-ports
dokku neo4j:info lollipop --id
dokku neo4j:info lollipop --internal-ip
dokku neo4j:info lollipop --links
dokku neo4j:info lollipop --service-root
dokku neo4j:info lollipop --status
dokku neo4j:info lollipop --version
```

### list all neo4j services

```shell
# usage
dokku neo4j:list 
```

List all services:

```shell
dokku neo4j:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku neo4j:logs <service> [-t|--tail] <tail-num-optional>
```

flags:

- `-t|--tail [<tail-num>]`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku neo4j:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku neo4j:logs lollipop --tail
```

The default tail setting is to show all logs, but an initial count can also be specified:

```shell
dokku neo4j:logs lollipop --tail 5
```

### link the neo4j service to the app

```shell
# usage
dokku neo4j:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link

A neo4j service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku neo4j:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_NEO4J_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_NEO4J_LOLLIPOP_PORT=tcp://172.17.0.1:7687
DOKKU_NEO4J_LOLLIPOP_PORT_7687_TCP=tcp://172.17.0.1:7687
DOKKU_NEO4J_LOLLIPOP_PORT_7687_TCP_PROTO=tcp
DOKKU_NEO4J_LOLLIPOP_PORT_7687_TCP_PORT=7687
DOKKU_NEO4J_LOLLIPOP_PORT_7687_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
DATABASE_URL=neo4j://lollipop:SOME_PASSWORD@dokku-neo4j-lollipop:7687/lollipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku neo4j:link other_service playground
```

It is possible to change the protocol for `DATABASE_URL` by setting the environment variable `NEO4J_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground NEO4J_DATABASE_SCHEME=neo4j2
dokku neo4j:link lollipop playground
```

This will cause `DATABASE_URL` to be set as:

```
neo4j2://lollipop:SOME_PASSWORD@dokku-neo4j-lollipop:7687/lollipop
```

### unlink the neo4j service from the app

```shell
# usage
dokku neo4j:unlink <service> <app>
```

You can unlink a neo4j service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku neo4j:unlink lollipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the neo4j connection tool

```shell
# usage
dokku neo4j:connect <service>
```

Connect to the service via the neo4j connection tool:

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku neo4j:connect lollipop
```

### enter or run a command in a running neo4j service container

```shell
# usage
dokku neo4j:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku neo4j:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku neo4j:enter lollipop touch /tmp/test
```

### expose a neo4j service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku neo4j:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku neo4j:expose lollipop 7687
```

Expose the service on the service's normal ports, with the first on a specified ip adddress (127.0.0.1):

```shell
dokku neo4j:expose lollipop 127.0.0.1:7687
```

### unexpose a previously exposed neo4j service

```shell
# usage
dokku neo4j:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku neo4j:unexpose lollipop
```

### promote service <service> as DATABASE_URL in <app>

```shell
# usage
dokku neo4j:promote <service> <app>
```

If you have a neo4j service linked to an app and try to link another neo4j service another link environment variable will be generated automatically:

```
DOKKU_DATABASE_BLUE_URL=neo4j://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:7687/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku neo4j:promote other_service playground
```

This will replace `DATABASE_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
DATABASE_URL=neo4j://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:7687/other_service
DOKKU_DATABASE_BLUE_URL=neo4j://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:7687/other_service
DOKKU_DATABASE_SILVER_URL=neo4j://lollipop:SOME_PASSWORD@dokku-neo4j-lollipop:7687/lollipop
```

### start a previously stopped neo4j service

```shell
# usage
dokku neo4j:start <service>
```

Start the service:

```shell
dokku neo4j:start lollipop
```

### stop a running neo4j service

```shell
# usage
dokku neo4j:stop <service>
```

Stop the service and the running container:

```shell
dokku neo4j:stop lollipop
```

### graceful shutdown and restart of the neo4j service container

```shell
# usage
dokku neo4j:restart <service>
```

Restart the service:

```shell
dokku neo4j:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku neo4j:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-R|--restart-apps "true"`: whether to force an app restart
- `-s|--shm-size SHM_SIZE`: override shared memory size for neo4j docker container

You can upgrade an existing service to a new image or image-version:

```shell
dokku neo4j:upgrade lollipop
```

Neo4j does not handle upgrading data for major versions automatically (eg. 11 => 12). Upgrades should be done manually. Users are encouraged to upgrade to the latest minor release for their neo4j version before performing a major upgrade.

While there are many ways to upgrade a neo4j database, for safety purposes, it is recommended that an upgrade is performed by exporting the data from an existing database and importing it into a new database. This also allows testing to ensure that applications interact with the database correctly after the upgrade, and can be used in a staging environment.

The following is an example of how to upgrade a neo4j database named `lollipop-11` from 11.13 to 12.8.

```shell
# stop any linked apps
dokku ps:stop linked-app

# export the database contents
dokku neo4j:export lollipop-11 > /tmp/lollipop-11.export

# create a new database at the desired version
dokku neo4j:create lollipop-12 --image-version 12.8

# import the export file
dokku neo4j:import lollipop-12 < /tmp/lollipop-11.export

# run any sql tests against the new database to verify the import went smoothly

# unlink the old database from your apps
dokku neo4j:unlink lollipop-11 linked-app

# link the new database to your apps
dokku neo4j:link lollipop-12 linked-app

# start the linked apps again
dokku ps:start linked-app
```

### Service Automation

Service scripting can be executed using the following commands:

### list all neo4j service links for a given app

```shell
# usage
dokku neo4j:app-links <app>
```

List all neo4j services that are linked to the `playground` app.

```shell
dokku neo4j:app-links playground
```

### create container <new-name> then copy data from <name> into <new-name>

```shell
# usage
dokku neo4j:clone <service> <new-service> [--clone-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password
- `-s|--shm-size SHM_SIZE`: override shared memory size for neo4j docker container

You can clone an existing service to a new one:

```shell
dokku neo4j:clone lollipop lollipop-2
```

### check if the neo4j service exists

```shell
# usage
dokku neo4j:exists <service>
```

Here we check if the lollipop neo4j service exists.

```shell
dokku neo4j:exists lollipop
```

### check if the neo4j service is linked to an app

```shell
# usage
dokku neo4j:linked <service> <app>
```

Here we check if the lollipop neo4j service is linked to the `playground` app.

```shell
dokku neo4j:linked lollipop playground
```

### list all apps linked to the neo4j service

```shell
# usage
dokku neo4j:links <service>
```

List all apps linked to the `lollipop` neo4j service.

```shell
dokku neo4j:links lollipop
```

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the neo4j service database

```shell
# usage
dokku neo4j:import <service>
```

Import a datastore dump:

```shell
dokku neo4j:import lollipop < data.dump
```

### export a dump of the neo4j service database

```shell
# usage
dokku neo4j:export <service>
```

By default, datastore output is exported to stdout:

```shell
dokku neo4j:export lollipop
```

You can redirect this output to a file:

```shell
dokku neo4j:export lollipop > data.dump
```

Note that the export will result in a file containing the binary neo4j export data. It can be converted to plain text using `neo4j-admin load` as follows

```shell
neo4j-admin load data.dump -f plain.sql
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### set up authentication for backups on the neo4j service

```shell
# usage
dokku neo4j:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

Setup s3 backup authentication:

```shell
dokku neo4j:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
```

Setup s3 backup authentication with different region:

```shell
dokku neo4j:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION
```

Setup s3 backup authentication with different signature version and endpoint:

```shell
dokku neo4j:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL
```

More specific example for minio auth:

```shell
dokku neo4j:backup-auth lollipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```

### remove backup authentication for the neo4j service

```shell
# usage
dokku neo4j:backup-deauth <service>
```

Remove s3 authentication:

```shell
dokku neo4j:backup-deauth lollipop
```

### create a backup of the neo4j service to an existing s3 bucket

```shell
# usage
dokku neo4j:backup <service> <bucket-name> [--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Backup the `lollipop` service to the `my-s3-bucket` bucket on `AWS`:`

```shell
dokku neo4j:backup lollipop my-s3-bucket --use-iam
```

Restore a backup file (assuming it was extracted via `tar -xf backup.tgz`):

```shell
dokku neo4j:import lollipop < backup-folder/export
```

### set encryption for all future backups of neo4j service

```shell
# usage
dokku neo4j:backup-set-encryption <service> <passphrase>
```

Set the GPG-compatible passphrase for encrypting backups for backups:

```shell
dokku neo4j:backup-set-encryption lollipop
```

### unset encryption for future backups of the neo4j service

```shell
# usage
dokku neo4j:backup-unset-encryption <service>
```

Unset the `GPG` encryption passphrase for backups:

```shell
dokku neo4j:backup-unset-encryption lollipop
```

### schedule a backup of the neo4j service

```shell
# usage
dokku neo4j:backup-schedule <service> <schedule> <bucket-name> [--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am

```shell
dokku neo4j:backup-schedule lollipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku neo4j:backup-schedule lollipop "0 3 * * *" my-s3-bucket --use-iam
```

### cat the contents of the configured backup cronfile for the service

```shell
# usage
dokku neo4j:backup-schedule-cat <service>
```

Cat the contents of the configured backup cronfile for the service:

```shell
dokku neo4j:backup-schedule-cat lollipop
```

### unschedule the backup of the neo4j service

```shell
# usage
dokku neo4j:backup-unschedule <service>
```

Remove the scheduled backup from cron:

```shell
dokku neo4j:backup-unschedule lollipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `NEO4J_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.