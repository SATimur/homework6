# Управление пакетами

1. Создать свой RPM пакет
2. Разместить в своем репозитории

#### Создаем свой RPM пакет

### Создаем Vagrantfile

```
   # -*- mode: ruby -*-
   # vi: set ft=ruby :

   Vagrant.configure("2") do |config|
     config.vm.box = "centos/7"
     config.vm.box_version = "2004.01"

    config.vm.provider "virtualbox" do |v|
       v.memory = 256
       v.cpus = 1
    end
    config.vm.define "nginx" do |nginx|
     nginx.vm.network "private_network", ip: "192.168.56.10",
     virtualbox_intnet: "net1"
       nginx.vm.hostname = "nginx"
    end
  
      config.vm.provision "shell", inline: <<-SHELL
       yum install -y \
         redhat-lsb-core \
         wget \
         rpmdevtools \
         rpm-build \
         createrepo \
         yum-utils \
         yum update
     #   apt-get update
     #   apt-get install -y apache2
     SHELL
   end
```
   
 Запустим машину и подключимся к ней
   <vagrant up>
   <vagrant ssh nginx>
   
 Загрузим SRPM пакет NGINX и соберем его с поддержкой openssl(в точности как описано в методичке)
   <wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm>
   
 Устанавливаем скаченый пакет nginx
   <rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm>
   
 Скачиваем последний исходник для openssl
   <wget https://ftp.openssl.org/source/openssl-1.1.1m.tar.gz>
   <tar -xvf openssl-1.1.1m.tar.gz>
 
 Поставим все зависимости
   <yum-builddep rpmbuild/SPECS/nginx.spec>
   
 Собираем RPM пакет
 <Вывод очень длинный>
```   
   [root@nginx ~]#  rpmbuild -bb ~/rpmbuild/SPECS/nginx.spec
   Выполняется(%clean): /bin/sh -e /var/tmp/rpm-tmp.r9kwMR
   + umask 022
   + cd /root/rpmbuild/BUILD
   + cd nginx-1.14.1
   + /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.14.1-1.el7_4.ngx.x86_64
   + exit 0
```
 Проверяем сборку
```
   [root@nginx ~]#  ll rpmbuild/RPMS/x86_64/
   итого 3072
   -rw-r--r--. 1 root root  769964 май 18 12:12 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
   -rw-r--r--. 1 root root 2375100 май 18 12:12 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm
```

 Устанавливаем, запускаем и проверяем пакет
```
   [root@nginx ~]#  yum localinstall -y \
   > rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
   Загружены модули: fastestmirror
   Проверка rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm: 1:nginx-1.14.1-1.el7_4.ngx.x86_64
   rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm отмечен для установки
   Разрешение зависимостей
   --> Проверка сценария
   ---> Пакет nginx.x86_64 1:1.14.1-1.el7_4.ngx помечен для установки
   --> Проверка зависимостей окончена
   Установлено:
     nginx.x86_64 1:1.14.1-1.el7_4.ngx  
   [root@nginx ~]# systemctl start nginx
   [root@nginx ~]# systemctl status nginx
    ● nginx.service - nginx - high performance web server
       Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
       Active: active (running) since Ср 2022-05-18 12:14:19 UTC; 10s ago
         Docs: http://nginx.org/en/docs/
      Process: 18219 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
     Main PID: 18220 (nginx)
       CGroup: /system.slice/nginx.service
               ├─18220 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
               └─18221 nginx: worker process

   май 18 12:14:19 nginx systemd[1]: Starting nginx - high performance web server...
   май 18 12:14:19 nginx systemd[1]: Can't open PID file /var/run/nginx.pid (yet?) after start: No such file or directory
   май 18 12:14:19 nginx systemd[1]: Started nginx - high performance web server.
```

### Создаем свой репозиторий на локальной машине, публикуем наш пакет на git

 Создаем свой репозиторий
```
   [root@nginx ~]#  mkdir /usr/share/nginx/html/repo 
   [root@nginx ~]# cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
   [root@nginx ~]# wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noa
   --2022-05-18 12:18:28--  http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noa
   Распознаётся www.percona.com (www.percona.com)... 172.67.8.157, 104.22.8.28, 104.22.9.28, ...
   Подключение к www.percona.com (www.percona.com)|172.67.8.157|:80... соединение установлено.
   HTTP-запрос отправлен. Ожидание ответа... 301 Moved Permanently
   Адрес: https://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noa [переход]
   --2022-05-18 12:18:29--  https://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noa
   Подключение к www.percona.com (www.percona.com)|172.67.8.157|:443... соединение установлено.
   HTTP-запрос отправлен. Ожидание ответа... 404 Not Found
   2022-05-18 12:18:30 ОШИБКА 404: Not Found.
```
  <столкнулись с не актуальным пакетом, спасибо всем кто скинул актуальную ссылку>

