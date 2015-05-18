
# Задача #

Проект состоит в том, чтобы написать программу, которая составляет граф или таблицу переходов между модулями исследуемой программы, на основе динамического (runtime) анализа ее выполнения.

# Задача подробно #
Есть некая исполняемая программа P, состоящая N модулей, которые лежат в одной библиотеке. В этой библиотеке кроме программы P находятся еще некоторые программы. Из программы P известен только запускаемый модуль и параметры (PARM, DD имена).
Требуется запустить программу P, сконструировать для нее окружение, составить список вызываемых модулей, создать таблицу переходов между ними.

Было предложено три решения:
  * Испортить код исследуемой программмы, в частности команды перехода и команды SVC 6, SVC 7 SVC 8 и ловить исключения по ESTAE
  * Испортить адрес вызываемого модуля на этапе его загрузки в память, и перехватывать ESTAE рутиной возникающие ошибки адресации (bound).
  * Анализировать записи в GTF, генерируемые командой slip trap (команда, основанная на PER, записывает в GTF события исполнения определенных команд: нам надо перехода, XCTL и LINK).

Сейчас реализуется два решения:
  * порча команд перехода
  * slip trap

# Контроль версий #
Казалось бы контроль версий программ для z/OS сложен или требует платного пакета для нее, но было обнаружено, что на z/OS можно запустить FTP сервер, а затем прокинуть сетевое подключение из виртуалки в систему host. [Делается это так](FTPD.md) Таким образом, сейчас работа осуществляется так:
  1. Запустить z/OS в виртуальной машине
  1. Запустить git
  1. Загрузить код с сервера
  1. Запустить Filezilla и подключиться к z/OS
  1. С помощью Filezilla загрузить код в z/OS
  1. Сделать изменения
  1. С помощью Filezilla выгрузить код из z/OS
  1. Сделать Commit и Push

# Организация работы #
  1. Заглянуть в Issues и выбрать задачку для реализации. Чтобы посмотреть ВСЕ Issues (в т.ч. решенные) надо выбрать Search All issues и нажать Search.
  1. Создать для реализации задачи ветку
  1. Создаем себе копию программы EMCPROJ.CNTL(templt) (это "болванка" в ней создаются все структуры данных, подключен отладочный датасет, есть основные макросы, и таким образом в ней легко создать нужное окружение и отладить свою рутину) и EMCPROJ.SRC(templt) и пишем и отлаживаем в ней свой макрос или рутину.
  1. Когда стало ясно, что все работает, то копируем этот dataset member в EMCPROJ.COPY и удаляем из него сопровождающий код (т.е оставляем только код решающий задачу)
  1. Сливаемся

# Документация #
На местном WIKI движке оказалось невозможным вставить гаджет для отображения google документов. Кроме того, на нем нельзя рисовать схемки. Так что использовать мы его не будем. Работа будет осуществляться на платформе Google Документы+Draw.IO