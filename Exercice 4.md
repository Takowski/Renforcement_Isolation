# Identité OS et socket MariaDB

## Interroger la base avec l’utilisateur courant

L'utilisateur courant étant vagrant , celui-ci n'a pas les droits nécessaire pour accéder à *mariadb*.  

```bash
vagrant@maintenance-sec:~$ mysql demo -e 'SELECT content FROM messages'
ERROR 1698 (28000): Access denied for user 'vagrant'@'localhost'

```

## se substituer à ***appphp***

```bash
vagrant@maintenance-sec:~$ sudo -u appphp mysql demo -e 'SELECT content FROM messages'
+------------------------------------+
| content                            |
+------------------------------------+
| Bonjour depuis la base de données  |
| Deuxième message de test           |
+------------------------------------+

```

## Droits accordés, et identité vue par la base

```bash
vagrant@maintenance-sec:~$ sudo mysql -e "SHOW GRANTS FOR 'appphp'@'localhost';"
+-----------------------------------------------------------------------+
| Grants for appphp@localhost                                           |
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `appphp`@`localhost` IDENTIFIED VIA unix_socket |
| GRANT SELECT ON `demo`.`messages` TO `appphp`@`localhost`             |
+-----------------------------------------------------------------------+

```

```bash
vagrant@maintenance-sec:~$ sudo mysql -e "SELECT CURRENT_USER();"
+----------------+
| CURRENT_USER() |
+----------------+
| root@localhost |
+----------------+

```

```bash

```
```bash
vagrant@maintenance-sec:~$ sudo ss -lxp
[...]
u_str LISTEN  0       80 /run/mysqld/mysqld.sock 9412 * 0     
users:(("mariadbd",pid=1587,fd=25))                 
[...]                          
```

```bash
vagrant@maintenance-sec:~$ ls -l /run/mysqld/mysqld.sock
srwxrwxrwx 1 mysql mysql 0 Nov 21 21:11 /run/mysqld/mysqld.sock

```

