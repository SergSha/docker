<h3>### Docker ###</h3>

<h4>Описание домашнего задания</h4>

<p>Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)</p>
<p>Определите разницу между контейнером и образом</p>
<p>Вывод опишите в домашнем задании.</p>
<p>Ответьте на вопрос: Можно ли в контейнере собрать ядро?</p>
<p>Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий.</p>

<h4>Установка Docker</h4>

<p>Установим пакет yum-utils:</p>

<pre>[user@localhost ~]$ sudo yum install yum-utils -y
[sudo] password for user: 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * epel: mirror.logol.ru
 * extras: centos-mirror.rbc.ru
 * rpmfusion-free-updates: mirror.yandex.ru
 * updates: centos-mirror.rbc.ru
https://rpm.releases.hashicorp.com/RHEL/7/x86_64/stable/repodata/repomd.xml: [Errno 14] HTTPS Error 403 - Forbidden
Trying other mirror.
To address this issue please refer to the below wiki article

https://wiki.centos.org/yum-errors

If above article doesn't help to resolve this issue please use https://bugs.centos.org/.

Package yum-utils-1.1.31-54.el7_8.noarch already installed and latest version
Nothing to do
[user@localhost ~]$</pre>

<p>Как видим, пакет yum-utils уже установлен.</p>

<p>Добавим официальный репозиторий Docker в систему:</p>

<pre>[user@localhost ~]$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[user@localhost ~]$</pre>

<p>Установим Docker:</p>

<pre>[user@localhost ~]$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
...
Installed:
  containerd.io.x86_64 0:1.6.6-3.1.el7                                          
  docker-ce.x86_64 3:20.10.17-3.el7                                             
  docker-ce-cli.x86_64 1:20.10.17-3.el7                                         
  docker-compose-plugin.x86_64 0:2.6.0-3.el7                                    

Dependency Installed:
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8                            
  docker-ce-rootless-extras.x86_64 0:20.10.17-3.el7                             
  docker-scan-plugin.x86_64 0:0.17.0-3.el7                                      
  fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                                         
  fuse3-libs.x86_64 0:3.6.1-4.el7                                               
  slirp4netns.x86_64 0:0.4.3-4.el7_8                                            

Complete!
[user@localhost ~]$</pre>

<p>В домашней директории создадим каталог docker:</p>

<pre>[user@localhost otus]$ mkdir ./docker
[user@localhost otus]$</pre>

<p>Перейдём в этот каталог:</p>

<pre>[user@localhost otus]$ cd ./docker/
[user@localhost docker]$</pre>

<p>В этом каталоге создадим файл Dockerfile:</p>

<pre>[user@localhost ansible]$ vi ./Dockerfile</pre>

<p>Заполним его содержимым из https://gist.github.com/lalbrekht/f811ce9a921570b1d95e07a7dbebeb1e:</p>

<pre># -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :nginx => {
        :box_name => "centos/7",
        :ip_addr => '192.168.56.150'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "200"]
          end
          
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
          SHELL

      end
  end
end
</pre>

<p>Запустим систему:</p>

<pre>[user@localhost ansible]$ vagrant up</pre>

<p>Смотрим статус запущенной ВМ:</p>

<pre>[user@localhost ansible]$ vagrant status
Current machine states:

nginx                     running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
[user@localhost ansible]$</pre>

<p>Пробуем подключиться к ней с помощью SSH:</p>

<pre>[user@localhost ansible]$ vagrant ssh
[vagrant@nginx ~]$</pre>

<p>Как видим, подключение к этой ВМ по SSH прошло успешно.</p>

<p>Для подключения к хосту nginx нам необходимо будет передать множество параметров - это особенность Vagrant. Узнать эти параметры можно с помощью команды vagrant ssh-config:</p>

<pre>[user@localhost ansible]$ vagrant ssh-config
Host nginx  <----- имя хоста
  HostName 127.0.0.1  <----- IP адрес
  User vagrant  <---- имя пользователя, под которым подключаемся
  Port 2222 <---- порт, который проброшен на 127.0.0.1
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/user/otus/ansible/.vagrant/machines/nginx/virtualbox/private_key <----- путь до приватного ключа
  IdentitiesOnly yes
  LogLevel FATAL

[user@localhost ansible]$</pre>

<p>Используя эти параметры создадим свой первый inventory файл. Предварительно создадим директории staging/hosts:</p>

<pre>[user@localhost ansible]$ mkdir -p ./staging/hosts
[user@localhost ansible]$</pre>

<pre>[user@localhost ansible]$ vi ./staging/hosts/inventory</pre>
