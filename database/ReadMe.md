## Database setup

1. Boot up an Ubuntu 20.04 VM
1. Setup Mysql [using this DigitalOcean ref](https://www.digitalocean.com/community/tutorials/how-to-install-the-latest-mysql-on-ubuntu-16-04)
1. Call `uuidgen`  then `sudo nano /etc/mysql/my.cnf`
1. Fill it in following the below. Read the comments for `host-specific` or `shared` config. Make sure to open port `3306` and `33061`.

    ```bash
    [mysqld]

    # General replication settings
    gtid_mode = ON
    enforce_gtid_consistency = ON
    master_info_repository = TABLE
    relay_log_info_repository = TABLE
    binlog_checksum = NONE
    log_slave_updates = ON
    log_bin = binlog
    binlog_format = ROW
    transaction_write_set_extraction = XXHASH64
    loose-group_replication_bootstrap_group = OFF
    loose-group_replication_start_on_boot = OFF
    loose-group_replication_ssl_mode = REQUIRED
    loose-group_replication_recovery_use_ssl = 1

    # Shared replication group configuration

    loose-group_replication_group_name = "89de6b81-f610-4498-b9ca-0ea0b20f5242" # optional change name, but remember to keep the same across VM
    loose-group_replication_ip_whitelist = "10.184.0.2,10.184.0.3,10.184.0.4"
    loose-group_replication_group_seeds = "10.184.0.2:33061,10.184.0.3:33061,10.184.0.4:33061"

    # Single or Multi-primary mode? Uncomment these two lines
    # for multi-primary mode, where any host can accept writes
    #loose-group_replication_single_primary_mode = OFF
    #loose-group_replication_enforce_update_everywhere_checks = ON

    # Host specific replication configuration
    server_id = 1
    bind-address = "10.184.0.4"
    report_host = "10.184.0.4"
    loose-group_replication_local_address = "10.184.0.4:33061"
    ```
1. Run the below to restart mysql
    ```bash
    sudo rm /var/lib/mysql/auto.cnf
    sudo systemctl restart mysql
    ```
1. Run the below to setup the plugin. Make sure to change the user and pass
    ```bash
    mysql -u root -p
    ```

    ```sql
    /*Set to don't log*/
    SET SQL_LOG_BIN=0;
    UNINSTALL COMPONENT 'file://component_validate_password';

    /*Change details*/
    CREATE USER 'repl'@'%' IDENTIFIED BY 'repl'; 
    GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
    FLUSH PRIVILEGES;

    /*Set to log*/
    SET SQL_LOG_BIN=1;

    /*Change details*/
    CHANGE MASTER TO MASTER_USER='repl', MASTER_PASSWORD='repl' FOR CHANNEL 'group_replication_recovery';
    INSTALL PLUGIN group_replication SONAME 'group_replication.so';

    /*Should see group_replication plugin at bottom*/
    SHOW PLUGINS; 
    ```
1. Run the below in SQL to start group replication, but READ the DB number!
    ```sql
    /*DO THIS FIRST*/
    /*Master DB or DB1*/
    SET GLOBAL group_replication_bootstrap_group=ON;
    START GROUP_REPLICATION;
    SET GLOBAL group_replication_bootstrap_group=OFF;
    ```

    ```sql
    /*Slave DB or DB2..N*/
    START GROUP_REPLICATION;
    ```
1. (OPTIONAL) To let it join on boot
    ```bash
    sudo nano /etc/mysql/my.cnf

    # Change the below from OFF to ON
    # loose-group_replication_start_on_boot = ON
    ```
1. (OPTIONAL) Create a user that is given permission to the db from your server webapp
    ```sql
    /*Change details below*/
    CREATE USER 'user'@'%' IDENTIFIED BY 'password';
    GRANT SELECT ON playground.* TO 'user'@'%';
    FLUSH PRIVILEGES;
    ```

## Credits: 

Vong Chanvichet (NanoCode012)

Github: https://github.com/NanoCode012/DistributedSetup

## References:

https://www.digitalocean.com/community/tutorials/how-to-configure-mysql-group-replication-on-ubuntu-16-04