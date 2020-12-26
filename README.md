# iRedMail Docker Container #

iRedMail allows deployment of an OPEN SOURCE, FULLY FLEDGED, FULL-FEATURED mail server in several minutes, for free. If several minutes is long time then this docker container can reduce the deployment time and help you to get a mail server in the matter of seconds.

The current version of container uses MySQL for accounts saving. In the future the LDAP can be used, so pull requests are welcome. Container contains all components (Postfix, Dovecot, Fail2ban, ClamAV, Roundcube, and SoGo) and MySQL server. The hostname of the mail server can be set using the normal docker methods (```docker run -h <host>``` or setting 'hostname' in a docker compose file). In order to customize the container several environmental variables are allowed:

  * MYSQL_ROOT_PASSWORD - Root password for MySQL server installation
  * POSTMASTER_PASSWORD - Initial password for postmaster@DOMAIN. Password can be generated according to [wiki](http://www.iredmail.org/docs/reset.user.password.html). ({PLAIN}password)
  * TZ - Container timezone that is propagated to other components
  * SOGO_WORKERS - Number of SOGo workers which can affect SOGo interface performance.

Container is prepared to handle data as persistent using mounted folders for data. Folders prepared for initialization are:PATH/

 * /var/lib/mysql
 * /var/vmail
 * /var/lib/clamav

With all information prepared, let's test your new iRedMail server:
这里停掉了原有的自动生成证书，将已有的证书挂载进去
另外80/443已经被占用了，换8080/8443使用
时区修改的话 -e "TZ=Asia/Shanghai" 默认是Shanghai
```
docker run -p 8080:80 -p 8443:443 \
           -h HOSTNAME.DOMAIN \
           -e "MYSQL_ROOT_PASSWORD=passw0rd" \
           -e "SOGO_WORKERS=1" \
           -e "POSTMASTER_PASSWORD={PLAIN}passw0rd" \
           -e "IREDAPD_PLUGINS=['reject_null_sender', 'reject_sender_login_mismatch', 'greylisting', 'throttle', 'amavisd_wblist', 'sql_alias_access_policy']" \
           -v /srv/iredmail/mysql:/var/lib/mysql \
           -v /srv/iredmail/vmail:/var/vmail \
           -v /srv/iredmail/clamav:/var/lib/clamav \
           -v host.key:/etc/ssl/private/host.key \
           -v host.crt:/etc/ssl/certs/host.crt \
           -v host.pem:/var/lib/dkim/host.pem \
           --name=iredmail orzzz0/iredmail:v1.3.1

```

## Upgrade from version 1.0 or above

The iRedMail container gained automatic database schema migration with version 1.3 of the iRedMail container. What does it mean? It means upgrades should be smooth, and one wont no longer need to care about studying [release notes](https://docs.iredmail.org/iredmail.releases.html). 

If you are running and older version of the container the automatic upgrade needs to be activated by installation of control table in vmail database using the following steps.

```
-- Switch to vmail database
use vmail

-- Create version tracking table
CREATE TABLE IF NOT EXISTS `versions` (
    `component` varchar(120) NOT NULL,
    `version` varchar(20) NOT NULL,
    PRIMARY KEY(`component`)
);

-- Insert initial line representing installed version
INSERT INTO versions VALUES('iredmail', 'YOUR_CURRENT_VERSION');
```

The next step is just to upgrade the container version.

### Notes for contributors

When a new version of iRedMail gets released and I am not providing the upgrade. Feel free to open a pull request with migration stored in `mysql/static_files/opt/iredmail/migrations/DATABASE`. The file name should follow this schema:

```INDEX_IREDMAILVERSION__SHORTDESCRIPTION.sql```
