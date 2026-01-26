# mailstack
tine groupware basic mailstack

## tine docs

https://tine-docu.s3web.rz1.metaways.net/operators/howto/MailserverIntegration/

## some inspiration

https://github.com/setiseta/docker-mailstack

# todo
- dovecote works with tine
    - could not test
        - if it receives emails from postfix
        - or if sassl auth is working
- postfix is starting
- postfix dose not work

# setup
```bash
DB_HOST=127.0.0.1 DB_ROOT_PW=root DB_USER=mail DB_PASS=mail
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" -e"CREATE DATABASE IF NOT EXISTS dovecot"
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" -e"CREATE DATABASE IF NOT EXISTS postfix"
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" -e"CREATE USER IF NOT EXISTS '$DB_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD';"
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" -e"GRANT ALL PRIVILEGES ON dovecot.* TO '$DB_USER'@'%'"
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" -e"GRANT ALL PRIVILEGES ON postfix.* TO '$DB_USER'@'%'"

mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" postfix
    INSERT IGNORE INTO `smtp_virtual_domains` (`domain`, `instancename`) VALUES ('mailtest.local.tine-dev.de', 'localhost');
    exit

mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" dovecot < ~/workspace/gitlab.metaways.net/tine20/docker/tine20/etc/sql/dovecot_tables.sql
mysql -h$DB_HOST -uroot -p"$DB_ROOT_PW" postfix < ~/workspace/gitlab.metaways.net/tine20/docker/tine20/etc/sql/postfix_tables.sql

mysql -h$DB_HOST -u$DB_USER -p"$DB_PASS" postfix # Access denied for user 'mail'@'172.18.0.1' (using password: YES)

echo -e 'server dns0.metaways.net\nupdate add _acme-challenge.mx.mailtest.local.tine-dev.de. 60 txt cC-4F9BvojqBupgkiiBBSlVfrPcIAimOhOslpMQWuPI\nsend' | nsupdate -k ~/.mwclouddns
echo -e 'server dns0.metaways.net\nupdate add mailtest.local.tine-dev.de. 60 MX 10 mx.mailtest.local.tine-dev.de.\nsend' | nsupdate -k ~/.mwclouddns

echo -e 'server dns0.metaways.net\nupdate add mx.mailtest.local.tine-dev.de. 60 A 192.168.0.190\nsend' | nsupdate -k ~/.mwclouddns
echo -e 'server dns0.metaways.net\nupdate add mailtest.local.tine-dev.de. 60 A 192.168.0.190\nsend' | nsupdate -k ~/.mwclouddns

docker compose exec dovecot bash -c "chown -R mail:mail /var/mail/"
docker compose run postfix bash -c "mkdir -p /var/spool/postfix/queue"
docker compose run postfix bash -c "chown postfix:postdrop /var/spool/postfix/queue/"
```

tine config:
dio
```diff
diff --git a/docs/operators/docker/docker-compose.yml b/docs/operators/docker/docker-compose.yml
index 78332bf753..edfaaecda6 100644
--- a/docs/operators/docker/docker-compose.yml
+++ b/docs/operators/docker/docker-compose.yml
@@ -10,11 +10,14 @@ services:
       MYSQL_PASSWORD: &MYSQL_PASSWORD tine
       MARIADB_AUTO_UPGRADE: 1
     ### use volume for persistent DB
+    ports:
+      - 3306:3306
     volumes:
       - "tine_db:/var/lib/mysql"
     ### OR
 #      - "./data/tine_mysql:/var/lib/mysql"
     networks:
+      - external_network
       - internal_network
```

```php
<?php

return  [
    "imap" => [
        "active" => true,
        "backend" => "dovecot_imap",
        "domain" => "mailtest.local.tine-dev.de",
        "dovecot" => [
            "dbname" => "dovecot",
            "gid" => "mail",
            "home" => "/var/mail/%d/%u",
            "host" => "db",
            "password" => "root",
            "scheme" => "SSHA256",
            "uid" => "mail",
            "username" => "root",
        ],
        "host" => "mx.mailtest.local.tine-dev.de",
        "instanceName" => "mailtest.local.tine-dev.de",
        "port" => 993,
        "ssl" => "SSL",
        "useSystemAccount" => 1,
        "verifyPeer" => 1,
    ],
    "smtp" => [
        "active" => true,
        "auth" => "login",
        "backend" => "postfix",
        "from" => "mailtest.local.tine-dev.de",
        "hostname" => "mx.mailtest.local.tine-dev.de",
        "instanceName" => "mailtest.local.tine-dev.de",
        "name" => "mx.mailtest.local.tine-dev.de",
        "port" => 25,
        "postfix" => [
            "dbname" => "postfix",
            "host" => "db",
            "password" => "root",
            "username" => "root",
        ],
        "primarydomain" => "mailtest.local.tine-dev.de",
        "ssl" => "tls",
    ],
];
```

``` bash
# cleanup dns
echo -e 'server dns0.metaways.net\nupdate del _acme-challenge.mx.mailtest.local.tine-dev.de. 60 txt cC-4F9BvojqBupgkiiBBSlVfrPcIAimOhOslpMQWuPI\nsend' | nsupdate -k ~/.mwclouddns
echo -e 'server dns0.metaways.net\nupdate del mailtest.local.tine-dev.de. 60 MX 10 mx.mailtest.local.tine-dev.de.\nsend' | nsupdate -k ~/.mwclouddns
echo -e 'server dns0.metaways.net\nupdate del mx.mailtest.local.tine-dev.de. 60 A 192.168.0.190\nsend' | nsupdate -k ~/.mwclouddns
echo -e 'server dns0.metaways.net\nupdate del mailtest.local.tine-dev.de. 60 A 192.168.0.190\nsend' | nsupdate -k ~/.mwclouddns
```
