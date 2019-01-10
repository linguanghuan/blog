# mysql安装

## 安装

上传解压

```
mysql-5.7.24-1.el7.x86_64.rpm-bundle.tar
[root@h104 ~]# tar xvf mysql-5.7.24-1.el7.x86_64.rpm-bundle.tar 
mysql-community-common-5.7.24-1.el7.x86_64.rpm
mysql-community-minimal-debuginfo-5.7.24-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.24-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.24-1.el7.x86_64.rpm
mysql-community-embedded-5.7.24-1.el7.x86_64.rpm
mysql-community-libs-5.7.24-1.el7.x86_64.rpm
mysql-community-devel-5.7.24-1.el7.x86_64.rpm
mysql-community-server-5.7.24-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.24-1.el7.x86_64.rpm
mysql-community-client-5.7.24-1.el7.x86_64.rpm
mysql-community-server-minimal-5.7.24-1.el7.x86_64.rpm
mysql-community-test-5.7.24-1.el7.x86_64.rpm
```



查看包里面有什么

rpm -qpl

```
[root@h104 mysql_rpm]# rpm -qpl mysql-community-libs-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-libs-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
/etc/ld.so.conf.d/mysql-x86_64.conf
/usr/lib64/mysql
/usr/lib64/mysql/libmysqlclient.so.20
/usr/lib64/mysql/libmysqlclient.so.20.3.11
/usr/share/doc/mysql-community-libs-5.7.24
/usr/share/doc/mysql-community-libs-5.7.24/COPYING
/usr/share/doc/mysql-community-libs-5.7.24/README
```



