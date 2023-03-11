Home01
#Домашнее задание
##Обновить ядро в базовой системе

###Цель домашнего задания
###Научиться обновлять ядро в ОС Linux. Получение навыков работы с Vagrant, Packer и публикацией готовых образов в Vagrant Cloud. 

Описание домашнего задания
1) Обновить ядро ОС из репозитория ELRepo
2) Создать Vagrant box c помощью Packer
3) Загрузить Vagrant box в Vagrant Cloud
   
1) Обновить ядро ОС из репозитория ELRepo
   
выполнено отдельным файлом, пример файла

---
    #содзаем скрипт для отображения версии
    $uname = <<~SCRIPT
      uname -r
    SCRIPT
    MACHINES = {

      :"su" => {
        :box_name => "centos8-kernel5",
        :cpus => 2,
        :memory => 1024,
      }
    }
    Vagrant.configure("2") do |config|
      MACHINES.each do |boxname, boxconfig|
      #отключаем маппинг папки
        config.vm.synced_folder ".", "/vagrant", disabled: false
        config.vm.define boxname do |box|
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.provider "virtualbox" do |v|
            v.memory = boxconfig[:memory]
            v.cpus = boxconfig[:cpus]
          end
          #отображаем версию
        config.vm.provision "shell", inline: $uname
        end
      end
    end
---
2) Создать Vagrant box c помощью Packer  
   
   доработка методички. в ks.cfg добавлено:   

---
    #для обхода меню выбора пакетов
    %packages --ignoremissing
    # dnf group info minimal-environment
    @^minimal-environment
    %end
    %post
    #для доступа уз вагранта при подключении диска
    echo "vagrant ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/999-vagrant
    echo "Defaults:vagrant !requiretty" >> /etc/sudoers.d/999-vagrant
    chmod 440 /etc/sudoers.d/999-vagrant
    %end
---

в json файле измменено выполнение команд - добавлен вывод пароля при выполнении команд
`
execute_command": "{{.Vars}} echo 'vagrant' | sudo -S -E bash '{{.Path}}'",
`
так же для отключения виртуалки после окончания работ
`
"shutdown_command": "echo 'vagrant' | sudo -S /sbin/halt -h -p",
`
изменено имя вобраза под версию ядра 

После сборки
`
packer build centos.json
`
и добавления образа в vagrant, при запуске столкнулся с проблемой подключения дисков

```
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

yum install -y kernel-devel-`uname -r` --enablerepo=C*-base --enablerepo=C*-updates

Stdout from the command:



Stderr from the command:

Error: Unknown repo: 'C*-base'
```

решением стало отказаться от использования vagrant-vbguest, т.к. он пытался установиться при запуске, но из-за неверных репозиториев не мог этого выполнить. Нужную репу я не нашел, более того, эта ошибка осталась нерешенной и на форумах вагранта для Centos8, поэтому vbguest был отключен. Но он требуется для работы шар дисков, иначе получаем ошибку
```
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000,_netdev vagrant /vagrant

The error output from the command was:

mount: /vagrant: unknown filesystem type 'vboxsf'.

```
обходным путем стало использование модуля sftp
`
vagrant plugin install vagrant-sshfs
`
и установкой sftp сервера
`
sudo apt install openssh-server
`
после чего получаем рабочую система с маунтингом папки
```
    default: in which case you may ignore this message.
==> default: Installing SSHFS client...
==> default: Mounting SSHFS shared folder...
==> default: Mounting folder via SSHFS: /home/altemans/otus/home01 => /vagrant
==> default: Checking Mount..
==> default: Folder Successfully Mounted!
altemans@Home01:~/otus/home01$ vagrant ssh
Last login: Wed Mar  8 19:57:10 2023 from 10.0.2.2
```
3) Загрузить Vagrant box в Vagrant Cloud

`
vagrant cloud publish --release ilitar/centos8-kernel6 1.0 virtualbox centos-8-kernel-6-x86_64-Minimal.box --force --release
`
```
You are about to publish a box on Vagrant Cloud with the following options:
ilitar/centos8-kernel6:   (v1.0) for provider 'virtualbox'
Automatic Release:     true
Saving box information...
Uploading provider with file /home/altemans/otus/home01/packer/centos-8-kernel-6-x86_64-Minimal.box
Releasing box...
Complete! Published ilitar/centos8-kernel6
Box:              ilitar/centos8-kernel6
Description:      
Private:          yes
Created:          2023-03-11T18:37:50.401+03:00
Updated:          2023-03-11T18:37:51.895+03:00
Current Version:  N/A
Versions:         1.0
Downloads:        0
```
в аккаунте box переведен из приватного в публичный