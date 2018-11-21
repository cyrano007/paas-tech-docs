# PostgreSQL

PostgreSQL is an object-relational database management system. It is open source and designed to be extensible.

### Set up a PostgreSQL service

To set up a PostgreSQL service:

1. Run the following code in the command line to see what plans are available for PostgreSQL:

    ```
    cf marketplace -s postgres
    ```

    Here is an example of the output you will see (the exact plans will vary):

    ```
    service plan             description                                                                                                                                                       free or paid
    small-9.5                20GB Storage, Dedicated Instance, Max 500 Concurrent Connections. Postgres Version X.X. DB Instance Class: db.m4.large.                                           paid
    small-ha-9.5             20GB Storage, Dedicated Instance, Highly Available, Max 500 Concurrent Connections. Postgres Version X.X. DB Instance Class: db.m4.large.                         paid
    ...
    tiny-9.5                 5GB Storage, NOT BACKED UP, Dedicated Instance, Max 50 Concurrent Connections. Postgres Version X.X. DB Instance Class: db.t2.micro.                              free
    ```

    The syntax in this output is explained in the following table:

    |Syntax|Meaning|
    |:---|:---|
    |`ha`|High availability|
    |`X.X`|Version number|
    |`small / medium / large / xlarge`|Size of instance|

    More information can be found in the [PostgreSQL plans](/deploying_services/postgresql/#postgresql-plans) section.

1. Run the following code in the command line:

    ```
    cf create-service postgres PLAN SERVICE_NAME
    ```

    where `PLAN` is the plan you want, and `SERVICE_NAME` is a unique descriptive name for this service instance. For example:

    ```
    cf create-service postgres small-9.5 my-pg-service
    ```

    You should use a high-availability (`ha`) encrypted plan for production apps.

1. It will take between 5 and 10 minutes to set up the service instance. To check its progress, run:

    ```
    cf service SERVICE_NAME
    ```

    for example:

    ```
    cf service my-pg-service
    ```

    The service is set up when the `cf service SERVICE_NAME` command returns a `create succeeded` status. Here is an example of the output you will see:

    ```
    Service instance: my-pg-service
    Service: postgres
    Bound apps:
    Tags:
    Plan: small-9.5
    Description: AWS RDS PostgreSQL service
    Documentation url: https://aws.amazon.com/documentation/rds/
    Dashboard:

    Last Operation
    Status: create succeeded
    Message: DB Instance 'rdsbroker-9f053413-97a5-461f-aa41-fe6e29db323e' status is 'available'
    Started: 2016-08-23T15:34:41Z
    Updated: 2016-08-23T15:42:02Z
    ```

### Bind a PostgreSQL service to your app

You must bind your app to the PostgreSQL service so you can access the database from the app.

1. Use the [app's manifest](/deploying_apps.html#deploying-public-apps) to bind the app to the service instance. It will bind automatically when you next deploy your app. An example manifest:

    ```
    --
    applications:
    - name: my-app
      services:
      - my-pg-service
    ```

1. Deploy your app in line with your normal deployment process.

This binds your app to a service instance called `my-pg-service`.

Refer to the Cloud Foundry documentation on [deploying with app manifests](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html#services-block) [external link] for more information.

#### Use the cf bind-service command

Alternatively, you can manually bind your service instance to your app.

1. Run the following:

    ```
    cf bind-service APP_NAME SERVICE_NAME
    ```

    where `APP_NAME` is the exact name of a deployed instance of your application and `SERVICE_NAME` is the name of the service instance you created. For example:

    ```
    cf bind-service my-app my-pg-service
    ```

1. Deploy your app in line with your normal deployment process.

### Connect to a PostgreSQL service from your app

Your app must make a [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) [external link] connection to the service. Some libraries use TLS by default, but others will need to be manually configured.

GOV.UK PaaS will automatically parse the ``VCAP_SERVICES`` [environment variable](/deploying_apps.html#system-provided-environment-variables) to get details of the service and then set the `DATABASE_URL` variable to the first database found.

If your app writes database connection errors to `STDOUT` or `STDERR`, you can view recent errors with `cf logs APP_NAME --recent`. See the section on [Logs](/monitoring_apps.html#logs) for details.

### Connect to a PostgreSQL service from your local machine

We have created the [Conduit](/guidance.html#conduit) plugin to simplify the process of connecting your local machine to a PostgreSQL service. To install this plugin, run the following code from the command line:

```
cf install-plugin conduit
```

Once the plugin has finished installing, run the following code in the command line to access an SQL shell for your backing service:

```
cf conduit SERVICE_NAME -- psql
```

where `SERVICE_NAME` is a unique descriptive name for this service instance.

Run `cf conduit --help` for more options, and refer to the [Conduit readme file](https://github.com/alphagov/paas-cf-conduit/blob/master/README.md) [external link] for more information on how to use the plugin.

### Import and export bulk data to and from a PostgreSQL database

#### Prerequisites

You must:

- install and configure the [PostgreSQL command line (CLI) tools](https://postgresapp.com/documentation/cli-tools.html) [external link] on your local machine (configuration options vary depending on operating system and version)
- [log into Cloud Foundry](/get_started.html#set-up-command-line)
- [create the new PaaS-hosted PostgreSQL database](/deploying_services/postgresql/#set-up-a-postgresql-service)
- [target the space](/deploying_apps.html#set-a-target) where your new database is located

#### Non-PaaS to PaaS

To move data from a non-PaaS PostgreSQL database to a PaaS PostgreSQL database:

1. Run the following command in the CLI to export data from the non-PaaS database to an SQL data file:

    ```
    pg_dump --host HOST_NAME --file DATA_FILE_NAME DATABASE_NAME
    ```

    where:
      - `HOST_NAME` is the name of your host
      - `DATA_FILE_NAME` is the SQL data file
      - `DATABASE_NAME` is the name of the non-PaaS source database

2. Use the [Conduit plugin](/deploying_services/postgresql/#connect-to-a-postgresql-service-from-your-local-machine) to import the data file into the PaaS database by running:

    ```
    cf conduit SERVICE_NAME -- psql < DATA_FILE_NAME
    ```

    where `SERVICE_NAME` is a unique descriptive name for this service instance, and `DATA_FILE_NAME` is the SQL file created in the previous step.

> You can only use [certain PostgreSQL extensions](/deploying_services/postgresql/#postgresql-extensions-whitelist).

#### PaaS to PaaS

To move data between two PaaS-hosted PostgreSQL databases:

1. Use the [Conduit plugin](/deploying_services/postgresql/#connect-to-a-postgresql-service-from-your-local-machine) to connect to the source database and export the data into an SQL file by running:

    ```
    cf conduit SERVICE_NAME -- pg_dump --file DATA_FILE_NAME
    ```

    where `SERVICE_NAME` is a unique descriptive name for this service instance and `DATA_FILE_NAME` is the SQL data file created by the `pg_dump` command.


2. Run the following command to import the data file into the target database:

     ```
     cf conduit DESTINATION_SERVICE_NAME -- psql < DATA_FILE_NAME
     ```

    where `DESTINATION_SERVICE_NAME` is the name of the target database.

Contact the PaaS team at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) if you have any questions.

### Upgrade PostgreSQL service plan

#### Same encryption type

If your new plan uses the same encryption type as your current plan, you can upgrade your plan using the `cf update-service` command. Run the following in the command line:

```
cf update-service SERVICE_NAME -p NEW_PLAN_NAME
```

where `SERVICE_NAME` is a unique descriptive name for this service instance, and `NEW_PLAN_NAME` is the name of your new plan. For example:

```
cf update-service my-pg-service -p small-ha-9.5
```

The plan upgrade will start immediately and finish within an hour. You can check the status of the upgrade by running `cf services`.

You can also [queue a plan upgrade](/deploying_services/postgresql/#queue-a-plan-migration-postgresql) to happen during a maintenance window to minimise service interruption.

#### Different encryption type

You cannot upgrade your service plan using the `cf update-service` command if the new plan uses a different encryption type to your current plan.

To upgrade, you must set up a new service and migrate your app data.

1. [Set up a new PostgreSQL service](/deploying_services/postgresql/#set-up-a-postgresql-service) with a new plan that has a different encryption type to your current plan.
1. Move your app data from the current service to the new service by following the [import and export bulk data documentation](/deploying_services/postgresql/#paas-to-paas).

#### Downgrade PostgreSQL service plan

You cannot currently downgrade your service plan.

### Unbind a PostgreSQL service from your app

You must unbind the PostgreSQL service before you can delete it. To unbind the PostgreSQL service, run the following code in the command line:

```
cf unbind-service APPLICATION SERVICE_NAME
```

where `APPLICATION` is the name of a deployed instance of your application (exactly as specified in your app's `manifest.yml` file or push command) and `SERVICE_NAME` is a unique descriptive name for this service instance, for example:

```
cf unbind-service my-app my-pg-service
```

If you unbind your services from your app but do not delete them, the services will persist even after your app is deleted, and you can re-bind or re-connect to them in future.

### Delete a PostgreSQL service

Once the PostgreSQL service has been unbound from your app, you can delete it. Run the following code in the command line:

```
cf delete-service SERVICE_NAME
```

where `SERVICE_NAME` is a unique descriptive name for this service instance.

Type `yes` when asked for confirmation.

### Data classification

You can store data classified up to ‘official’ on the GOV.UK PaaS. Refer to the [data security classification documentation](/deploying_services/#data-security-classification) for more information.

### PostgreSQL plans

Each service in the marketplace has multiple plans that vary by availability and storage capacity.

#### Paid plans - PostgreSQL

Some service plans are paid and we can potentially bill you based on your service usage.

New organisations cannot access paid plans by default. Enabling this access is controlled by an organisation's [quota](/#quotas) settings.

If paid plans are not enabled, when you try to use a paid service you will receive an error stating “service instance cannot be created because paid service plans are not allowed”. One of your [Org Managers](/orgs_spaces_users.html#org-manager) must contact us at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) to request that we enable paid services.

There is a free plan available with limited storage which should only be used for development or testing, but not production.

#### Encrypted plans - PostgreSQL

All plans have encryption at rest unless stated otherwise. This means that both the data on the disk and in snapshots is encrypted.

#### High availability plans - PostgreSQL

We recommend you use a high availability plan (`HA`) for your PostgreSQL apps. These plans use Amazon RDS Multi-AZ instances, which are designed to be 99.95% available. See [Amazon's SLA](https://aws.amazon.com/rds/sla/) [external link] for details.

When you use a high availability plan, Amazon RDS provides a hot standby service for failover in the event that the original service fails.

Refer to the [Amazon RDS documentation on the failover process](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html#Concepts.MultiAZ.Failover) [external link] for more information.

You should test how your app deals with a failover to make sure you are benefiting from the high availability plan. Contact us at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) to arrange for us to trigger a failover for you.

#### Read replicas - PostgreSQL

Amazon RDS has the capability to provide a read-only copy of your database known as a read replica. This can be useful for performance, availability or security reasons.

Refer to the [Amazon RDS documentation on read replicas](https://aws.amazon.com/rds/details/read-replicas/) [external link] for more information.

GOV.UK PaaS doesn't currently support read replicas, but if you think you would find them useful, please contact us at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk), providing details of your use case.

### PostgreSQL maintenance & backups

#### PostgreSQL maintenance times

Each PostgreSQL service you create will have a randomly-assigned weekly 30 minute maintenance window, during which there may be brief downtime. Select a high availability (`HA`) plan to minimise this downtime. Minor version upgrades (for example from 9.4.1 to 9.4.2) are applied during this maintenance window.

Contact us at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) to find out the default time of your maintenance window. Window start times will vary from 22:00 to 06:00 UTC.

You can set your own maintenance window by running `cf update-service` in the command line and setting the `preferred_maintenance_window` custom parameter:

```
cf update-service SERVICE_NAME -c '{"preferred_maintenance_window": "START_DAY:START_TIME-END_DAY:END_TIME"}'
```

where `SERVICE_NAME` is a unique, descriptive name for this service instance, for example:

```
cf update-service my-pg-service -c '{"preferred_maintenance_window": "Tue:04:00-Tue:04:30"}'
```

For more information on maintenance times, refer to the [Amazon RDS Maintenance documentation](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Maintenance.html) [external link].

#### Queue a plan migration - PostgreSQL

Migrating to a new plan may cause interruption to your service instance. To minimise interruption, you can queue the change to begin during a maintenance window by running the following code in the command line:

```
cf update-service SERVICE_NAME -p PLAN -c '{"apply_at_maintenance_window": true, "preferred_maintenance_window": "START_DAY:START_TIME-END_DAY:END_TIME"}'
```

where `SERVICE_NAME` is a unique, descriptive name for this service instance and `PLAN` is the plan that you are upgrading to, for example:

```
cf update-service my-pg-service -p small-ha-9.5 -c '{"apply_at_maintenance_window": true, "preferred_maintenance_window": "wed:03:32-wed:04:02"}'
```

Passing the `preferred_maintenance_window` parameter will alter the default maintenance window for any future maintenance events required for the database instance.

You can only migrate your service if the new plan has the [same encryption type](/deploying_services/postgresql/#same-encryption-type) as your current plan.

#### PostgreSQL service backup

The data stored within any PostgreSQL service you create is backed up using the standard Amazon RDS backup system if you are using a paid plan. Your data is not backed up if you are using the free plan.

Backups are taken nightly at some time between 22:00 and 06:00 UTC. Data is retained for 7 days.

There are two ways you can restore data to an earlier state:

1. You can restore to the latest snapshot yourself. See [Restoring a PostgreSQL service snapshot](/deploying_services/postgresql/#restoring-a-postgresql-service-snapshot) for details.

1. We can manually restore to any point from 5 minutes to 7 days ago, with a resolution of one second. Data can be restored to a new PostgreSQL service instance running in parallel, or it can replace the existing service instance.

    To arrange a manual restore, contact us at [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk). We will need approval from your organization manager if restoring will involve overwriting data.

Note that data restore will not be available in the event of an RDS outage that affects the entire Amazon availability zone.

For more details about how the RDS backup system works, see [Amazon's DB Instance Backups documentation](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.BackingUpAndRestoringAmazonRDSInstances.html) [external link].

#### Restoring a PostgreSQL service snapshot

You can create a copy of any existing PostgreSQL service instance using the latest snapshot of the RDS instance. These snapshots are taken during [the PostgreSQL nightly backups](/deploying_services/postgresql/#postgresql-service-backup).

This can be useful if you want to clone a production database to be used for testing or batch processing.

To restore from a snapshot:

 1. Get the global unique identifier (GUID) of the existing instance by running the following code in the command line:

    ```
    cf service SERVICE_NAME --guid
    ```

    where `SERVICE_NAME` is the name of the PostgreSQL service instance you want to copy. For example:

    ```
    cf service my-pg-service --guid
    ```

    This returns a `GUID` in the format `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`, for example `32938730-e603-44d6-810e-b4f12d7d109e`.

 2. Trigger the creation of a new service based on the snapshot by running:

    ```
    cf create-service postgres PLAN NEW_SERVICE_NAME -c '{"restore_from_latest_snapshot_of": "GUID"}'
    ```

    where `PLAN` is the plan used in the original instance (you can find this out by running `cf service SERVICE_NAME`), and `NEW_SERVICE_NAME` is a unique, descriptive name for this new instance. For example:

    ```
    cf create-service postgres small-9.5 my-pg-service-copy  -c '{"restore_from_latest_snapshot_of": "32938730-e603-44d6-810e-b4f12d7d109e"}'
    ```

 3. It takes between 5 to 10 minutes for the new service instance to be set up. To find out its status, run:

    ```
    cf service NEW_SERVICE_NAME
    ```

    for example:

    ```
    cf service my-pg-service-copy
    ```

 4. The new instance is set up when the `cf service NEW_SERVICE_NAME` command returns a `create succeeded` status. See [Set up a PostgreSQL service](/deploying_services/postgresql/#set-up-a-postgresql-service) for more details.

 This feature has the following limitations:

  * You can only restore the most recent snapshot from the latest nightly backup
  * You cannot restore from a service instance that has been deleted
  * You must use the same service plan for the copy as for the original service instance
  * You must create the new service instance in the same organisation and space as the original. This is to prevent unauthorised access to data between spaces. If you need to copy data to a different organisation and/or space, you can [connect to your PostgreSQL instance from a local machine using Conduit](/deploying_services/postgresql/#connect-to-a-postgresql-service-from-your-local-machine).

### PostgreSQL extensions whitelist

We currently enable the following extensions for PostgreSQL:

- postgis
- uuid-ossp