```
   [root@nginx repo]# wget https://downloads.percona.com/downloads/percona-release/percona-release-1.0-9/redhat/percona-release-1.0-9.noarch.rpm
   --2022-05-18 18:56:05--  https://downloads.percona.com/downloads/percona-release/percona-release-1.0-9/redhat/percona-release-1.0-9.noarch.rpm
   Распознаётся downloads.percona.com (downloads.percona.com)... 162.220.4.221, 74.121.199.231, 162.220.4.222
   Подключение к downloads.percona.com (downloads.percona.com)|162.220.4.221|:443... соединение установлено.
   HTTP-запрос отправлен. Ожидание ответа... 200 OK
   Длина: 16664 (16K) [application/octet-stream]
   Сохранение в: «percona-release-1.0-9.noarch.rpm»

   100%   [=============================================================================================================================================================================================>] 16 664      73,2KB/s   за 0,2s   

   2022-05-18 18:56:07 (73,2 KB/s) - «percona-release-1.0-9.noarch.rpm» сохранён [16664/16664]
```
 
 Инициализируем репозиторий командой:
```
   [root@nginx repo]# createrepo /usr/share/nginx/html/repo/
   Spawning worker 0 with 2 pkgs
   Workers Finished
   Saving Primary metadata
   Saving file lists metadata
   Saving other metadata
   Generating sqlite DBs
   Sqlite DBs complete
```
 Открываем доступ NGINX к листингу каталога ==>
```
   [root@nginx repo]# nano /etc/nginx/conf.d/default.conf
   location / {
   root /usr/share/nginx/html;
   index index.html index.htm;
   autoindex on; 
```

 Проверяем синтаксис и ребутаем NGINX
```
   [root@nginx repo]# nginx -t
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   [root@nginx repo]# nginx -s reload
```

 Curl-анем:
```
   [root@nginx repo]# curl -a http://localhost/repo
   <html>
   <head><title>301 Moved Permanently</title></head>
   <body bgcolor="white">
   <center><h1>301 Moved Permanently</h1></center>
   <hr><center>nginx/1.14.1</center>
   </body>
   </html>
```
 
 Все готово для теста, добавим его в /etc/yum.repos.d:
```
   [root@nginx repo]# cat >> /etc/yum.repos.d/otus.repo << EOF
   > [otus]
   > name=otus-linux
   > baseurl=http://localhost/repo
   > gpgcheck=0
   > enabled=1
   > EOF
   [root@nginx repo]# yum repolist enabled | grep otus
   otus                                   otus-linux                           2
   [root@nginx repo]# yum list | grep otus
   percona-release.noarch                      1.0-9                      otus     
```

 Т.к. NGINX установлен, установим репозиторий percona-release:
```
   [root@nginx repo]# yum install percona-release -y
   Установлено:
     percona-release.noarch 0:1.0-9                                                                                  

   Выполнено!
```

  Теперь запушим наш пакет на git:
1. Клонируем наш репозиторий 
```
   [root@nginx repo]# git clone git@github.com:SATimur/homework6
   Cloning into 'homework6'...
   The authenticity of host 'github.com (140.82.121.4)' can't be established.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added 'github.com,140.82.121.4' (ECDSA) to the list of known hosts.
   remote: Enumerating objects: 3, done.
   remote: Counting objects: 100% (3/3), done.
   remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
   Receiving objects: 100% (3/3), done.
```   
2. Двигаем файл в каталог /homework6
 <[root@nginx repo]# mv nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/homework6>

3.  Пушим наш пакет:
```
   [root@nginx homework6]# git add nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
   [root@nginx homework6]# git commit -m "hw"
   [main 78568e1] hw
    1 file changed, 0 insertions(+), 0 deletions(-)
    create mode 100644 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
   [root@nginx homework6]# git push
   warning: push.default is unset; its implicit value is changing in
   Git 2.0 from 'matching' to 'simple'. To squelch this message
   and maintain the current behavior after the default changes, use:

     git config --global push.default matching 

   To squelch this message and adopt the new behavior now, use:

     git config --global push.default simple

   See 'git help config' and search for 'push.default' for further information.
   (the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
   'current' instead of 'simple' if you sometimes use older versions of Git)

   Counting objects: 4, done.
   Compressing objects: 100% (3/3), done.
   Writing objects: 100% (3/3), 742.70 KiB | 0 bytes/s, done.
   Total 3 (delta 0), reused 0 (delta 0)
   To git@github.com:SATimur/homework6
      86f6f5a..78568e1  main -> main
```

#### Готово
   
   