```
[root@h104 mysql_rpm]# rpm -qpl mysql-community-server-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
/etc/logrotate.d/mysql
/etc/my.cnf
/etc/my.cnf.d
/usr/bin/innochecksum
/usr/bin/lz4_decompress
/usr/bin/my_print_defaults
/usr/bin/myisam_ftdump
/usr/bin/myisamchk
/usr/bin/myisamlog
/usr/bin/myisampack
/usr/bin/mysql_install_db
/usr/bin/mysql_plugin
/usr/bin/mysql_secure_installation
/usr/bin/mysql_ssl_rsa_setup
/usr/bin/mysql_tzinfo_to_sql
/usr/bin/mysql_upgrade
/usr/bin/mysqld_pre_systemd
/usr/bin/mysqldumpslow
/usr/bin/perror
/usr/bin/replace
/usr/bin/resolve_stack_dump
/usr/bin/resolveip
/usr/bin/zlib_decompress
/usr/lib/systemd/system/mysqld.service
/usr/lib/systemd/system/mysqld@.service
/usr/lib/tmpfiles.d/mysql.conf
/usr/lib64/mysql/mecab
/usr/lib64/mysql/mecab/dic
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/char.bin
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/dicrc
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/left-id.def
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/matrix.bin
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/pos-id.def
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/rewrite.def
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/right-id.def
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/sys.dic
/usr/lib64/mysql/mecab/dic/ipadic_euc-jp/unk.dic
/usr/lib64/mysql/mecab/dic/ipadic_sjis
/usr/lib64/mysql/mecab/dic/ipadic_sjis/char.bin
/usr/lib64/mysql/mecab/dic/ipadic_sjis/dicrc
/usr/lib64/mysql/mecab/dic/ipadic_sjis/left-id.def
/usr/lib64/mysql/mecab/dic/ipadic_sjis/matrix.bin
/usr/lib64/mysql/mecab/dic/ipadic_sjis/pos-id.def
/usr/lib64/mysql/mecab/dic/ipadic_sjis/rewrite.def
/usr/lib64/mysql/mecab/dic/ipadic_sjis/right-id.def
/usr/lib64/mysql/mecab/dic/ipadic_sjis/sys.dic
/usr/lib64/mysql/mecab/dic/ipadic_sjis/unk.dic
/usr/lib64/mysql/mecab/dic/ipadic_utf-8
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/char.bin
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/dicrc
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/left-id.def
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/matrix.bin
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/pos-id.def
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/rewrite.def
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/right-id.def
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/sys.dic
/usr/lib64/mysql/mecab/dic/ipadic_utf-8/unk.dic
/usr/lib64/mysql/mecab/etc
/usr/lib64/mysql/mecab/etc/mecabrc
/usr/lib64/mysql/plugin
/usr/lib64/mysql/plugin/adt_null.so
/usr/lib64/mysql/plugin/auth_socket.so
/usr/lib64/mysql/plugin/authentication_ldap_sasl_client.so
/usr/lib64/mysql/plugin/connection_control.so
/usr/lib64/mysql/plugin/debug
/usr/lib64/mysql/plugin/debug/adt_null.so
/usr/lib64/mysql/plugin/debug/auth_socket.so
/usr/lib64/mysql/plugin/debug/authentication_ldap_sasl_client.so
/usr/lib64/mysql/plugin/debug/connection_control.so
/usr/lib64/mysql/plugin/debug/group_replication.so
/usr/lib64/mysql/plugin/debug/ha_example.so
/usr/lib64/mysql/plugin/debug/innodb_engine.so
/usr/lib64/mysql/plugin/debug/keyring_file.so
/usr/lib64/mysql/plugin/debug/keyring_udf.so
/usr/lib64/mysql/plugin/debug/libmemcached.so
/usr/lib64/mysql/plugin/debug/libpluginmecab.so
/usr/lib64/mysql/plugin/debug/locking_service.so
/usr/lib64/mysql/plugin/debug/mypluglib.so
/usr/lib64/mysql/plugin/debug/mysql_no_login.so
/usr/lib64/mysql/plugin/debug/mysqlx.so
/usr/lib64/mysql/plugin/debug/rewrite_example.so
/usr/lib64/mysql/plugin/debug/rewriter.so
/usr/lib64/mysql/plugin/debug/semisync_master.so
/usr/lib64/mysql/plugin/debug/semisync_slave.so
/usr/lib64/mysql/plugin/debug/validate_password.so
/usr/lib64/mysql/plugin/debug/version_token.so
/usr/lib64/mysql/plugin/group_replication.so
/usr/lib64/mysql/plugin/ha_example.so
/usr/lib64/mysql/plugin/innodb_engine.so
/usr/lib64/mysql/plugin/keyring_file.so
/usr/lib64/mysql/plugin/keyring_udf.so
/usr/lib64/mysql/plugin/libmemcached.so
/usr/lib64/mysql/plugin/libpluginmecab.so
/usr/lib64/mysql/plugin/locking_service.so
/usr/lib64/mysql/plugin/mypluglib.so
/usr/lib64/mysql/plugin/mysql_no_login.so
/usr/lib64/mysql/plugin/mysqlx.so
/usr/lib64/mysql/plugin/rewrite_example.so
/usr/lib64/mysql/plugin/rewriter.so
/usr/lib64/mysql/plugin/semisync_master.so
/usr/lib64/mysql/plugin/semisync_slave.so
/usr/lib64/mysql/plugin/validate_password.so
/usr/lib64/mysql/plugin/version_token.so
/usr/sbin/mysqld
/usr/sbin/mysqld-debug
/usr/share/doc/mysql-community-server-5.7.24
/usr/share/doc/mysql-community-server-5.7.24/COPYING
/usr/share/doc/mysql-community-server-5.7.24/ChangeLog
/usr/share/doc/mysql-community-server-5.7.24/INFO_BIN
/usr/share/doc/mysql-community-server-5.7.24/INFO_SRC
/usr/share/doc/mysql-community-server-5.7.24/README
/usr/share/man/man1/innochecksum.1.gz
/usr/share/man/man1/lz4_decompress.1.gz
/usr/share/man/man1/my_print_defaults.1.gz
/usr/share/man/man1/myisam_ftdump.1.gz
/usr/share/man/man1/myisamchk.1.gz
/usr/share/man/man1/myisamlog.1.gz
/usr/share/man/man1/myisampack.1.gz
/usr/share/man/man1/mysql.server.1.gz
/usr/share/man/man1/mysql_install_db.1.gz
/usr/share/man/man1/mysql_plugin.1.gz
/usr/share/man/man1/mysql_secure_installation.1.gz
/usr/share/man/man1/mysql_ssl_rsa_setup.1.gz
/usr/share/man/man1/mysql_tzinfo_to_sql.1.gz
/usr/share/man/man1/mysql_upgrade.1.gz
/usr/share/man/man1/mysqldumpslow.1.gz
/usr/share/man/man1/mysqlman.1.gz
/usr/share/man/man1/perror.1.gz
/usr/share/man/man1/replace.1.gz
/usr/share/man/man1/resolve_stack_dump.1.gz
/usr/share/man/man1/resolveip.1.gz
/usr/share/man/man1/zlib_decompress.1.gz
/usr/share/man/man8/mysqld.8.gz
/usr/share/mysql/dictionary.txt
/usr/share/mysql/fill_help_tables.sql
/usr/share/mysql/innodb_memcached_config.sql
/usr/share/mysql/install_rewriter.sql
/usr/share/mysql/magic
/usr/share/mysql/mysql-log-rotate
/usr/share/mysql/mysql_security_commands.sql
/usr/share/mysql/mysql_sys_schema.sql
/usr/share/mysql/mysql_system_tables.sql
/usr/share/mysql/mysql_system_tables_data.sql
/usr/share/mysql/mysql_test_data_timezone.sql
/usr/share/mysql/uninstall_rewriter.sql
/var/lib/mysql
/var/lib/mysql-files
/var/lib/mysql-keyring
/var/run/mysqld
```



