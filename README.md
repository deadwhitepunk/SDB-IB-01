# Домашнее задание к занятию «Уязвимости и атаки на информационные системы»

### Желонкин Дмитрий


------

### Задание 1

Скачайте и установите виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/.

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя **nmap**.

Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

- Какие сетевые службы в ней разрешены?
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)
  1)Apache Tomcat Manager - Application Deployer (Authenticated) Code Execution (Metasploit) https://www.exploit-db.com/exploits/16317 на порт 8180
  2)PostgreSQL - PostgreSQL 8.3.6 - Conversion Encoding Remote Denial of Service https://www.exploit-db.com/exploits/32849 на порт 5432
  3) ftp - vsftpd 2.3.4 - Backdoor Command Execution https://www.exploit-db.com/exploits/49757 на порт 21

Вывод NMAP:
[nmap](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/nmap)

Сканирование NMAP:

Видим открытые порты 
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1

Мы будем атаковать nfs,ftp,http

1. NFS

![Показываем showmount](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/nfs/image_nfs_showmount.png)

Следуя из вывода можем определить что корневая директория жертвы замаунчена для всех.
Маунтим корневую директорию жертвы к себе в директорию /mnt.

![Маунтим корневую директорию жертвы](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/nfs/image_nfs_mnt_victim.png)

Далее переходим в директорию /mnt и как можем заметить в ней находится весь "корень" жертвы. Далее потенциальный злоумышленник может повысить права, редактировать конфиги, собирать информацию об атакуемой машине.

![Примаунченная директория атакуемой машины](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/nfs/image_nfs_success.png)

2. FTP
Основная уязвимость в этом протоколе на атакуемой машине это "anonymous login", это позволяет подключиться любой машине по ftp используя логин и пароль anonymous/anonymous и получить доступ к файлам которые "шарятся" по протоколу FTP на данной машине.

![Примаунченная директория атакуемой машины](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/ftp/image_ftp_anonymous_logon.png)

К сожалению директория FTP оказалась пуста, но это позволяет злоумышленнику отправить свои "зараженные файлы" или в другом случае если данная папка не пуста получить доступ к конфиденициальной информации.

3. HTTP (Apache Tomcat)
Эксплуатируемая уязвимость Apache Tomcat Manager - Application Deployer (Authenticated) Code Execution (Metasploit) https://www.exploit-db.com/exploits/16317

С помощью эксплойта в метасплойте находим логин и пароль от админки tomcat 

![Находим логин и пароль от админки Tomcat](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/http/image_http_tomcat_login.png)
![Находим логин и пароль от админки Tomcat](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/http/image_http_find_login.png)

Логин и пароль найден! 
Login:  tomcat
Password: tomcat


Далее с помощью эксплойта exploit/multi/http/tomcat_mgr_deploy  Apache Tomcat Manager Application Deployer Authenticated Code Execution производим атаку, цель которой является открытие сессии meterpreter на атакуемой машине и получения доступа к терминалу жертвы

1. Вводим параметры
2. Запускаем эксплойт.
3. Проверяем сессию meterpreter

![Процесс эксплойта](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/img/http/image_http_exploit.png)


### Задание 2

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

- Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
- Как отвечает сервер?

*Приведите ответ в свободной форме.*

Данные режими отличаются:
1) Флагами:
SYN - Флаг SYN
FIN - Флаг FIN
Xmas - FIN + PSH + URG
UDP - другой протокол.
2) Тип трафика:
Syn - завершение tcp handshake не происходит, после SYN прилетает RST
FIN - отправляет FIN без SYN. FIN и сразу в ответ RST, ACK
Xmas - отправляет tcp с 3мя флагами, прилетает RST
UDP - самое долгое сканирование. Отправляет UDP дейтаграмму, в ответ прилетает ответ от ICMP port

Как отвечает Сервер?
1) SYN - SYN/ACK если порт открыт. RST если порт закрыт. Filtered - Нет ответа.
2) FIN, XMAS - Открыт - нет ответа. Закрыт - RST. Filtered - Нет ответа.
3) UDP - Открыт - UDP response. Закрыт - ICMP - Destination unreachable. Filtered - Нет ответа. 

SYN - сканирование:
[SYN](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/wireshark/syn_scan.pcapng)
FIN - сканирование:
[FIN](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/wireshark/fin_scan.pcapng)
XMAS - сканирование:
[XMAS](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/wireshark/xmas_scan.pcapng)
UDP - сканирование:
[UDP](https://github.com/deadwhitepunk/sdb-ib-01/blob/main/wireshark/udp_scan.pcapng)