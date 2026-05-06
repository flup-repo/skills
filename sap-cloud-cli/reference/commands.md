# sapccm Command Syntax Reference

## config

```bash
sapccm config list
sapccm config show KEY
sapccm config set KEY VALUE
sapccm config delete KEY
```

Common keys: `auth-credentials`, `subscription-code`, `application-code`, `environment-code`,
`build-code`, `branch`, `name`, `database-update-mode`, `strategy`, `description`,
`service-code`, `property-service-code`, `no-wait`

## build

```bash
sapccm build list [--subscription-code=<subscriptionCode>]
sapccm build show [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>]
sapccm build logs [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>] [--file-path=<filePath>]
sapccm build create [--subscription-code=<subscriptionCode>] [--application-code=<applicationCode>] [--branch=<branch>] [--name=<name>] [--no-wait]
```

## deployment

```bash
sapccm deployment list [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>]
sapccm deployment show [--subscription-code=<subscriptionCode>] [--deployment-code=<deploymentCode>]
sapccm deployment create [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] [--build-code=<buildCode>] [--database-update-mode=<databaseUpdateMode>] [--strategy=<strategy>] [--no-wait]
sapccm deployment cancel-options [--subscription-code=<subscriptionCode>] --deployment-code=<deploymentCode>
sapccm deployment cancel [--subscription-code=<subscriptionCode>] --deployment-code=<deploymentCode> --rollback-database=<true|false>
```

## databackup

```bash
sapccm databackup list [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>]
sapccm databackup show [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --databackup-code=<databackupCode>
sapccm databackup create [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --description=<description> [--no-wait]
sapccm databackup delete [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --databackup-code=<databackupCode> [--no-wait]
```

## datarestore

```bash
sapccm datarestore list [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>]
sapccm datarestore show [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --datarestore-code=<datarestoreCode>
sapccm datarestore create [--subscription-code=<subscriptionCode>] --source-environment-code=<sourceEnvironmentCode> [--environment-code=<environmentCode>] --databackup-code=<databackupCode> [--no-wait]
```

## service-properties

```bash
sapccm service-properties get [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --service-code=<serviceCode> --property-service-code=<servicePropertyCode>
sapccm service-properties put [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>] --service-code=<serviceCode> --property-service-code=<servicePropertyCode> --property-service-value=<servicePropertyValue>
```
