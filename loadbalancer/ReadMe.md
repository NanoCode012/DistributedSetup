## Loadbalancer setup

1. Boot up an Ubuntu 20.04 VM
1. Run the below
    ```bash
    sudo apt update
    sudo apt-get install -y haproxy
    sudo sed -i "s/ENABLED=0/ENABLED=1/g" /etc/default/haproxy
    sudo /etc/init.d/haproxy start
    sudo nano /etc/haproxy/haproxy.cfg
    ```
1. Add the following config. Open port `80`.
    ```bash
    global
        log /dev/log	local0
        log /dev/log	local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        ## Comment the below for non-SSL
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets
        tune.ssl.default-dh-param 2048

    defaults
        log	global
        mode	http
        option	httplog
        option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
    frontend client
        bind 192.168.57.8:443 ssl crt /etc/ssl/certs/haproxy.pem
        default_backend webapps
        option forwardfor
    backend webapps
        # (OPTIONAL) Change balance mode
        balance roundrobin

        # Change details to use name and real local ip
        server  app01   192.168.59.6:80 check
        server  app02   192.168.60.4:80 check

    ## Uncomment the below to open endpoint to see stats. Make sure to change details
    #listen stats
    #    bind 192.168.57.8:8443 ssl crt /etc/ssl/certs/haproxy.pem
    #    stats enable                    # enable statistics reports  
    #    stats hide-version              # Hide the version of HAProxy
    #    stats refresh 30s               # HAProxy refresh time
    #    stats show-node                 # Shows the hostname of the node
    #    stats auth haadmin:P@ssword     # Enforce Basic authentication for Stats page
    #    stats uri /stats                # Statistics URL
    ```
1. Run the below to restart and check status
    ```bash
    sudo /etc/init.d/haproxy restart
    sudo update-rc.d haproxy defaults
    sudo service haproxy status
    ```
1. _STEPS FROM HERE TO BELOW ARE OPTIONAL TO SETUP SSL_
1. Run the below
    ```bash
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot

    sudo systemctl stop haproxy

    sudo certbot certonly --standalone

    sudo mkdir -p /etc/haproxy/certs
    ```
1. Change domain name and run
    ```bash
    DOMAIN='example.com' sudo -E bash -c 'cat /etc/letsencrypt/live/$DOMAIN/fullchain.pem /etc/letsencrypt/live/$DOMAIN/privkey.pem > /etc/haproxy/certs/$DOMAIN.pem'
    sudo chmod -R go-rwx /etc/haproxy/certs
    
    sudo nano /etc/haproxy/haproxy.cfg
    ```
1. Update the config. REMOVE ALL FRONTEND AND BACKEND from earlier! Open port `54321`
    ```bash
    # Set in global
    maxconn 2048
    tune.ssl.default-dh-param 2048

    # Set in defaults
    option forwardfor
    option http-server-close

    # Set below
    frontend www-http
        bind 10.148.0.5:80
        reqadd X-Forwarded-Proto:\ http
        default_backend www-backend

    frontend www-https
        # Change the local ip and domain name
        bind 10.148.0.5:443 ssl crt /etc/haproxy/certs/DOMAIN.COM.pem
        reqadd X-Forwarded-Proto:\ https

        # Test URI to see if its a letsencrypt request
        acl letsencrypt-acl path_beg /.well-known/acme-challenge/
        use_backend letsencrypt-backend if letsencrypt-acl

        default_backend www-backend

    # LE Backend
    backend letsencrypt-backend
        server letsencrypt 127.0.0.1:54321

    backend www-backend
        redirect scheme https if !{ ssl_fc }
        # Optional to change balance mode
        balance roundrobin
        # Make sure to update localip
        server  web2   10.148.0.4:80 check
        server  web3   10.148.0.8:80 check
    ```
1. Run the command to restart HaProxy.
    ```bash
    sudo systemctl restart haproxy
    ```

## Credits: 

Github: https://github.com/NanoCode012/DistributedSetup

## References:
https://kifarunix.com/install-and-setup-haproxy-on-ubuntu-20-04/
https://certbot.eff.org/lets-encrypt/ubuntufocal-haproxy

https://www.digitalocean.com/community/tutorials/

how-to-secure-haproxy-with-let-s-encrypt-on-ubuntu-14-04