安装rpm包

```
[root@h104 ~]# rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-libs-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
        mysql-community-common(x86-64) >= 5.7.9 被 mysql-community-libs-5.7.24-1.el7.x86_64 需要
        mariadb-libs 被 mysql-community-libs-5.7.24-1.el7.x86_64 取代
[root@h104 ~]# 
[root@h104 ~]# 
[root@h104 ~]# 
```



安装mysql-community-common-5.7.24-1.el7.x86_64.rpm

和mariadb-libs-1:5.5.56-2.el7.x86_64冲突，卸载掉mariadb-libs-1:5.5.56-2.el7.x86_64

```
[root@h104 ~]# rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-common-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
        file /usr/share/mysql/czech/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/danish/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/dutch/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/english/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/estonian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/french/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/german/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/greek/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/hungarian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/italian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/japanese/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/korean/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/norwegian-ny/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/norwegian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/polish/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/portuguese/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/romanian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/russian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/serbian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/slovak/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/spanish/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/swedish/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/ukrainian/errmsg.sys from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/Index.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/armscii8.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/ascii.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp1250.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp1256.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp1257.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp850.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp852.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/cp866.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/dec8.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/geostd8.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/greek.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/hebrew.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/hp8.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/keybcs2.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/koi8r.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/koi8u.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/latin1.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/latin2.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/latin5.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/latin7.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/macce.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/macroman.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
        file /usr/share/mysql/charsets/swe7.xml from install of mysql-community-common-5.7.24-1.el7.x86_64 conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_64
[root@h104 ~]# rpm -e mariadb-libs-1:5.5.56-2.el7.x86_64     
错误：依赖检测失败：
        libmysqlclient.so.18()(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要
        libmysqlclient.so.18(libmysqlclient_18)(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要
[root@h104 ~]# rpm -e postfix-2:2.10.1-6.el7.x86_64
[root@h104 ~]# rpm -e mariadb-libs-1:5.5.56-2.el7.x86_64
[root@h104 ~]# 
[root@h104 ~]# 
[root@h104 ~]# rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-common-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-5.7.24-1.e################################# [100%]
```



```

[root@h104 ~]# rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm --force --nodeps
警告：mysql-community-libs-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-5.7.24-1.el7################################# [100%]
[root@h104 ~]# rpm -ivh  mysql-community-client-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-client-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-5.7.24-1.e################################# [100%]

```



安装mysql-community-server-5.7.24-1.el7.x86_64.rpm

```
[root@h104 ~]# rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
        /usr/bin/perl 被 mysql-community-server-5.7.24-1.el7.x86_64 需要
        perl(Getopt::Long) 被 mysql-community-server-5.7.24-1.el7.x86_64 需要
        perl(strict) 被 mysql-community-server-5.7.24-1.el7.x86_64 需要
```

解决：

```
[root@h104 ~]# yum install perl-devel -y
```



再安装就成功了

```
[root@h104 ~]# rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.24-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-5.7.24-1.e################################# [100%]
```







## 配置

> 为了保证数据库目录为与文件的所有者为 mysql 登陆用户，如果你是以 root 身份运行 mysql 服务，需要执行下面的命令初始化
> mysqld --initialize --user=mysql
>
> 如果是以 mysql 身份运行，则可以去掉 `--user` 选项。
>
> 另外 `--initialize` 选项默认以“安全”模式来初始化，则会为 root 用户生成一个密码并将该密码标记为过期，登陆后你需要设置一个新的密码，而使用 `--initialize-insecure` 命令则不使用安全模式，则不会为 root 用户生成一个密码。
>
> 这里演示使用的 `--initialize` 初始化，会生成一个 root 账户密码，密码在log文件里，红色区域的就是自动生成的密码
>
> ```
> [root@node21 mysql]# mysqld --initialize --user=mysql
> ```
>
> mysql配置文件在/etc/my.cnf,里面指明了数据目录。日志位置



初始化mysql

```
[root@h104 ~]# mysqld --initialize-insecure --user=mysql
```



启动mysql

```
[root@h104 mysql_rpm]# service mysqld start
Redirecting to /bin/systemctl start mysqld.service
```



检查监听端口

```
[root@h104 mysql_rpm]# netstat -lntp|grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      2834/mysqld   
```



本地连接mysql

数据密码直接回车

```
[root@h104 mysql_rpm]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

mysql -uroot 也能直接进来

```
[root@h104 mysql_rpm]# mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.7.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

免密只能localhost连接

后续就可以在命令行操作，或者用xshell建立隧道，通过navicat连接操作



## 参考

[CentOS7.5安装Mysql5.7.22](https://www.cnblogs.com/frankdeng/p/9017384.html)



https://dev.mysql.com/doc/refman/5.7/en/

https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html





