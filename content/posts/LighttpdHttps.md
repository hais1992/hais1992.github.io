---
title: Lighttpd配置Https
date: 2019-01-14T11:20:37+08:00
tags: [Lighttpd,Linux,Http]
categories: ["Linux"]
---

1、安装Lighttpd过程省略

2、编辑 /etc/lighttpd/lighttpd.conf 如下
``` bash
server.modules = (
        "mod_access",
        "mod_alias",
        "mod_compress",
        "mod_redirect",
)

server.document-root        = "/mnt/harddisk/www"
server.upload-dirs          = ( "/var/cache/lighttpd/uploads" )
server.errorlog             = "/var/log/lighttpd/error.log"
server.pid-file             = "/var/run/lighttpd.pid"
server.username             = "www-data"
server.groupname            = "www-data"
server.port                 = 80
server.max-worker           = 4

index-file.names            = ("/_h5ai/public/index.php", "index.php", "index.html", "index.lighttpd.html" )
url.access-deny             = ( "~", ".inc" )
static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

compress.cache-dir          = "/var/cache/lighttpd/compress/"
compress.filetype           = ( "application/javascript", "text/css", "text/html", "text/plain" )

# default listening port for IPv6 falls back to the IPv4 port
include_shell "/usr/share/lighttpd/use-ipv6.pl " + server.port
include_shell "/usr/share/lighttpd/create-mime.assign.pl"
include_shell "/usr/share/lighttpd/include-conf-enabled.pl"
```


3、配置/etc/lighttpd/conf-enabled/10-ssl.conf
``` bash
# /usr/share/doc/lighttpd/ssl.txt
#证书可到https://console.cloud.tencent.com/ssl 申请，使用Nginx根
$SERVER["socket"] == ":443" {
     ssl.engine = "enable"
     ssl.pemfile = "/mnt/harddisk/ssl/myk.pw.pem"
     $HTTP["host"] == "myk.pw" {
         ssl.pemfile = "/mnt/harddisk/ssl/myk.pw.pem"
         ssl.ca-file = "/mnt/harddisk/ssl/myk.pw_bundle.crt"
         server.name = "myk.pw"
         server.document-root = "/mnt/harddisk/myk.pw"
     }
     $HTTP["host"] == "d.myk.pw" {
         ssl.pemfile = "/mnt/harddisk/ssl/d.myk.pw.pem"
         ssl.ca-file = "/mnt/harddisk/ssl/d.myk.pw_bundle.crt"
         server.name = "d.myk.pw"
         server.document-root = "/mnt/harddisk/d.myk.pw"
     }
}

$SERVER["socket"] == ":80" {
    $HTTP["host"] =~ "([^:/]+)" {
        url.redirect = ( "^/(.*)" => "https://%0:443/$1" )
    }
}

```


4、假如需要目录验证编辑/etc/lighttpd/conf-enabled/05-auth.conf
``` bash
# /usr/share/doc/lighttpd/authentication.txt.gz

server.modules                += ( "mod_auth" )

 auth.backend                 = "plain"
 auth.backend.plain.userfile  = "/etc/lighttpd/lighttpd.user" #账号密码配置文件 格式 hais:000000 一行一个
# auth.backend.plain.groupfile = "lighttpd.group"

# auth.backend.ldap.hostname   = "localhost"
# auth.backend.ldap.base-dn    = "dc=my-domain,dc=com"
# auth.backend.ldap.filter     = "(uid=$)"

 auth.require                 = ("/个人临时文件/加密目录" =>
                                (
                                  "method"  => "basic",
                                  "realm"   => "Server Status WebSite",
                                  "require" => "user=hais"	
                                )
                              )


```