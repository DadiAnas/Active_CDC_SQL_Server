# Activating Change Data Capture (CDC)

This guide provides steps to activate Change Data Capture (CDC) on SQL Server, Oracle, and PostgreSQL.

## Prerequisites
Activating Change Data Capture (CDC) or similar features in databases often requires certain prerequisites to be met. Here are the general prerequisites for SQL Server, Oracle, and PostgreSQL:

### SQL Server:

1. **Edition**: CDC is available only in the Enterprise, Developer, and Evaluation editions of SQL Server.

2. **Permissions**: The account used to enable CDC must have the `db_owner` role on the database.

3. **SQL Server Agent**: The SQL Server Agent service must be running, as it's used to support the capture mechanism of CDC.

4. **Backup**: It's advisable to have a recent backup of your database before enabling CDC.

### Oracle:

1. **Supplemental Logging**: Before enabling CDC, ensure that supplemental logging is enabled at the database level.

2. **Privileges**: The user account should have the necessary privileges to execute the `DBMS_CDC_PUBLISH` and `DBMS_CDC_SUBSCRIBE` packages.

3. **Table Types**: Some types of tables, like temporary tables, might not support CDC.

4. **Archiving**: The database should be in ARCHIVELOG mode.

5. **Space**: Ensure there's sufficient space in the redo log and undo tablespaces, as CDC can increase the amount of redo log generated.

### PostgreSQL:

1. **Version**: Logical replication (used for CDC-like functionality) is available starting from PostgreSQL version 10.

2. **Configuration**: The `postgresql.conf` file must be configured to support logical replication (`wal_level`, `max_replication_slots`, `max_wal_senders`).

3. **Replication Role**: The user account should have the `REPLICATION` role.

4. **Connection**: Ensure that the `pg_hba.conf` file is configured to allow replication connections.

5. **Disk Space**: Ensure there's sufficient disk space, as replication slots can retain WAL segments and increase disk usage.

6. **Backup**: As with any significant change, it's advisable to have a recent backup of your database.

For all databases, it's also essential to:

- **Test in a Non-Production Environment**: Before enabling CDC in a production environment, test the setup in a development or staging environment to understand the implications and ensure smooth activation.

- **Monitor Performance**: Activating CDC can have performance implications. Monitor the system's performance after enabling CDC and adjust configurations as necessary.

- **Review Official Documentation**: Always refer to the official documentation of the respective database system for detailed prerequisites and considerations specific to your database version and setup.

# Activation 
## SQL Server

### 1. Activate CDC at the Database Level:

```
USE [YourDatabaseName]
GO

EXEC sys.sp_cdc_enable_db
GO
```

### 2. Activate CDC for a Specific Table:

```
USE [YourDatabaseName]
GO

EXEC sys.sp_cdc_enable_table
@source_schema = N'[TableSchema]',
@source_name   = N'[TableName]',
@role_name     = NULL,
@supports_net_changes = 1
GO
```

## Oracle

For Oracle, you'll need to use Oracle's DBMS_CDC_PUBLISH package.

### 1. Grant Necessary Privileges:

```
GRANT EXECUTE ON DBMS_CDC_PUBLISH TO [YourUsername];
GRANT EXECUTE ON DBMS_CDC_SUBSCRIBE TO [YourUsername];
```

### 2. Begin the CDC Configuration:

```
BEGIN
  DBMS_CDC_PUBLISH.CREATE_CHANGE_SET(change_set_name => 'YOUR_CHANGE_SET_NAME', description => 'Change set description', change_source_name => 'YOUR_SOURCE_NAME', stop_on_ddl => 'N', begin_date => NULL, end_date => NULL);
END;
/
```

## PostgreSQL

For PostgreSQL, logical replication is used for CDC.

### 1. Modify the Configuration File:

In `postgresql.conf`, set:

```
wal_level = logical
max_replication_slots = 4
max_wal_senders = 4
```

### 2. Create an Output Plugin:

```
SELECT * FROM pg_create_logical_replication_slot('your_slot_name', 'pgoutput');
```

### 3. Set Up a Subscription:

For this, you'll need to set up another PostgreSQL instance as a subscriber.

---

Remember to replace placeholders like `[YourDatabaseName]`, `[TableSchema]`, and `[TableName]` with appropriate values for your setup. Always backup your data and configurations before making changes, and test in a development environment first.

## MySQL

Change Data Capture in MySQL is achieved through binary logging. The steps to enable binary logging differ slightly between versions.

### MySQL 5.5:

