# Identité OS et socket MariaDB

## Interroger la base avec l’utilisateur courant

L'utilisateur courant étant vagrant , celui-ci n'a pas les droits nécessaire pour accéder à MariaDB.  

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

## Pourquoi vagrant est-il refusé et sudo -u appphp accepté ?

Vagrant ne possède pas les droits pour accéder à MariaDB, ceux-ci ne sont pas dépendant du unix-socket mais sont déterminé au niveau de ;a dB qui est configurée pour n'accepter que apphp et root.

```bash
vagrant@maintenance-sec:~$ sudo mysql -e "SHOW GRANTS FOR 'vagrant'@'localhost';"
ERROR 1141 (42000) at line 1: There is no such grant defined for user 'vagrant' on host 'localhost'
```

```bash
vagrant@maintenance-sec:~$ sudo ss -lxp
[...]
u_str LISTEN  0       80 /run/mysqld/mysqld.sock 9412 * 0     
users:(("mariadbd",pid=1587,fd=25))                 
[...]                          
```

On peut d'ailleurs constater que les droits sur le socket ne sont pas bloquant, et qu'il s'agit bien d'une restriction au niveau de la DB.
MariaDb fait confiance à l’identité OS transmise par le kernel lors de la connexion au socket.

```bash
vagrant@maintenance-sec:~$ ls -l /run/mysqld/mysqld.sock
srwxrwxrwx 1 mysql mysql 0 Nov 21 21:11 /run/mysqld/mysqld.sock

```

## sudo mysql (sans -u) passe : sous quelle identité ?

il Passe sous l'identité root 

```bash 
vagrant@maintenance-sec:~$ sudo mysql demo -e 'SELECT content FROM messages'
+------------------------------------+
| content                            |
+------------------------------------+
| Bonjour depuis la base de données  |
| Deuxième message de test           |
+------------------------------------+

vagrant@maintenance-sec:~$ sudo mysql -e "SELECT CURRENT_USER();"
+----------------+
| CURRENT_USER() |
+----------------+
| root@localhost |
+----------------+

```

## Le sudo “contourne”-t-il la base, ou change-t-il l’identité OS qu’elle lit ?

Il change l'identité OS , c'est pour cela que MariaDB accepte la connexion.

## Quel mécanisme relie identité OS et accès ?

c'est via l'Unix_socket `/run/mysqld/mysqld.sock`, lorsqu'un process se connecte, le kernel fourni à MariaDB les informations d'identité OS.
MariaDB vérifie alors que l'identité OS correspond à un compte autorisé (apphp ou root), sinon elle refuse l'authentification.

