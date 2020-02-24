В каталоге /manual находятся инструменты и описание выполнени задачи

# Обновление ядра linux  

Обновить ядро в базовой системе.  
Полученный в ходе выполнения ДЗ Vagrantfile должен быть залит в ваш репозиторий. Для проверки ДЗ необходимо прислать ссылку на него.  
Для выполнения ДЗ со * и ** вам потребуется сборка ядра и модулей из исходников.  
\* : Ядро собрано из исходников  
** : В вашем образе нормально работают VirtualBox Shared Folders  

# Решение  
В решении описанa непосредственно сборка ядра на стенде [/1.kernel_update/manual/Vagrantfile](https://github.com/dbudakov/1.kernel_update/manual.git) для описания установки VirtualBox, Vagrant, Packer, и настройки Git, смотрите файлы с соответствующими именами разделе [/support](https://github.com/dbudakov/support.git).   
  
Поднимаем ВМ, выполняя указанную команду из тестовой директории с Vagrantfile файлом  
```
vagrant up 
vagrant ssh  
```
Понадобиться утилита wget (или её альтернатива сurl)  
```
sudo su -
yum install wget -y
```
Создаем и переходим в каталог для сборки.  
``` 
mkdir /opt/src/kernel -p && cd /opt/src/kernel
```  



Далее качиваем rpm пакет с наличием **__исходного кода ядра__** с одного из указанных ресурсов, и достаем исходники   
```
#wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.213.tar.xz  
#tar -xvf linux-4.4.213.tar.xz  
wget https://git.kernel.org/torvalds/t/linux-5.6-rc3.tar.gz
tar -xvf linux-5.6-rc3.tar.gz

```
В дирректории появится новый каталог с именем пакета, переходим в него и копируем действующий конфиг ядра  
```
cd linux-5.6-rc3  
cp /boot/config-`uname -r` ./.config  
```
альтернативный источник исходников [_vault.centos.org_](http://vault.centos.org/)  
```
wget http://vault.centos.org/7.0.1406/updates/Source/SPackages/kernel-3.10.0-123.1.2.el7.src.rpm
rpm2cpio kernel-3.10.0-123.1.2.el7.src.rpm | cpio -t | grep tar
rpm2cpio kernel-3.10.0-123.1.2.el7.src.rpm | cpio -iv linux-3.10.0-123.1.2.el7.tar.xz  
tar -xvf linux-3.10.0-123.1.2.el7.tar.xz  
```

Для запуска **__make oldconfig__** потебуется _gcc_ , первая команда проверит возможна ли сборка, появиться настройка новых опций, можно передать параметр для ответа за паросы  
```
yum install gcc flex bison -y 
make oldconfig .config 
или
make olddefconfig .config
или
yes "" | make oldconfig
```
Далее запуск **__make__** , в несколько потоков, _могут потребоваться следующие пакеты:_  
```  
yum install elfutils-libelf-devel openssl-devel bc perl -y
#yum install openssl-devel  
#yum install bc  
#yum install perl  

make -j[N]
```
После успешной сборки запускаем `make -j[N] install`  , предварительно запустив `make -j[N] modules_install`
```
make -j[N] modules_install  
make -j[N] install  
```
ядро устанавливается сразу в каталог _`/boot`_ после установки остаеся обновить настройки _`grub2`_ и установить ядро по умолчанию  
```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  
sudo grub2-set-default 0  
sudo reboot  
```
проверить ядро можно командой `uname -r`  
```
uname -r  
```
#### Настройка VirtualBox Shared Folders  
На ВМ с собранным ядром из исходников добавляем Shared-Folders в _`Machine-Settings_Shared Folders`_ и _cd-rom_ через _`Machine-Settings_Storage`_, загружаем её, модуль отвечающий за cd-rom называется sr_mod, прожимаем в меню инструментов _`Devices-Insert Guest Additions...`_ монтируем _cd-rom_ запускаем _`/cdrom/VBoxLinuxAdditions.run`_ , в ходе установки могут понадобиться пакеты с drm, находим их через _`yum search drm`_ , ставим всё что нашли, в идеале после установки назначенные _`Shared Folders`_ автоматически подцепятся, если при создании назначено автоматичесткое монтирование.
дополнительная информация по настройке на [странице](https://github.com/dbudakov/support/blob/master/virtualbox_shared_folder_centos.txt)
