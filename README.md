В каталоге /manual находятся инструменты и описание выполнени задачи

# Обновление ядра linux  

Обновить ядро в базовой системе.  
Полученный в ходе выполнения ДЗ Vagrantfile должен быть залит в ваш репозиторий. Для проверки ДЗ необходимо прислать ссылку на него.  
Для выполнения ДЗ со * и ** вам потребуется сборка ядра и модулей из исходников.  
\* : Ядро собрано из исходников  
** : В вашем образе нормально работают VirtualBox Shared Folders  

# Решение  
В решении описанa непосредственно сборка ядра на /manual/Vagrantfile для описания установки VirtualBox, Vagrant, Packer, и настройки Git, смотрите файлы с соответствующими именами разделе /support.   
  
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
mkdir /opt/src/kernel && cd /opt/src/kernel
```  
Далее качиваем rpm пакет с наличием **__исходного кода ядра__** с одного из указанных ресурсов, и достаем исходники   
```
wget http://vault.centos.org/7.0.1406/updates/Source/SPackages/kernel-3.10.0-123.1.2.el7.src.rpm
rpm2cpio kernel-3.10.0-123.1.2.el7.src.rpm | cpio -t | grep tar
rpm2cpio kernel-3.10.0-123.1.2.el7.src.rpm | cpio -iv linux-3.10.0-123.1.2.el7.tar.xz  
tar -xvf linux-3.10.0-123.1.2.el7.tar.xz  
```
или другой ресурс, но в решении используестся _vault.centos.org_
```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.213.tar.xz
tar -xvf linux-4.4.213.tar.xz
```
В дирректории появится новый каталог с именем пакета, переходим в него и копируем действующий конфиг ядра  
```
cd linux-3.10.0-123.1.2.el7
cp /boot/config"uname -r" ./.config
```
Для запуска **__make oldconfig__** потебуется _gcc_ , первая команда проверит возможна ли сборка, появиться настройка новых опций, можно передать параметр для ответа за паросы  
```
yum install gcc -y
make oldconfig .config 
or
make olddefconfig
yes "" | make oldconfig
```
Далее запуск **__make__** , в несколько потоков  
```  
make -j[N]  
```
