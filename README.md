# rsyslog
#### Настройка центрального лог сервера на прием логов   
##### Задачи    
a) все критичные логи с web сервера должны собираться и локально и удаленно    
b) все логи с nginx должны уходить на удаленный сервер (локально только критичные)   
c) логи аудита должны также уходить на удаленную систему   

`vagrant up` поднимает 2 машины:    
1 - log-server с ролью rsyslog    
  ------ip  192.168.111.16   
2 - web сервер nginx    
  ------ip  192.168.111.15   

##### Реализация задач. Настройка сервера rsyslog  
* настроить log-server на прием логов. Конфигурационный файл `/etc/rsyslog.conf`    
```
$ModLoad imtcp
$InputTCPServerRun 514
$ModLoad imudp
$UDPServerRun 514
$AllowedSender TCP, 192.168.111.15
$AllowedSender UDP, 192.168.111.15  
```   
* настройка лог сервера для приема событий аудита по `tcp 60` выполняется в конфигурационном файле `/etc/audit/auditd.conf` параметр:    
`tcp_listen_port = 60`

##### Реализация задач. Настройка веб-сервера   
a) конфигурационный файл копируется при поднятии vagrant по пути `/etc/rsyslog.d/crit.conf` с параметром:    
`*.crit @@192.168.111.16:514`   
дополнительно в nginx.conf указать `error_log /var/log/nginx/error.log crit;`   

b) nginx с версии 1.5.2 умеет посылать логи напрямую с сервера. Прописать в nginx.conf   
```
error_log syslog:server=192.168.111.16:514,tag=nginx_error;
access_log  syslog:server=192.168.111.16:514,facility=local6,tag=nginx_access,severity=info main;
```   
c) аудит настроен на передачу событий не сохраняя их локально в журнале средствами утилиты audisp.    
* настроить отслеживание изменения файла nginx. Для этого добавим правило. В `/etc/audit/rules.d/audit.rules`   
```
-w /etc/nginx/nginx.conf -p wa
-w /etc/nginx/conf.d/*.conf -p wa
```   
* изменить /etc/audisp/plugins.d/au-remote.conf    
`active = yes`    
* файл настройки отправки аудита /etc/audisp/audisp-remote.conf    
```	
 скоприует  /etc/audisp/audisp-remote.conf
 скопирует /vagrant/audisp_client/au-remote.conf /etc/audisp/plugins.d/au-remote.conf
```
