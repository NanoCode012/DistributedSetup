## Webserver setup

1. Boot up an Ubuntu 20.04 VM
1. Run the below
    ```bash
    sudo apt update
    sudo apt -y install apache2 php memcached php-memcached libmemcached-tools php-mysql
    sudo nano /etc/apache2/apache2.conf
    ```
1. Update ServerName
    ```bash
    # For example
    # ServerName web1
    ```
1. Run the below to check that memcache exists
    ```bash
    php -m | grep memcache
    ```
1. Run the below to enable memcache
    ```bash
    sudo nano /etc/php/7.4/mods-available/memcache.ini
    ```
1. Update the settings to the following
    ```bash
    extension=memcache.so
    memcache.maxreclevel=0
    memcache.maxfiles=0
    memcache.archivememlim=0
    memcache.maxfilesize=0
    memcache.maxratio=0
    memcache.hash_strategy = consistent
    memcache.allow_failover = 1

    # This has to be number of web VM + 1
    # This is a long standing bug
    memcache.session_redundancy = 3
    ```
1. Run the below to enable session handling. Make sure to open port `11211`.
    ```bash
    sudo sed -i "s@session.save_handler = files@session.save_handler = memcached@" /etc/php/7.4/apache2/php.ini

    # Change the details below
    sudo sed -i 's@;session.save_path = *@session.save_path = "10.148.0.4:11211, 10.148.0.8:11211"@' /etc/php/7.4/apache2/php.ini
    ```
1. Edit memcache conf to change ip
    ```bash
    sudo nano /etc/memcached.conf 
    # Replace ip with localip
    ```
1. Restart services
    ```bash
    sudo systemctl restart apache2.service
    sudo systemctl restart memcached.service

    sudo systemctl enable apache2.service
    sudo systemctl enable memcached.service
    ```
1. Nano a php file
    ```bash
    sudo nano /var/www/html/session.php
    ```
1. Add the following details
    ```php
    <?php
    header('Content-Type: text/plain');
    session_start();

    if(!isset($_SESSION['visit']))
    {
        echo "This is NanoCode012! Thank you for visiting this server.\n";
        $_SESSION['visit'] = 0;
    }
    else
    {
        echo "You have visited this server ".$_SESSION['visit'] . " times. \n";
    }

    $_SESSION['visit']++;
    echo "Server IP: ".$_SERVER['SERVER_ADDR'] . "\n";
    echo "Client IP: ".$_SERVER['REMOTE_ADDR'] . "\n";
    print_r($_COOKIE);
    ?>
    ```
1. (OPTIONAL) Enable pdo_mysql and restart apache
    ```bash
    sudo sed -i "s@;extension=pdo_mysql@extension=pdo_mysql@" /etc/php/7.4/apache2/php.ini
    sudo systemctl restart apache2.service
    ```

## Credits: 

Github: https://github.com/NanoCode012/DistributedSetup