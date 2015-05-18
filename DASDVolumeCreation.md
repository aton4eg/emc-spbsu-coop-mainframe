## Создание DASD-тома ##
Для создания пустого тома воспользуемся программой dasdinit. Формат команды следующий:
```
dasdinit [-options] filename devtype[-model] volser [size]```
Параметры:
  * options - опции сжатия, указывать необязательно
  * filename - путь к файлу, в котором будет создан образ тома
  * devtype - тип тома
  * model - модель тома (определяет размер), указывать необязательно
  * volser - volume serial number (1-6 символов)
  * size - размер тома в цилиндрах (для CKD divices) или в секторах (для FBA devices), необходимо указывать, если не указали модель
Пример создания тома:
<br>
<img src='http://wiki.emc-spbsu-coop-mainframe.googlecode.com/git/1.jpg' />
<h2>Подключение тома к Hercules</h2>
Для подключения созданного тома к Hercules пишем в hercules.cnf:<br>
<pre><code>0A9E    3390    ${DASD_PATH:=F:\EMC\main\cckd\}user11.cckd</code></pre>
<h2>Инициализация</h2>
Для использования тома необходимо его проинициализировать. Для успешной инициализации том должен находиться в состоянии "offline", для этого пишем:<br>
<pre><code>v 0A9E,offline</code></pre>
Теперь инициализация. Для этого создаем job и делаем это с помощью ICKDSF с нужными нам параметрами. Следующий пример инициализирует том для использования с SMS (Storage Managment Subsystem).<br>
<pre><code><br>
//INITVOL     JOB<br>
//RUN         EXEC  PGM=ICKDSF<br>
//SYSPRINT    DD    SYSOUT=*<br>
//SYSIN       DD    *<br>
INIT  UNIT(0A9E) NOVERIFY STORAGEGROUP -<br>
OWNERID(NEWSTGRP) VTOC(2,1,10) INDEX(2,11,15)<br>
/*<br>
</code></pre>
Этот пример проинициализирует DASD том для обычного использования.<br>
<pre><code><br>
//INITVOL     JOB<br>
//RUN         EXEC  PGM=ICKDSF<br>
//SYSPRINT    DD    SYSOUT=*<br>
//SYSIN       DD    *<br>
INIT  UNIT(0A9E) NOVERIFY DEVTYP(DASD)<br>
VTOC(2,1,10)<br>
/*<br>
</code></pre>
Затем можно вернуть том в состояние "online" и использовать.<br>
<pre><code>v 0A9E,online</code></pre></li></ul>

<h2>Если не работает V ##
Не всегда срабатывает команда
```
v 0A9E,offline```
В таких случаях можно в HERCULES просто написать detach, чтобы отсоединить том от виртуального мейнфрейма
```
detach 0A9E```
а затем присоединить его
```
attach 0A9E 3390 D:\emc\main\cckd\emcprj.cckd```
После присоединения том по умолчанию будет offline.