1. **Enable Binary Logging**: Edit your `my.cnf` or `my.ini` file (depending on your OS) and add the following lines:

```
log-bin=mysql-bin
server-id=1
```

2. **Restart MySQL Server**: After making the changes, restart the MySQL server.

3. **Verify**: Log in to MySQL and execute the following command to ensure binary logging is enabled:

```
SHOW BINARY LOGS;
```

### MySQL 5.6+:

1. **Enable Binary Logging**: Edit your `my.cnf` or `my.ini` file and add the following lines:

```
log-bin=mysql-bin
server-id=1
binlog_format=ROW
```

The `binlog_format=ROW` ensures that the binary log records changes in a row-based format, which is more suitable for CDC.

2. **Restart MySQL Server**: After making the changes, restart the MySQL server.

3. **Verify**: Log in to MySQL and execute the following command to ensure binary logging is enabled:

```
SHOW BINARY LOGS;
```

### Notes:

- **Disk Space**: Binary logs can consume a significant amount of disk space over time. Regularly purge old logs or configure `expire_logs_days` to auto-purge logs after a certain number of days.

- **Performance**: There might be a slight performance overhead due to binary logging, especially in high-transaction environments. Monitor and adjust as necessary.

- **Backup**: Always backup your data and configurations before making changes.


## Advanced Considerations & Troubleshooting

### SQL Server:

- **Disk Space**: CDC uses the transaction log to capture changes. Ensure that there's sufficient disk space, and monitor log file growth.
  
- **Permissions**: Ensure that the SQL Server Agent is running, as it's used to support the capture mechanism of CDC.

- **Troubleshooting**: If CDC isn't capturing changes, check the SQL Server Agent jobs related to CDC. They should be running without errors.

### Oracle:

- **Redo Log Space**: CDC in Oracle can increase the amount of redo log generated. Monitor the size and frequency of redo log switches.

- **Asynchronous Autolog**: For high-transaction systems, consider using the asynchronous autolog method to capture change data without impacting source system performance.

- **Troubleshooting**: Use the `DBA_APPLY_ERROR` view to diagnose issues related to CDC in Oracle. It provides details on errors encountered during the apply process.

### PostgreSQL:

- **Slot Deletion**: If a replication slot is no longer needed, ensure it's deleted to prevent old WAL files from accumulating and consuming disk space.

- **Monitoring**: Use the `pg_replication_slots` view to monitor the status and lag of replication slots.

- **Troubleshooting**: If you encounter issues with logical replication, check the PostgreSQL logs. They provide detailed information on replication processes and potential errors.

## Maintenance & Cleanup

Regular maintenance is crucial for the smooth operation of CDC:

- **SQL Server**: Use `sys.sp_cdc_cleanup_change_table` to manually clean up old records from change tables.
  
- **Oracle**: Consider scheduling periodic purges of the change tables to manage space and performance.

- **PostgreSQL**: Ensure that the `max_replication_slots` and `max_wal_senders` parameters are set appropriately based on the number of subscribers and replication slots.

## Feedback & Community

CDC is a dynamic area with continuous improvements and community contributions. Stay updated with the latest developments:

- Join forums and communities related to your database system.
- Attend webinars and workshops on CDC.
- Provide feedback to the database vendors to help improve CDC features.

Remember, while CDC provides valuable data capture capabilities, it's essential to balance its benefits with the potential performance overhead and maintenance considerations.


# Conclusion

Change Data Capture (CDC) is a powerful feature that allows you to track and capture data changes. When implemented correctly, it can provide valuable insights and support various use cases, from data replication to auditing. Always ensure you understand the implications of enabling CDC, regularly monitor the system, and adjust configurations as needed to maintain optimal performance.

For further reading and detailed documentation, refer to the official documentation of each database system:

- [SQL Server CDC Documentation](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server)
- [Oracle CDC Documentation](https://docs.oracle.com/en/database/)
- [Oracle CDC DBMS_CDC_PUBLISH](https://docs.oracle.com/cd/B13789_01/appdev.101/b10802/d_logpub.htm) [Latest Documentation](https://support.oracle.com/knowledge/Oracle%20Database%20Products/1203935_1.html)
- [PostgreSQL Logical Replication Documentation](https://www.postgresql.org/docs/current/logical-replication.html)
- [MySQL 5.5 Binary Logging](https://dev.mysql.com/doc/refman/5.5/en/binary-log.html)
- [MySQL 5.6 Binary Logging](https://dev.mysql.com/doc/refman/5.6/en/binary-log.html)

