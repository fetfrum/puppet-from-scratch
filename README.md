# Puppet с нуля
Личные заметки по настройке инфраструктуры на Puppet

* VirtualBox
* 1 сервер Puppet
* 3 ноды
* Операционная система Debian

Устанавливаем операционную систему на серверы (или копируем существующую виртуальную машину).
Переходим в рута, тавим sudo, puppet и puppetmaster, добавляем требуемого пользователя в группу sudo. Дальше логинимся в этого пользователя и работаем в его окружении через sudo 

```
apt-get update
apt-get install sudo puppet puppetmaster
adduser username sudo
```
Создаем окружение puppet
```
mkdir /etc/puppet/environments
chgrp puppet /etc/puppet/environments
chmod 2775 /etc/puppet/environments
```
(2 в начале прав означает присвоении GID при выполнении)

Базовая конфигурация `/etc/puppet/puppet.conf`:
```
[main]
  environment   = production
  confdir       = /etc/puppet
  logdir        = /var/log/puppet
  vardir        = /var/lib/puppet
  ssldir        = $vardir/ssl
  rundir        = /var/run/puppet
  factpath      = $vardir/lib/facter
  templatedir   = $confdir/templates
  pluginsync    = true

[agent]
  environment   = production
  report        = true
  show_diff     = true

[master]
  environment   = production
  #manifest      = $confdir/environments/$environment/manifests/site.pp
  #modulepath    = $confdir/environments/$environment/modules:$confdir/environments/$environment/site
  # Passenger
  ssl_client_header        = SSL_CLIENT_S_DN
  ssl_client_verify_header = SSL_CLIENT_VERIFY
  
  ```
  * `manifest` и  `modulepath` заменили на окружения, `pluginsync` деприкейтед.
Если еще не настроено определение имени "puppet" в DNS, вы можете добавить `server = your.server.com` в секцию `[main]` (добавил).

### настройки hiera
Файл `/etc/puppet/hiera.yaml`

```
---
:hierarchy:
  - "nodes/%{::fqdn}"
  - "manufacturers/%{::manufacturer}"
  - "virtual/%{::virtual}"
  - common
:backends:
  - yaml
:yaml:
  :datadir: "/etc/puppet/environments/%{::environment}/hieradata"
```

