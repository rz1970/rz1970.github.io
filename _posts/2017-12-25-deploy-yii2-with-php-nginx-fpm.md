---
title: deploy php + nginx + php-fpm + yii2
tags: ["php", "yii2"]
---

根据Ref1 部署Yii2。注意：fastcgi_pass 修改为：fastcgi_pass unix:/var/run/php5-fpm.sock;

根据Ref2 了解Nginx的命令。
To start nginx, run the executable file. Once nginx is started, it can be controlled by invoking the executable with the -s parameter. Use the following syntax:`nginx -s signal`

Where signal may be one of the following:
* stop — fast shutdown
* quit — graceful shutdown
* reload — reloading the configuration file
* reopen — reopening the log files

Reference:
1. [Yii2 Reference](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)
2. [Nginx](https://nginx.org/en/docs/)

## Update:
当前通过Docekr+Volume的方式来实现开发环境的搭建