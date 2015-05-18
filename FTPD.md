# Как поднять FTP сервер на z/OS, запущенной на Hercules версии x64 под Windows 7 x64 #
Здесь и далее подразумевается, что Hercules запускается из своей директории:
```

D:
cd D:\program files\hercules\
start hercules.exe -f D:\emc\main\hercules.cnf
```
  * Установить Microsoft Windows Visual c++ 2012 redistributable для x64 и x86
  * Скачать этот архив https://code.google.com/p/emc-spbsu-coop-mainframe/source/browse/CTCI.7z?repo=wiki
  * Разархивировать его в укромное место, где файлы никто не тронет, и внести путь к ним в системную переменную Path (правой кнопкой по Мой компьютер->свойства->дополнительные параметры системы->переменные среды->системные переменные->добавить через точку с запятой свой каталог, например ;D:\emcproj\ctci)
  * Установить WinPCap (есть в архиве)
  * Установить Microsoft Loobback адаптер (пуск->в строке файлы и программы вводим hdwwiz->жмем Enter->далее->установка из списка вручную->далее->сетевые адаптеры->далее->Microsoft слева, а справа либо Microsoft loopback adapter либо Адаптер Microsoft замыкания на себя->далее->готово)
  * Теперь правим реестр: лезем в
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\Tcpip\Parameters``` и меняем IPEnableRouter с 0 на 1.
  * Лезем в центр управления сетями->изменение параметров адаптера->лезем в свойства microsoft loopback adapter->жмем на Протокол интернета версии 4->свойства->IP адрес 192.168.2.1 маска 255.255.255.0 шлюз 192.168.2.2 dns сервер 192.168.2.2 второй dns не надо->ок->закрыть
  * Лезем в конфигурационный файл Hercules и добавляем три строчки после PANTITLE:
```

# Channels
0E20 CTCI -n 02-00-4C-4F-4F-50 192.168.2.2 192.168.2.1
0E21 CTCI -n 02-00-4C-4F-4F-50 192.168.2.2 192.168.2.1
```
  * Перезагружаемся
  * Запускаем Hercules от имени администратора и врубаем z/OS.
  * Следим за логом когда загрузится служба TCPIP. Когда строки перестанут бежать ее->пишем в логе команду stop tcpip
  * Лезем в dataset ADCD.Z110.PROCLIB(TCPIP) и меняем в нем
```

//PROFILE  DD DISP=SHR,DSN=ADCD.Z110.TCPPARMS(PROFILE)
//*PROFILE  DD DISP=SHR,DSN=TCPIP.PROFILE.TCPIP
```
на
```

//*PROFILE  DD DISP=SHR,DSN=ADCD.Z110.TCPPARMS(PROFILE)
//PROFILE  DD DISP=SHR,DSN=TCPIP.PROFILE.TCPIP
```
  * Теперь соответственно лезем в TCPIP.PROFILE.TCPIP и комментируем в нем старые параметры "hardware definitions", "HOME", "GATEWAY", "defaultnet". Комментарий в этом датасете отмечается ;
  * Вставляем в соответствующие секции TCPIP.PROFILE.TCPIP следующие строки:
```

; Hardware definitions:
DEVICE CTCA1  CTC           E20
LINK CTC1  CTC      0 CTCA1
...
; HOME Internet (IP) addresses of each link in the host.
HOME
192.168.2.2     CTC1
...
GATEWAY
; Direct Routes - Routes that are directly connected to my interfaces.
; Network  First Hop  Link Name Packet Size  Subnet Mask  Subnet Value
192.168.2.1   =         CTC1    1492     HOST
...
; Indirect Routes - Routes that are reachable through routers on my
;                   network.
; Network  First Hop  Link Name Packet Size  Subnet Mask  Subnet Value
defaultnet  192.168.2.1   CTC1   1492 0
...
; Start all the defined devices.
; NOTE: To use these START statements, update the device name
; to reflect your installation configuration and remove the semicolon
START CTCA1
```
  * Сохраняем изменения
  * Лезем в TCPIP.FTP.DATA и меняем настройки на:
```

...
BLOCKSIZE     80
...
LRECL         80
...
RECFM         FB
```
  * Сохраняем изменения
  * Выключаем z/OS и Hercules.
  * Вырубаем брэндмауэр
  * Ставим FileZilla
  * ???
  * PROFIT!!!


Теперь система готова к использованию. Сеть будет работать только если Hercules запущен от имени администратора и подсистема TCPIP нашей z/OS загрузилась (а загружается она в последнюю очередь). Параметры FTP сервера: 192.168.2.2:21 логин ibmuser пароль sys1. Чтобы получить доступ к нужной библиотеке в Filezilla вместо 'IBMUSER.' надо ввести 'LIBNAME.' где LIBNAME - имя вашей библиотеки или начало ее имени и нажать Enter.