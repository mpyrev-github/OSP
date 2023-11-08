# 🧑💻 3      Linux

## 3.1         Практическое занятие. Права доступа. Атрибуты файла. POSIX ACL

### Управление доступом с помощью классических битов защиты

При листинге директорий с помощью команды **ls -l** будет получена информация о дискреционном разграничении доступа к файлам.

Например, следующий вывод:

```bash
mpyrev@debian-kb:~$ ls -l
total 44
drwxr-xr-x 3 mpyrev mpyrev 4096 Mar 10 18:29 Desktop
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Documents
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Downloads
-rwxr--r-x 1 root   root     10 Mar  7 19:37 mk123.sh
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Music
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Pictures
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Public
-rwxr-xr-x 1 root   root     79 Mar  7 13:09 script.sh
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Templates
drwx-w-rwx 2 user1  user1  4096 Mar  7 18:59 test
drwxr-xr-x 2 mpyrev mpyrev 4096 Mar  7 10:28 Videos
```

В левой части вывода напротив каждого вывода представляется строка вида:

<pre class="language-bash"><code class="lang-bash"><strong>PERMIT        LINKS    USER    GROUP
</strong><strong>drwxrwxrwx    1        root    root   
</strong></code></pre>

В поле <mark style="color:green;">PERMIT</mark> указывается информация о типе файла и разрешения разделенные по логике user:group:others или u:g:o. В поле <mark style="color:green;">USER</mark> указывается владелец файла, в поле <mark style="color:green;">GROUP</mark> группа владельцев файла. В обозначение others входят все остальные пользователи, которые не являются владельцем или членом группы владельцев.

В unix-системах существуют семь типов файлов:

* `-` — обычный файл (regular file)
* `d` — каталог (directory)
* `p` — именованный канал (named pipe)
* `l` — символическая ссылка (soft link)
* `c` и `b` — символьные и блочные файлы устройств (device file)
* `s` — сокет (socket)
* `D` — дверь (door)

Биты разрешений могут принимать значения <mark style="color:green;">r w x</mark>, что означает:

* <mark style="color:red;">r</mark>ead — чтение
* <mark style="color:red;">w</mark>rite — запись&#x20;
* e<mark style="color:red;">x</mark>ecute — выполнение

### Изменение классических битов и смена владельца и группы владельцев файла

#### Изменение классических битов

При необходимости, возможно изменить стандартно назначаемые биты защиты, путем ввода команды:

```bash
chmod o+rwx mk123.sh
```

Данная команда может быть выполнена суперпользователем, администратором с использованием sudo или владельцем файла.

Синтаксис команды:

```bash
chmod [u|g|o][+|-][rwxst] [FILE]
```

В данном синтаксисе представлены биты s и t, которые рассмотрены ниже.

Кроме того, биты могут быть записаны в восьмеричном значении:

* 1 - x
* 2 - w
* 4 - r

Таким образом, при необходимости определить все права на файл (например, полные права пользователю, чтение и исполнение группе и только исполнение всем остальным), можно записать в следующем виде (по октетам):

```bash
chmod 731 mk123.sh
```

#### Смена владельца и группы владельцев файла

Если требуется сменить владельца файла или группы владельцев необходимо воспользоваться следующей командой:

```bash
chown mpyrev:root mk123.sh
```

Синтаксис команды:

```bash
chown [u]:[g] [FILE]
```

{% hint style="warning" %}
Данная команда может быть выполнена только суперпользователем или администратором с использованием sudo
{% endhint %}

### Специальные разрешения файлов в Linux (SetUID, SetGID, Sticky bit)

#### SetUID, SetGID

SetUID – это бит разрешения, который позволяет пользователю запускать исполняемый файл с правами владельца этого файла. Другими словами, использование этого бита позволяет нам поднять привилегии пользователя в случае, если это необходимо.

Принцип работы SetGID очень похож на setuid с отличием, что файл будет запускаться пользователем от имени группы, которая владеет файлом.

<pre class="language-bash"><code class="lang-bash">chmod g+s mk123.sh
ls -l mk123.sh
-rwxrw<a data-footnote-ref href="#user-content-fn-1">s</a>r-x 1 root   root     10 Mar  7 19:37 mk123.sh
</code></pre>

В случае, если бит на выполнение был удален, бит SetUID или SetGID становится заглавным.

<pre class="language-bash"><code class="lang-bash">chmod g-x mk123.sh
ls -l mk123.sh
-rwxrw<a data-footnote-ref href="#user-content-fn-2">S</a>r-x 1 root   root     10 Mar  7 19:37 mk123.sh
</code></pre>

SetUID и SetGID биты действуют только на бинарные файлы, на любых других скриптах игнорируются.

{% hint style="danger" %}
Необходимо с осторожностью назначать SetUID и SetGID биты, так как утилиты могут иметь функции по исполнению команд.&#x20;

Например, с помощью команды:
{% endhint %}

```bash
find . -exec /bin/sh -p ; -quit
```

{% hint style="danger" %}
При наличии на бинарном файле /usr/bin/find SUID бита, команда выведет консоль от имени суперпользователя.
{% endhint %}

#### Sticky bit

В случае, если этот бит установлен для папки, то файлы в этой папке могут быть удалены только их владельцем. Пример использования этого бита в операционной системе это системная папка _/tmp_ . Эта папка разрешена на запись любому пользователю, но удалять файлы в ней могут только пользователи, являющиеся владельцами этих файлов.

<pre class="language-bash"><code class="lang-bash">ls -ld /tmp
drwxrwxrw<a data-footnote-ref href="#user-content-fn-3">t</a> 8 root root 4096 Mar 25 10:22 /tmp
</code></pre>

При отсутствии бита на исполнение, sticky bit отображается заглавной буквой:

<pre class="language-bash"><code class="lang-bash">chmod o-x /tmp
ls -ld /tmp
drwxrwxrw<a data-footnote-ref href="#user-content-fn-4">T</a> 8 root root 4096 Mar 25 10:22 /tmp
</code></pre>

### &#x20;Атрибуты файла

Просмотреть все атрибуты файла возможно командой:

```bash
mpyrev@debian-kb:~$ lsattr
--------------e------- ./Templates
--------------e------- ./Desktop
--------------e------- ./mk123.sh
--------------e------- ./Documents
--------------e------- ./Music
--------------e------- ./Pictures
--------------e------- ./script.sh
--------------e------- ./Public
--------------e------- ./test
--------------e------- ./Downloads
--------------e------- ./Videos
```

В случае необходимости создать неизменяемый файл, возможно добавить атрибут +i необходимому файлу:

```bash
mpyrev@debian-kb:~$ sudo chattr +i mk123.sh 
mpyrev@debian-kb:~$ su
Password: 
root@debian-kb:/home/mpyrev# rm mk123.sh 
rm: cannot remove 'mk123.sh': Operation not permitted
root@debian-kb:/home/mpyrev# lsattr mk123.sh 
----i---------e------- mk123.sh
```

{% hint style="warning" %}
Данная команда может быть выполнена только суперпользователем или администратором с использованием sudo
{% endhint %}

{% hint style="info" %}
При назначении данного атрибута, даже суперпользователь не в праве изменить данный файл
{% endhint %}

Если потребуется создать файл, который является лог-файлом (нельзя изменять, но возможно дописывать), то необходимо добавить атрибут +a

```bash
pyrev@debian-kb:~$ cd test/
mpyrev@debian-kb:~/test$ ls
1
mpyrev@debian-kb:~/test$ cat 1
test
test
mpyrev@debian-kb:~/test$ ls -la
total 12
drwx-w-rwx  2 user1  user1  4096 Mar  7 18:59 .
drwxr-xr-x 16 mpyrev mpyrev 4096 Apr  3 16:37 ..
-rw-r--r--  1 mpyrev mpyrev   10 Mar  7 20:22 1
mpyrev@debian-kb:~/test$ chattr +a 1
mpyrev@debian-kb:~/test$ echo "hello" > 1
bash: 1: Operation not permitted
mpyrev@debian-kb:~/test$ echo "hello" >> 1
mpyrev@debian-kb:~/test$ cat 1
test
test
hello
mpyrev@debian-kb:~/test$ lsattr 1
-----a--------e------- 1
```

### POSIX ACL

ACL (Access control lists) — более сложный, но гибкий инструмент управления правами доступа. Он позволяет сегментировать категорию «other» и разрешить работу с файлом/директорией нескольким конкретным пользователям и группам.

Проверить ACL возможно командой:

```bash
mpyrev@debian-kb:~$ getfacl script.sh 
# file: script.sh
# owner: root
# group: root
# flags: s--
user::rwx
group::r-x
other::r-x
```

Для того, чтобы добавить определенным пользователям права на файл, возможно модифицировать ACL:

```bash
mpyrev@debian-kb:~$ sudo setfacl -m u:mpyrev:rwx,g:user1:r script.sh 
mpyrev@debian-kb:~$ getfacl script.sh 
# file: script.sh
# owner: root
# group: root
# flags: s--
user::rwx
user:mpyrev:rwx
group::r-x
group:user1:r--
mask::rwx
other::r-x
```

В данном случае, помимо добавленных разрешений появилась маска. По умолчанию, составляется как дизъюнкция всех разрешений, кроме владельца. Данная маска необходима для определения максимально возможных прав доступа отличных от владельца пользователям. Например, при маске `-wx`, будет отображено следующее:

```bash
mpyrev@debian-kb:~$ sudo setfacl -m mask::wx script.sh 
mpyrev@debian-kb:~$ getfacl script.sh 
# file: script.sh
# owner: root
# group: root
# flags: s--
user::rwx
user:mpyrev:rwx			#effective:-wx
group::r-x			#effective:--x
group:user1:r--			#effective:---
mask::-wx
other::r-x
```

Кроме того, возможно создать шаблон на папку для автоматического распространения ACL при добавлении содержимого (наследственность):

```bash
mpyrev@debian-kb:~$ mkdir default
mpyrev@debian-kb:~$ getfacl default/
# file: default/
# owner: mpyrev
# group: mpyrev
user::rwx
group::r-x
other::r-x

mpyrev@debian-kb:~$ sudo setfacl -m d:u:mpyrev:rwx,d:g:user1:rx default/
mpyrev@debian-kb:~$ getfacl default/
# file: default/
# owner: mpyrev
# group: mpyrev
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:mpyrev:rwx
default:group::r-x
default:group:user1:r-x
default:mask::rwx
default:other::r-x

mpyrev@debian-kb:~$ touch default/file
mpyrev@debian-kb:~$ getfacl default/file 
# file: default/file
# owner: mpyrev
# group: mpyrev
user::rw-
user:mpyrev:rwx			#effective:rw-
group::r-x			#effective:r--
group:user1:r-x			#effective:r--
mask::rw-
other::r--
```

Если необходимо удалить какое-либо разрешение, то потребуется использовать ключ -x для построчного удаления или -b для общего удаления всех ACL:

```bash
mpyrev@debian-kb:~$ sudo setfacl -x u:mpyrev default/file 
mpyrev@debian-kb:~$ getfacl default/file 
# file: default/file
# owner: mpyrev
# group: mpyrev
user::rw-
group::r-x
group:user1:r-x
mask::r-x
other::r--

mpyrev@debian-kb:~$ sudo setfacl -b default/file 
mpyrev@debian-kb:~$ getfacl default/file 
# file: default/file
# owner: mpyrev
# group: mpyrev
user::rw-
group::r-x
other::r--
```

## 3.2         Практическое занятие. Настройка OpenLDAP KDC

### Предварительная настройка разрешения доменных имен

Для практического занятия потребуются две виртуальные машины (ВМ) Debian.\
Далее будет использоваться Debian 11 _"bullseye"_. Однако одна ВМ будет настроена без графической оборочки, далее - Debian Server. ВМ с графической оболочкой - Debian Client.

При установке рекомендуется сразу указать домен, к примеру - [`kb.edu`](#user-content-fn-5)[^5].

Таким образом hostname ВМ будут выглядеть следующим образом: `server.kb.edu` и `client.kb.edu`.&#x20;

В файл `/etc/hosts` на `server.kb.edu` необходимо добавить информацию об IP-адресах клиента и сервера. В реальной инфраструктуре данная информация вносится в DNS-сервер.

```
root@server:~# cat /etc/hosts
127.0.0.1       localhost
#127.0.1.1      server.kb.edu   server
192.168.3.139   server.kb.edu   server
192.168.3.140   client.kb.edu   client

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

{% hint style="danger" %}
В файл `/etc/hosts` на `client.kb.edu` также необходимо добавить информацию об IP-адресах клиента и сервера.
{% endhint %}

Также рекомендуется проверить наличие ping при запросе по имени:

```
root@client:~# ping server
PING server.kb.edu (192.168.3.139) 56(84) bytes of data.
64 bytes from server.kb.edu (192.168.3.139): icmp_seq=1 ttl=64 time=0.449 ms
64 bytes from server.kb.edu (192.168.3.139): icmp_seq=2 ttl=64 time=0.269 ms
64 bytes from server.kb.edu (192.168.3.139): icmp_seq=3 ttl=64 time=0.978 ms
64 bytes from server.kb.edu (192.168.3.139): icmp_seq=4 ttl=64 time=1.20 ms
^C
--- server.kb.edu ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3094ms
rtt min/avg/max/mdev = 0.269/0.723/1.199/0.378 ms
root@client:~# ping client
PING client.kb.edu (192.168.3.140) 56(84) bytes of data.
64 bytes from client.kb.edu (192.168.3.140): icmp_seq=1 ttl=64 time=0.037 ms
64 bytes from client.kb.edu (192.168.3.140): icmp_seq=2 ttl=64 time=0.063 ms
64 bytes from client.kb.edu (192.168.3.140): icmp_seq=3 ttl=64 time=0.085 ms
64 bytes from client.kb.edu (192.168.3.140): icmp_seq=4 ttl=64 time=0.047 ms
^C
--- client.kb.edu ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3072ms
rtt min/avg/max/mdev = 0.037/0.058/0.085/0.018 ms

```

### Установка OpenLDAP

Где работаем: **server**

Мы сразу устанавливаем **krb5-kdc-ldap**, даже несмотря на то, что настроим их позже. Идея заключается в том, чтобы сразу получить необходимые схемы наборов данных OpenLDAP.

<pre class="language-bash"><code class="lang-bash"><strong>root@server:~# apt-get install slapd ldap-utils krb5-kdc-ldap
</strong></code></pre>

Во время установки Вам будет предложено задать некоторые настройки:

* пароль учётной записи **admin** для доступа к конфигурации OpenLDAP;
* название области Kerberos (realm);
* сервер Kerberos для нашей области;
* управляющий сервер области.

На данном этапе эти настройки не важны. Мы последовательно опишем их в дальнейшем.

Прежде чем продолжить, убедитесь, что нужные пакеты установились:

<pre class="language-bash"><code class="lang-bash"><strong>root@server:~# dpkg --get-selections|egrep '(slapd|ldap-utils|krb5-kdc-ldap)'
</strong>krb5-kdc-ldap                                   install
ldap-utils                                      install
slapd                                           install
</code></pre>

### Инициализация конфигурации каталога

Где работаем: **server**

Инициализацию конфигурации каталога мы произведём с нуля с использованием нового подхода, OLC (`cn=config`).

Создайте каталог для работы со службой каталогов. У нас будет много конфигурационных файлов, неплохо бы класть их в одно место:

<pre class="language-bash"><code class="lang-bash"><strong>root@server:~# mkdir ~/ldap
</strong>root@server:~# cd ~/ldap
</code></pre>

Избавимся от установленных по умолчанию конфигурации **slapd** и его базы данных:

```bash
root@server:~# service slapd stop
root@server:~# rm -rf /etc/ldap/slapd.d
root@server:~# rm -rf /var/lib/ldap
```

Создадим пустой каталог для нашей новой конфигурации:

```bash
root@server:~# mkdir /etc/ldap/slapd.d
```

Создадим файл `init-config.ldif` с новой конфигурацией и запишем в него:

{% hint style="warning" %}
Здесь и далее рекомендуется писать конфигурацию вручную или производить копирование c заменой знаков табуляции вначале каждой строки, если такие имеются
{% endhint %}

```
dn: cn=config
objectClass: olcGlobal
cn: config
olcPidFile: /var/run/slapd/slapd.pid
olcArgsFile: /var/run/slapd/slapd.args

dn: olcDatabase={0}config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {0}config
olcAccess: to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage

dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

```

В этом файле первым делом мы определяем корневую запись DIT (Directory Information Tree), `cn=config`. С помощью директив `olcPidFile` и `olcArgsFile` мы указали, куда необходимо записать ID процесса службы каталогов и аргументы его запуска соответственно.

Во втором разделе задаём служебную базу данных конфигурации `cn=config`. Мы так же добавили правило доступа (ACL, Access Control List), разрешающее манипулировать ей от имени пользователя **root** (uid=0, gid=0) с помощью механизма [SASL EXTERNAL](http://pro-ldap.ru/tr/admin24/sasl.html#EXTERNAL) и идентификационной сущности IPC. Помните, что в конце каждого ACL, если не задан модификатор `break`, подразумевается наличие правила `by * none` . То есть остальной доступ к объекту в условии `to` — запрещён.

Последним разделом мы добавляем в конфигурацию контейнер для наборов схем данных.

Инициализируем конфигурацию:

```bash
root@server:~# slapadd -n 0 -F /etc/ldap/slapd.d -l init-config.ldif
_#################### 100.00% eta   none elapsed            none fast!
Closing DB...
```

Модификатор `-n 0` говорит о том, что мы добавляем данные в базу данных с индексом 0, который зарезервирован для `cn=config`.

Проверим, всё ли в порядке с нашей конфигурацией:

```bash
root@server:~# slaptest -uF /etc/ldap/slapd.d
config file testing succeeded
```

Поправим права доступа, разрешив пользователю **openldap** заправлять в каталоге `/etc/ldap/slapd.d`:

```bash
root@server:~# chown -R openldap:openldap /etc/ldap/slapd.d
root@server:~# chmod 750 /etc/ldap/slapd.d
```

### Запуск службы каталогов

Где работаем: **server**

Разрешим нашей службе каталогов использовать только IPv4. Для этого установим  `SLAPD_OPTIONS="-4"` в файле `/etc/default/slapd`. В остальном конфигурация стандартная:

<pre><code>SLAPD_CONF=
SLAPD_USER="openldap"
SLAPD_GROUP="openldap"
SLAPD_PIDFILE=
SLAPD_SERVICES="ldap:/// ldapi:///"
SLAPD_SENTINEL_FILE=/etc/ldap/noslapd
<strong>SLAPD_OPTIONS="-4"
</strong><strong>
</strong></code></pre>

Настроим **rsyslog**, чтобы он писал события службы каталогов в отдельный файл. Для этого достаточно добавить три строки в его конфигурацию после глобальных директив `/etc/rsyslog.conf`:

<pre><code>$ModLoad imuxsock # provides support for local system logging
$ModLoad imklog   # provides kernel logging support

$KLogPermitNonKernelFacility on

$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

$RepeatedMsgReduction on

$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

$WorkDirectory /var/spool/rsyslog

$IncludeConfig /etc/rsyslog.d/*.conf

<strong># Пишем журнал демона slapd в /var/log/slapd.log
</strong><strong>if $programname == 'slapd' then /var/log/slapd.log
</strong><strong>&#x26; ~
</strong><strong>
</strong></code></pre>

Создадим файл для журнала службы каталогов и зададим для него права доступа. Затем перезапустим **rsyslog**, чтобы изменения вступили в силу:

```bash
root@server:~# touch /var/log/slapd.log
root@server:~# chmod 0640 /var/log/slapd.log
root@server:~# chown root:adm /var/log/slapd.log
root@server:~# service rsyslog restart
```

Настроим **logrotate** для управления этим журналом. Создадим файл конфигурации `/etc/logrotate.d/slapd` и запишем в него:

```
/var/log/slapd.log {
 daily
 missingok
 rotate 7
 compress
 delaycompress
 notifempty
}

```

Проверим настройку **logrotate**:

```bash
root@server:~# logrotate -df /etc/logrotate.conf
```

Наконец запускаем нашу службу каталогов:

```bash
root@server:~# service slapd start
```

Заглянем в файл журнала `slapd.log`. Всё ли в порядке?

```bash
root@server:~# tail /var/log/slapd.log
...
slapd starting
```

Проверим, активен ли TCP порт 389:

```bash
root@server:~# ss -tan | grep 389
LISTEN 0      1024         0.0.0.0:389       0.0.0.0:*
```

Убедимся, что UNIX сокет тоже активен:

```bash
root@server:~# ss -xa | grep ldap
u_str LISTEN 0      1024                         /var/run/slapd/ldapi 32943            * 0
```

Для пущей убедительности проверим текущую конфигурацию каталога с помощью **ldapsearch**:

```bash
root@server:~# ldapsearch -QLLL -Y EXTERNAL -H ldapi:/// -b 'cn=config' dn
dn: cn=config
dn: cn=schema,cn=config
dn: olcDatabase={-1}frontend,cn=config
dn: olcDatabase={0}config,cn=config

```

Заметьте, что **slapadd** (из предыдущего пункта) добавил в наш каталог описание служебной базы данных `frontend` (в ней можно определить опции, которые будут применяться ко всем базам данных в текущей конфигурации OLC). Однако, если те же самые опции (в том числе правила ACL) определены в конкретной базе данных, то будут применяться именно они.

### Подключение динамических модулей

Где работаем: **server**

Для работы нам понадобится два модуля. Один — для механизма базы данных **mdb**. На данный момент он рассматривается как основной для нормальных баз данных и [должен прийти на смену](http://pro-ldap.ru/tr/art/20130702-mdb/#r2) **bdb** и **hdb**. Второй модуль — **monitor**, для создания и динамической поддержки ветки с информацией о текущем статусе демона **slapd**.

Чтобы их подключить нам понадобится всего один короткий LDIF-файл и специальная запись `cn=module,cn=config`. Назовём файл `add-modules.ldif` и запишем в него:

```
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib/ldap
olcModuleLoad: back_mdb.la
olcModuleLoad: back_monitor.la

```

Каталог с модулями в нашем случае — `/usr/lib/ldap`.

Добавим наш LDIF-файл в конфигурацию:

<pre class="language-bash"><code class="lang-bash"><strong>root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f add-modules.ldif
</strong>adding new entry "cn=module,cn=config"
</code></pre>

### Добавление наборов схем данных

Где работаем: **server**

Прежде чем добавлять наборы схем данных в каталог, необходимо подготовить наборы схем из пакета **krb5-kdc-ldap** к загрузке (перевести их в формат LDIF).

В используемой нами версии Debian искомые наборы схем находятся по пути `/usr/share/doc/krb5-kdc-ldap/kerberos.schema.gz`

Скопируем их во временный каталог:

```bash
root@server:~# zcat /usr/share/doc/krb5-kdc-ldap/kerberos.schema.gz > ~/ldap/kerberos.schema
```

[На просторах сети](http://stuckinadoloop.wordpress.com/2011/04/14/script-to-convert-openldap-schema-files-to-ldif-format/) был найден несложный скрипт для преобразования файлов `schema` в формат LDIF. Немного откорректируем его (для универсальности) и сохраним в том же каталоге `~/ldap` под именем `schema-ldif.sh`:

<pre class="language-bash"><code class="lang-bash">#!/bin/bash

<strong>SCHEMAD=~/ldap
</strong>
tmpd=`mktemp -d`
pushd ${tmpd} >>/dev/null

<strong>echo '' > convert.dat
</strong>
for schema in ${SCHEMAS} ; do
    echo include ${SCHEMAD}/${schema} >> convert.dat
done

slaptest -f convert.dat -F .

if [ $? -ne 0 ] ; then
    echo "slaptest conversion failed"
    exit 
fi

for schema in ${SCHEMAS} ; do
    fullpath=${SCHEMAD}/${schema}
    schema_name=`basename ${fullpath} .schema`
    schema_dir=`dirname ${fullpath}`
    ldif_file=${schema_name}.ldif

    find . -name *${schema_name}.ldif -exec mv '{}' ./${ldif_file} \;
    sed -i "/dn:/ c dn: cn=${schema_name},cn=schema,cn=config" ${ldif_file}
    sed -i "/cn:/ c cn: ${schema_name}" ${ldif_file}
    sed -i '/structuralObjectClass/ d' ${ldif_file}
    sed -i '/entryUUID/ d' ${ldif_file}
    sed -i '/creatorsName/ d' ${ldif_file}
    sed -i '/createTimestamp/ d' ${ldif_file}
    sed -i '/entryCSN/ d' ${ldif_file}
    sed -i '/modifiersName/ d' ${ldif_file}
    sed -i '/modifyTimestamp/ d' ${ldif_file}
    sed -i '/^ *$/d' ${ldif_file}

    mv ${ldif_file} ${schema_dir}
done

popd >>/dev/null
rm -rf $tmpd

</code></pre>

Сделаем его исполняемым и запустим, передав через переменную `SCHEMAS` имя файла конвертируемого набора схем:

```bash
root@server:~# chmod +x schema-ldif.sh
root@server:~# SCHEMAS='kerberos.schema' ./schema-ldif.sh
config file testing succeeded
```

На выходе должны получить набор схем данных в формате LDIF:

```bash
root@server:~# ls ~/ldap | egrep 'kerberos.ldif'
kerberos.ldif
```

Переместим его в каталог к остальным наборам схем и поправим права доступа:

```bash
root@server:~# mv ~/ldap/kerberos.ldif /etc/ldap/schema
root@server:~# chown root:root /etc/ldap/schema/kerberos.ldif
root@server:~# chmod 0644 /etc/ldap/schema/kerberos.ldif
```

Удалим ненужный больше набор схем данных:

```bash
root@server:~# rm kerberos.schema
```

Создадим LDIF-файл с необходимыми нам наборами схем данных.&#x20;

{% hint style="warning" %}
Порядок их следования в файле очень важен
{% endhint %}

Атрибуты наборов схем иерархически связаны и требуют их объявления с соблюдением иерархии. Назовём файл `add-schemas.ldif` и запишем в него:

```
include: file:///etc/ldap/schema/core.ldif
include: file:///etc/ldap/schema/cosine.ldif
include: file:///etc/ldap/schema/inetorgperson.ldif
include: file:///etc/ldap/schema/collective.ldif
include: file:///etc/ldap/schema/corba.ldif
include: file:///etc/ldap/schema/duaconf.ldif
include: file:///etc/ldap/schema/openldap.ldif
include: file:///etc/ldap/schema/dyngroup.ldif
include: file:///etc/ldap/schema/java.ldif
include: file:///etc/ldap/schema/misc.ldif
include: file:///etc/ldap/schema/nis.ldif
include: file:///etc/ldap/schema/ppolicy.ldif
include: file:///etc/ldap/schema/kerberos.ldif

```

Последней строчкой мы указали получившийся в результате конвертации набор схем `kerberos.ldif`.

Добавим наши наборы схем:

```
root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f add-schemas.ldif
adding new entry "cn=core,cn=schema,cn=config"
adding new entry "cn=cosine,cn=schema,cn=config"
adding new entry "cn=inetorgperson,cn=schema,cn=config"
adding new entry "cn=collective,cn=schema,cn=config"
adding new entry "cn=corba,cn=schema,cn=config"
adding new entry "cn=duaconf,cn=schema,cn=config"
adding new entry "cn=openldap,cn=schema,cn=config"
adding new entry "cn=dyngroup,cn=schema,cn=config"
adding new entry "cn=java,cn=schema,cn=config"
adding new entry "cn=misc,cn=schema,cn=config"
adding new entry "cn=nis,cn=schema,cn=config"
adding new entry "cn=ppolicy,cn=schema,cn=config"
adding new entry "cn=kerberos,cn=schema,cn=config"
```

### Инициализация базы данных

Где работаем: **server**

Вот мы и подошли к созданию базы данных, в которой будем хранить нашу рабочую информацию.

Для начала создадим пароль администратора. Утилита **slappasswd** генерирует посоленный хэш вводимого нами пароля:

```bash
root@server:~# slappasswd -h '{SSHA}'
New password:
Re-enter new password:
{SSHA}RMjrfsi7NkpBMR+NQHTDk4iddvo/2le+
```

{% hint style="info" %}
Не забудьте сам пароль :blush:
{% endhint %}

Сформируем конфигурационный LDIF-файл для нашей базы данных `db.ldif` и запишем получившийся хэш в атрибут `olcRootPW`:

```
dn: olcDatabase=mdb,cn=config
objectClass: olcMdbConfig
olcDatabase: mdb
olcSuffix: dc=kb,dc=edu
olcDbDirectory: /var/lib/ldap
olcDbMaxsize: 1073741824
olcRootDN: cn=admin,dc=kb,dc=edu
olcRootPW: {SSHA}RMjrfsi7NkpBMR+NQHTDk4iddvo/2le+
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage
  by * break
olcAccess: {1}to attrs=userPassword
  by self write
  by anonymous auth
olcAccess: {2}to *
  by self read

dn: olcDatabase=monitor,cn=config
objectClass: olcDatabaseConfig
olcDatabase: monitor

```

И вновь несколько комментариев...

* В качестве механизма манипуляции данными выбираем ранее подключенный модуль **mdb**.
* В качестве суффикса (корневой записи) создаваемого DIT (Directory Information Tree) обычно используется имя домена DNS. Но это не является обязательным. Для примера мы задали суффикс `dc=kb,dc=edu`.
* В атрибуте `olcDbDirectory` мы указали путь к каталогу бузы данных. Атрибут обязательный и требует существование каталога на момент загрузки в базу данных.
* Механизм **mdb** требует указания максимального размера базы данных. Он задается в байтах и должен быть больше её ожидаемого размера, даже с учётом прироста. В файловой системе должно быть достаточно свободного места для размещения базы данных такого размера.
* В атрибут `olcRootDN` мы записываем DN администратора нашей базы данных.
* Для пользователя, указанного в атрибуте `olcRootDN`, задавать правило в ACL не нужно, он обладает полным доступом к данным в вашей базе. Поэтому постарайтесь сохранить его пароль в надежном месте (например, используя [keepass](http://keepass.info/) или [gpg](https://www.gnupg.org/)).
* Добавляем три правила доступа для базы данных **mdb**:
  * Доступ ко всей базе данных:
    * Разрешить доступ пользователю **root** с использованием механизма [SASL EXTERNAL](http://pro-ldap.ru/tr/admin24/sasl.html#EXTERNAL).
    * Продолжить анализ ACL, если нет совпадений с субъектами доступа, указанными с помощью директивы `by`.
  * Доступ к атрибуту `userPassword`:
    * Разрешить доступ для смены пароля самим пользователем.
    * Разрешить доступ для аутентификации.
  * Доступ к остальной базе данных:
    * Разрешить пользователям просматривать свои записи (важно для аутентификации через **nslcd**, [Настройка аутентификации пользователей через OpenLDAP на клиенте](3-linux.md#nastroika-autentifikacii-polzovatelei-cherez-openldap-na-kliente)).
* С помощью механизма **monitor** включаем мониторинг базы данных (добавляем базу данных `monitor`).

Для нашей базы данных необходимо создать каталог и задать ему права доступа:

```bash
root@server:~# mkdir /var/lib/ldap
root@server:~# chown openldap:openldap /var/lib/ldap
root@server:~# chmod 0700 /var/lib/ldap
```

Загрузим конфигурацию базы данных:

```bash
root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f db.ldif
adding new entry "olcDatabase=mdb,cn=config"
adding new entry "olcDatabase=monitor,cn=config"
```

Определим RootDN для доступа к конфигурации службы каталогов. Он будет ссылаться на RootDN, находящийся в нашей БД. Эта запись желательна только при первоначальной настройке или в тестовой конфигурации. Не оставляйте её в боевой системе! Для `{-1}frontend` зададим минимально необходимые права для доступа к [RootDSE](http://pro-ldap.ru/tr/zytrax/apd/#rootdse). Создадим LDIF-файл `acl-mod.ldif`, модифицирующий права доступа:

```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootDN
olcRootDN: cn=admin,dc=kb,dc=edu

dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to *
  by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth manage
  by * break
olcAccess: {1}to dn.base=""
  by * read
olcAccess: {2}to dn.base="cn=subschema"
  by * read

```

Загрузим LDIF, изменяющий ACL:

```bash
root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f acl-mod.ldif
modifying entry "olcDatabase={-1}frontend,cn=config"
modifying entry "olcDatabase={0}config,cn=config"
```

Теперь мы ещё больше повысили значимость учётной записи администратора. С её помощью теперь можно получить полный доступ к службе каталогов. Имейте это ввиду. :blush:

Проверим корректность всех ACL и, заодно, наличие всех добавленных нами данных (вывод отформатирован для наглядности):

```
root@server:~# ldapsearch -QLLL -Y EXTERNAL -H ldapi:/// -b 'cn=config' dn olcAccess
dn: cn=config

dn: cn=module{0},cn=config

dn: cn=schema,cn=config
dn: cn={0}core,cn=schema,cn=config
dn: cn={1}cosine,cn=schema,cn=config
dn: cn={2}inetorgperson,cn=schema,cn=config
dn: cn={3}collective,cn=schema,cn=config
dn: cn={4}corba,cn=schema,cn=config
dn: cn={5}duaconf,cn=schema,cn=config
dn: cn={6}openldap,cn=schema,cn=config
dn: cn={7}dyngroup,cn=schema,cn=config
dn: cn={8}java,cn=schema,cn=config
dn: cn={9}misc,cn=schema,cn=config
dn: cn={10}nis,cn=schema,cn=config
dn: cn={11}ppolicy,cn=schema,cn=config
dn: cn={12}kerberos,cn=schema,cn=config

dn: olcDatabase={-1}frontend,cn=config
olcAccess: {0}to *
  by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external ,cn=auth manage
  by * break
olcAccess: {1}to dn.base=""
  by * read
olcAccess: {2}to dn.base="cn=subschema"
  by * read

dn: olcDatabase={0}config,cn=config
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external ,cn=auth" manage

dn: olcDatabase={1}mdb,cn=config
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external ,cn=auth" manage
  by * break
olcAccess: {1}to attrs=userPassword
  by self write
  by anonymous auth
olcAccess: {2}to *
  by self read

dn: olcDatabase={2}monitor,cn=config
```

Убедимся, что учётная запись администратора имеет доступ к нашей службе каталогов:

```bash
root@server:~# ldapwhoami -WD cn=admin,dc=kb,dc=edu
Enter LDAP Password:
dn:cn=admin,dc=kb,dc=edu
```

Отлично! Теперь у нас есть работающий сервер OpenLDAP.

Отредактируем файл `/etc/ldap/ldap.conf`. Это нужно только для того, чтобы немного упростить себе жизнь и меньше печатать в дальнейшем. В `BASE` подставьте свой суффикс, а в `URI` — FQDN Вашего сервера OpenLDAP:

```bash
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE    dc=kb,dc=edu
URI     ldap://server.kb.edu

#SIZELIMIT      12
TIMELIMIT       15
TIMEOUT         20
#DEREF          never

# TLS certificates (needed for GnuTLS)
#TLS_CACERT     /etc/ssl/certs/ca-certificates.crt


```

С настройкой TLS мы разберемся в следующем разделе.

И последний штрих. Добавим демон нашей службы каталогов в автозагрузку:

```bash
root@server:~# update-rc.d slapd defaults
```

### Удостоверяющий центр на основе самоподписанного сертификата с помощью OpenSSL

Где работаем: **server**

Итак, мы создаём централизованную инфраструктуру открытых ключей (PKI) в миниатюре. Для этого сформируем пару закрытый ключ/корневой сертификат удостоверяющего центра. Причём сертификат будет подписан этим же закрытым ключом (т. е. станет самоподписанным).

Затем сгенерируем новый закрытый ключ для нашей службы каталогов и создадим для неё сертификат, подписанный закрытым ключом нашего удостоверяющего центра. При этом клиенские рабочие станции должны знать только корневой сертификат. Используя его они могут установить безопасное соединение с любым сервером, который имеет закрытый ключ и соответствующий ему сертификат, подписанный нашим УЦ.

Удостоверяющий центр желательно разворачивать на отдельной машине, не имеющей подключения к сети, но для простоты примера мы проделаем всю работу на **server.kb.edu**.

Создадим каталог для нашего удостоверяющего центра (CA) и установим безопасные права доступа. Затем зададим значение `umask` таким, чтобы вновь создаваемые файлы имели права доступа чтения и записи только для создавшего их пользователя:

```bash
root@server:~# mkdir /root/CA
root@server:~# chmod u=rwx,g=,o= /root/CA
root@server:~# cd /root/CA
root@server:~# umask 066
```

В конфигурационном файле `/etc/ssl/openssl.cnf` в секции `[ CA_default ]` для удобства поменяем значение директивы `dir = ./demoCA` на `dir = ./`. В этом же разделе директивой `default_days` задаётся срок действия выпускаемых сертификатов. Можете поменять её значение по своему усмотрению.

Файлы `index.txt` и `serial` будут играть роль своеобразной базы данных, чтобы отслеживать статус выпущенных закрытых ключей и сертификатов.

Создадим структуру каталогов и файлов для своего удостоверяющего центра:

```bash
root@server:~# mkdir certs crl newcerts private
root@server:~# chmod 700 private
root@server:~# touch index.txt
root@server:~# echo 1000 > serial
```

Сгенерируем закрытый ключ удостоверяющего центра (длиной 4096 бит). Он должен хранится особенно бережно. Поэтому мы защитим его при помощи шифра [AES](https://ru.wikipedia.org/wiki/Advanced\_Encryption\_Standard). Вам будет предложено ввести пароль для доступа к новому закрытому ключу. Запомните его и не потеряйте. Это последний рубеж защиты от злоумышленника. Без пароля выпустить новые сертификаты не получится.

```bash
root@server:~# openssl genrsa -aes256 -out private/rootca.key 4096
```

Изменим права доступа к новому ключу, чтобы ненароком его не стереть:

```bash
root@server:~# chmod 400 private/rootca.key
```

Откройте конфигурационный файл OpenSSL `/etc/ssl/openssl.cnf` и найдите секции         `[ usr_cert ]` и `[ v3_ca ]`. Убедитесь, что в них присутствуют следующие директивы:

```
[ usr_cert ]
# Эти расширения будут добавлены при подписывании запроса нашим УЦ.
basicConstraints=CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
nsComment = "OpenSSL Generated Certificate"
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

[ v3_ca ]
# Расширения для типового УЦ
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = CA:true
keyUsage = cRLSign, keyCertSign
```

Сейчас мы можем выпустить корневой сертификат удостоверяющего центра, подписав его закрытым ключом `rootca.key`. Так как это сертификат УЦ, используем расширение `v3_ca`:

```bash
root@server:~# openssl req -sha256 -new -x509 -days 3650 -extensions v3_ca \
-key private/rootca.key -out certs/rootca.crt \
-subj /C=RU/ST=Sverdlovskaya/L=Ekaterinburg/O=UrFU/OU=KB/CN=ca-server/emailAddress=support@kb.edu
root@server:~# chmod 444 certs/rootca.crt
```

В качестве алгоритма хэширования мы используем [SHA-2](https://ru.wikipedia.org/wiki/SHA-2) (подвид SHA-256). Стоимость [атаки](https://www.schneier.com/blog/archives/2012/10/when\_will\_we\_se.html) на SHA-1 [стремительно падает](https://www.opennet.ru/opennews/art.shtml?num=40534), а о практическом применении новоявленного [SHA-3](https://ru.wikipedia.org/wiki/Keccak) говорить пока рано. Почему мы выбрали именно SHA-256, а не SHA-512? С сертификатом, использующим SHA-512, Вы сможете запустить демон **slapd**, но не сможете к нему подключиться. Увидите лишь такую многозначную [ошибку](https://www.calmblue.net/blog/server/linux/openldap-server-on-debian-using-olc):

```
can't connect: A TLS packet with unexpected length was received...
```

Мы так же указываем количество дней, в течении которых сертификат будет действителен. Десять лет — достаточно большой срок.

С помощью модификатора `subj` заносим в сертификат информацию:

* C - Country Name (страна) - RU
* ST - State or Province Name (штат или провинция) - Sverdlovskaya
* L - Locality Name (город) - Ekaterinburg
* O - Organization Name (наименование организации) - UrFU
* OU - Organizational Unit Name (наименование подразделения) - KB
* CN - Common Name (имя субъекта или FQDN сервера) - ca-server
* emailAddress - Email Address (адрес электронной почты) - support@kb.edu

Можем посмотреть содержимое сертификата следующей командой:

```bash
root@server:~# openssl x509 -in certs/rootca.crt -noout -text
```

Удостоверяющий центр готов к выпуску сертификатов.

### Выпуск сертификата для сервера службы каталогов

Где работаем: **server**

Как правило, закрытые ключи клиентов УЦ не должны хранится в самом УЦ. Клиент может сам сформировать ключевую пару и прислать в УЦ лишь запрос на подпись (Certificate Signing Request, CSR). Запрос содержит открытый ключ. Однако мы будем создавать все ключи и хранить их в одном месте. В организации, которая серьёзно подходит к проблемам безопасности такой процесс работы может быть неприемлемым.

Продолжаем в том же каталоге `/root/CA`. Создадим закрытый ключ для сервера службы каталогов:

```bash
root@server:~# openssl genrsa -out private/server.kb.edu.key 4096
root@server:~# chmod 400 private/server.kb.edu.key
```

Сгенерируем запрос на подпись сертификата. Наименование организации (**UrFU**) должно совпадать с наименованием в корневом сертификате УЦ. В качестве `Common Name` укажем FQDN нашего сервера:

```bash
root@server:~# openssl req -sha256 -new \
-key private/server.kb.edu.key -out certs/server.kb.edu.csr \
-subj /C=RU/ST=Sverdlovskaya/L=Ekaterinburg/O=UrFU/OU=KB/CN=server.kb.edu/emailAddress=support@kb.edu
```

Следующим шагом должно быть подписание запроса CSR существующим доверенным удостоверяющим центром (например, [VeriSign](https://www.verisigninc.com/)) в обмен на деньги. Но нам не хочется платить за эту услугу, или у нас нет своего (корпоративного) CA, или это просто тестовая конфигурация, а может нам просто всё равно. Поэтому мы подпишем его с помощью своего собственного CA:

```bash
root@server:~# openssl ca -extensions usr_cert -notext -md sha256 \
-keyfile private/rootca.key -cert certs/rootca.crt \
-in certs/server.kb.edu.csr -out certs/server.kb.edu.crt
root@server:~# chmod 444 certs/server.kb.edu.crt
```

Создадим каталог для ключевой информации нашего сервера и поместим туда получившиеся у нас файлы:

```bash
root@server:~# mkdir /etc/ldap/ssl
root@server:~# chown openldap:openldap /etc/ldap/ssl
root@server:~# chmod 0500 /etc/ldap/ssl
root@server:~# cp certs/server.kb.edu.crt private/server.kb.edu.key /etc/ldap/ssl
```

Установим права доступа для ключевой информации:

```bash
root@server:~# chown openldap:openldap /etc/ldap/ssl/server.kb.edu.{crt,key}
root@server:~# chmod 0400 /etc/ldap/ssl/server.kb.edu.{crt,key}
```

Поместим корневой сертификат в каталог с сертификатами операционной системы и зададим для него права доступа:

```bash
root@server:~# cp certs/rootca.crt /etc/ssl/certs/
root@server:~# chmod 0644 /etc/ssl/certs/rootca.crt
```

В заключение можем удалить запрос CSR, он нам больше не нужен:

```bash
root@server:~# rm -rf certs/server.kb.edu.csr
```

Каталог `/root/CA` теперь выполняет роль удостоверяющего центра. Аналогичным образом его можно создать на другой машине, тогда надо будет копировать секретные ключи и подписанные сертификаты по сети с помощью **scp** или через съёмный носитель. Неплохой идеей будет хранить этот каталог на отдельном носителе и монтировать его только для выпуска новых сертификатов. Сам носитель можно, например, положить в сейф. Процесс создания новых пар ключ-сертификат в будущем будет состоять из трёх этапов:

1. Генерация секретного ключа и запроса на подписание сертификата;
2. Подписание сертификата с помощью закрытого ключа нашего CA (`rootca.key`);
3. Перемещение новых секретного ключа и подписанного сертификата в каталоги, где они будут использоваться.

### Настройка конфигурации TLS

Где работаем: **server**

Вернёмся во временный каталог:

```bash
root@server:~# cd ~/ldap
```

Создадим LDIF-файл `tls-config.ldif` для внесения в каталог конфигурации TLS и запишем в него:

```
dn: cn=config
add: olcTLSVerifyClient
olcTLSVerifyClient: never
-
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/ssl/server.kb.edu.crt
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/ssl/server.kb.edu.key
-
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/rootca.crt
```

Этими записи говорят демону **slapd**, где лежит его сертификат и ключ, где лежит корневой сертификат УЦ и что от клиентов требовать наличие сертификата не нужно. Чтобы окончательно всё запутать, мы могли бы создать сертификаты для всех клиентов. В реальной жизни такая аутентификация клиента принесёт небольшое усиление защиты за счёт большого увеличения работы по сопровождению всех этих сертификатов. Тем более, далее, на этой практике мы настроим механизмы аутентификации Kerberos.

Загрузим конфигурацию TLS в наш каталог:

```bash
root@server:~# ldapmodify -QY EXTERNAL -H ldapi:/// -f tls-config.ldif
modifying entry "cn=config"
```

Теперь наш сервер OpenLDAP должен поддерживать расширения TLS. Перепроверим, что всё в порядке:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b cn=config -s base | grep -i tls
olcTLSVerifyClient: never
olcTLSCertificateFile: /etc/ldap/ssl/server.kb.edu.crt
olcTLSCertificateKeyFile: /etc/ldap/ssl/server.kb.edu.key
olcTLSCACertificateFile: /etc/ssl/certs/rootca.crt
```

Вновь отредактируем конфигурационный файл `/etc/ldap/ldap.conf` и включим поддержку TLS:

```bash
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE    dc=kb,dc=edu
URI     ldap://server.kb.edu

#SIZELIMIT      12
TIMELIMIT       15
TIMEOUT         20
#DEREF          never

# TLS certificates (needed for GnuTLS)
#TLS_CACERT     /etc/ssl/certs/ca-certificates.crt
TLS_CACERT      /etc/ssl/certs/rootca.crt
TLS_REQCERT     demand


```

Обычно для конфигурации `cn=config` перезагрузка службы не требуется, но чтобы созданная нами ключевая информация подхватилась, придётся это сделать:

```bash
root@server:~# service slapd restart
```

Проверьте соединение с сервером с использованием TLS (модификатор `-ZZ`). На этот раз мы выполним запрос с использованием DN нашего администратора:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b cn=config -s base
Enter LDAP Password:
dn: cn=config
objectClass: olcGlobal
cn: config
olcArgsFile: /var/run/slapd/slapd.args
olcPidFile: /var/run/slapd/slapd.pid
olcTLSCACertificateFile: /etc/ssl/certs/rootca.crt
olcTLSCertificateFile: /etc/ldap/ssl/server.kb.edu.crt
olcTLSCertificateKeyFile: /etc/ldap/ssl/server.kb.edu.key
olcTLSVerifyClient: never
```

Обратите внимание, что в запросе **ldapsearch** нам не пришлось указывать имя хоста (`-H ldap://server.kb.edu`).  Всё потому что оно указано в файле `/etc/ldap/ldap.conf`. Что касается TLS, то во время инициирования соединения клиент (на данном этапе клиентский запрос выполняет сам сервер) получает от сервера его подписанный сертификат (`server.kb.edu.crt`). Клиент может ничего не знать о сервере, но у него есть сертификат CA (`rootca.crt`), с помощью которого и проверяется сервер.

### Создание CRL и отзыв сертификатов

Где работаем: **server**

Сертификаты не вечны. Во-первых при создании задаётся его срок действия. При этом частота обновления сертификатов — баланс между безопасностью и удобством. Во-вторых он может быть скомпрометирован, если скомпрометирован его закрытый ключ. Как во втором случае оповестить субъектов доступа, что сертификат более не действителен? Для этого и служит Certificate Revocation List (CRL). Он представляет собой список серийных номеров отозванных сертификатов, подписанный УЦ, который их выпустил.

Вернёмся в каталог удостоверяющего центра:

```bash
root@server:~# cd /root/CA
```

Прежде чем мы сможем сгенерировать CRL, надо создать файл `crlnumber`. Он нужен **openssl**, чтобы отслеживать номер следующего CRL:

```bash
root@server:~# echo 1000 > crlnumber
```

В стандартной конфигурации **openssl** использует CRL V1. \
Раcкомментируйте строку `crl_extensions = crl_ext` в `/etc/ssl/openssl.cnf`, чтобы переключиться на CRL V2. Это хорошая идея, за исключением тех случаев, когда надо использовать именно CRL V1 (например, при использовании сильно устаревшего браузера). Создаём CRL:

```bash
root@server:~# openssl ca -keyfile private/rootca.key -cert certs/rootca.crt -gencrl -out crl/rootca.crl
```

Посмотреть результат можно так:

```bash
root@server:~# openssl crl -in crl/rootca.crl -text
```

Предположим, что закрытый ключ сервера **server** был скомпрометирован. Чтобы оповестить об этом клиентские машины, надо отозвать сертификат  этого сервера, создать CRL, а затем — распространить CRL среди клиентов.

Отзываем сертификат:

```bash
root@server:~# openssl ca -keyfile private/rootca.key -cert certs/rootca.crt -revoke certs/server.kb.edu.crt
```

Заглянем в `index.txt`. Начало записи с нашим сертификатом теперь изменилось с `V` на `R`:

```bash
root@server:~# cat index.txt
R 160116072355Z 150119081313Z 1000 unknown /C=RU/ST=Sverdlovskaya/O=UrFU/OU=KB/CN=server.kb.edu/emailAddress=support@kb.edu
```

Обратите внимание, что копии новых сертификатов так же содержатся в каталоге `newcerts`. При этом имя файла содержит серийный номер сертификата. Когда отзываете сертификаты, вместо файлов в каталоге `certs` Вы можете пользоваться каталогом `newcerts`. Результат будет идентичен. Например:

```bash
root@server:~# openssl ca -keyfile private/rootca.key -cert certs/rootca.crt -revoke newcerts/1000.pem
```

Обновим CRL:

```bash
root@server:~# openssl ca -keyfile private/rootca.key -cert certs/rootca.crt -gencrl -out crl/rootca.crl
```

Взглянём на содержимое CRL:

```bash
root@server:~# openssl crl -in crl/rootca.crl -text
```

Вы должны увидеть нечто подобное:

```bash
...
Revoked Certificates:
    Serial Number: 1000
        Revocation Date: ...
```

Поместим CRL в каталог, где его увидит клиентское ПО:

```bash
root@server:~# cp crl/rootca.crl /etc/ssl
```

Осталось только распространить этот CRL среди клиентов. Мы делаем это простым путём — копированием файлов по сети вручную.

Где работаем: **client**

На каждой клиентской машине надо будет запустить:

```bash
root@client:~# scp root@server.kb.edu:/etc/ssl/certs/rootca.crt /etc/ssl/certs
root@client:~# scp root@server.kb.edu:/etc/ssl/rootca.crl /etc/ssl/
```

Настроим клиентскую конфигурацию в `/etc/ldap/ldap.conf` и укажем, где хранится CRL:

```bash
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE    dc=kb,dc=edu
URI     ldap://server.kb.edu

#SIZELIMIT      12
TIMELIMIT       15
TIMEOUT  20
#DEREF          never

# TLS certificates (needed for GnuTLS)
#TLS_CACERT     /etc/ssl/certs/ca-certificates.crt
TLS_CACERT      /etc/ssl/certs/rootca.crt
TLS_REQCERT     demand
TLS_CRLFILE     /etc/ssl/rootca.crl


```

Для следующей команды потребуется установить на клиенте утилиты ldap:

<pre class="language-bash"><code class="lang-bash"><strong>root@client:~# apt install ldap-utils
</strong></code></pre>

Попробуем получить доступ к серверу каталогов:

```bash
root@client:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b cn=config -s base dn
ldap_start_tls: Connect error (-11)
        additional info: (unknown error code)
```

Наш CRL работает. Но к **slapd** теперь не подключиться. Надо выпустить новый сертификат для сервера.

Где работаем: **server**

Сделаем это простой последовательностью команд. Мы повторяем пройденное, комментарии излишни:

```bash
root@server:~# cd /root/CA
root@server:~# openssl genrsa -out private/server.kb.edu.key 4096
root@server:~# chmod 400 private/server.kb.edu.key
root@server:~# openssl req -sha256 -new \
-key private/server.kb.edu.key -out certs/server.kb.edu.csr \
-subj  /C=RU/ST=Sverdlovskaya/L=Ekaterinburg/O=UrFU/OU=KB/CN=server.kb.edu/emailAddress=support@kb.edu
root@server:~# openssl ca -extensions usr_cert -notext -md sha256 \
-keyfile private/rootca.key -cert certs/rootca.crt \
-in certs/server.kb.edu.csr -out certs/server.kb.edu.crt
root@server:~# chmod 444 certs/server.kb.edu.crt
root@server:~# cp certs/server.kb.edu.crt private/server.kb.edu.key /etc/ldap/ssl
root@server:~# chown openldap:openldap /etc/ldap/ssl/server.kb.edu.{crt,key}
root@server:~# chmod 0400 /etc/ldap/ssl/server.kb.edu.{crt,key}
```

Перезапустим демон **slapd** и убедимся, что к серверу можно подключиться.

```bash
root@server:~# service slapd restart
```

Где работаем: **client**

```bash
root@client:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b cn=config -s base dn
Enter LDAP Password:
dn: cn=config
```

В целом по отзыву сертификатов... Иногда в реальных условиях лучше применить другой подход. Намного удобней, когда клиентские машины самостоятельно выясняют у сервера, действителен конкретный сертификат или нет. В выпускаемые сертификаты [можно вносить запись](https://www.openssl.org/docs/apps/x509v3\_config.html#crl\_distribution\_points\_) `CRL Distribution Points`. Она заставит клиентов самих периодически скачивать новый CRL. Или можно [использовать](http://backreference.org/2010/05/09/ocsp-verification-with-openssl/) [OCSP](https://en.wikipedia.org/wiki/Online\_Certificate\_Status\_Protocol). Но эта тема выходит за рамки данной практики.

### Управление пользователями и группами в OpenLDAP

Где работаем: **server**

На данном этапе у нас есть пустой каталог OpenLDAP. Мы можем проверить это, просто попытавшись извлечь из него информацию:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b dc=kb,dc=edu
Enter LDAP Password:
No such object (32)
```

Если Вам интересно, коды ошибок LDAP описаны в [RFC 4511](http://pro-ldap.ru/tr/rfc/rfc4511.html#appendix-A), в приложении A Результирующие коды LDAP. В случае ошибки **32** сервер информирует нас, что запрашиваемый объект `dc=kb,dc=edu` не найден. Но такую же ошибку можно получить, если ACL сервера запрещают доступ.

Для работы нашего каталога понадобится создать корневой суффикс базы данных и добавить в него две организационных единицы ([Organizational Unit, OU](http://pro-ldap.ru/tr/zytrax/apd/index.html#ou)) для хранения пользователей и групп. Создадим LDIF-файл `users+groups.ldif`, в котором опишем эту информацию:

```
dn: dc=kb,dc=edu
dc: kb
objectClass: top
objectClass: domain

dn: ou=users,dc=kb,dc=edu
ou: Users
objectClass: top
objectClass: organizationalUnit
description: Central location for UNIX users

dn: ou=groups,dc=kb,dc=edu
ou: Groups
objectClass: top
objectClass: organizationalUnit
description: Central location for UNIX groups

dn: ou=services,dc=kb,dc=edu
ou: Services
objectClass: top
objectClass: organizationalUnit
description: Group all services under this OU
```

Добавим эту информацию в наш каталог:

```bash
root@server:~# ldapmodify -a -xZZWD cn=admin,dc=kb,dc=edu -f users+groups.ldif
Enter LDAP Password:
adding new entry "dc=kb,dc=edu"
adding new entry "ou=users,dc=kb,dc=edu"
adding new entry "ou=groups,dc=kb,dc=edu"
adding new entry "ou=services,dc=kb,dc=edu"
```

Можем удостовериться в том, что информация добавлена:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu
Enter LDAP Password:
dn: dc=kb,dc=edu
dc: kb
objectClass: top
objectClass: domain

dn: ou=users,dc=kb,dc=edu
ou: Users
objectClass: top
objectClass: organizationalUnit
description: Central location for UNIX users

dn: ou=groups,dc=kb,dc=edu
ou: Groups
objectClass: top
objectClass: organizationalUnit
description: Central location for UNIX groups

dn: ou=services,dc=kb,dc=edu
ou: Services
objectClass: top
objectClass: organizationalUnit
description: Group all services under this OU
```

Теперь давайте добавим несколько групп:

* **sysadmin**, для объединения системных администраторов Linux;
* **nssproxy**, которая будет использоваться для опроса сервера, чтобы не делать анонимный опрос;
* **test.group**, для тестирования различных частей архитектуры PAM и LDAP.

Этот список не окончательный и может быть дополнен, чтобы соответствовать нуждам Вашей организации.

Для добавления групп в каталог создадим новый LDIF `groups.ldif`:

```
dn: cn=sysadmin,ou=groups,dc=kb,dc=edu
cn: sysadmin
objectClass: top
objectClass: posixGroup
gidNumber: 1100
description: UNIX systems administrators

dn: cn=nssproxy,ou=groups,dc=kb,dc=edu
cn: nssproxy
objectClass: top
objectClass: posixGroup
gidNumber: 801
description: Network Service Switch Proxy

dn: cn=test.group,ou=groups,dc=kb,dc=edu
cn: test.group
objectClass: top
objectClass: posixGroup
gidNumber: 1101
description: Test Group
```

Загрузим LDIF в наш каталог:

```bash
root@server:~# ldapmodify -a -xZZWD cn=admin,dc=kb,dc=edu -f groups.ldif
Enter LDAP Password:
adding new entry "cn=sysadmin,ou=groups,dc=kb,dc=edu"
adding new entry "cn=nssproxy,ou=groups,dc=kb,dc=edu"
adding new entry "cn=test.group,ou=groups,dc=kb,dc=edu"
```

Настало время создать несколько пользователей. И вновь отмечаем, что список может быть расширен в соответствии с потребностями Вашей организации:

* **mpyrev** - персональный пользователь.
* **nssproxy**, который будет использоваться для опроса сервера, чтобы не делать опрос от анонимного пользователя.
* **test.user** - как и тестовая группа, этот пользователь будет использоваться для проверки наших настроек.

Создадим LDIF-файл `users.ldif` для добавления пользователей. Пока не беспокойтесь о паролях, мы сейчас к ним вернёмся:

```
dn: cn=mpyrev,ou=users,dc=kb,dc=edu
uid: mpyrev
gecos: Mikhail Pyrev
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
userPassword: {SSHA}RsAMqOI3647qg1gAZF3x2BKBnp0sEVfa
shadowLastChange: 15140
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1100
gidNumber: 1100
homeDirectory: /home/mpyrev

dn: cn=nssproxy,ou=users,dc=kb,dc=edu
uid: nssproxy
gecos: Network Service Switch Proxy User
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
userPassword: {SSHA}RsAMqOI3647qg1gAZF3x2BKBnp0sEVfa
shadowLastChange: 15140
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/false
uidNumber: 801
gidNumber: 801
homeDirectory: /home/nssproxy

dn: cn=test.user,ou=users,dc=kb,dc=edu
uid: test.user
gecos: Test User
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
userPassword: {SSHA}RsAMqOI3647qg1gAZF3x2BKBnp0sEVfa
shadowLastChange: 15140
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1101
gidNumber: 1101
homeDirectory: /home/test.user
```

Пользователи связаны с группами через атрибут `gidNumber`. Для того, чтобы добавить в группу других пользователей, в запись группы необходимо добавить атрибут `memberUID`, перечислив в нём UID пользователей через запятую. Например, это может выглядеть вот так:

```
dn: cn=sysadmin,ou=groups,dc=kb,dc=edu
cn: sysadmin
objectClass: top
objectClass: posixGroup
gidNumber: 1100
description: UNIX systems administrators
memberUID: 801,1101
```

Добавьте пользователей в каталог:

```bash
root@server:~# ldapmodify -a -xZZWD cn=admin,dc=kb,dc=edu -f users.ldif
Enter LDAP Password:
adding new entry "cn=mpyrev,ou=users,dc=kb,dc=edu"
adding new entry "cn=nssproxy,ou=users,dc=kb,dc=edu"
adding new entry "cn=test.user,ou=users,dc=kb,dc=edu"
```

Теперь установим пароли для пользователей. Повторите нижеследующую команду для всех пользователей. Обратите внимание, что сначала Вам будет предложено дважды ввести пароль изменяемой записи, а затем — пароль записи `cn=admin,dc=kb,dc=edu`:

```bash
root@server:~# ldappasswd -xZZWD cn=admin,dc=kb,dc=edu -S cn=nssproxy,ou=users,dc=kb,dc=edu
New password:
Re-enter new password:
Enter LDAP Password:
```

Если заглянуть в журнал `/var/log/slapd.log`, то можно увидеть строки, говорящие об изменении записи:

```
slapd[5319]: conn=1022 op=1 PASSMOD id="cn=nssproxy,ou=users,dc=kb,dc=edu" new
slapd[5319]: conn=1022 op=1 RESULT oid= err=0 text=
```

Для того, чтобы пользователь **nssproxy** имел доступ к информации нашего каталога, необходимо поправить ACL базы данных с пользователями и группами. Но какое у этой базы DN? Давайте вспомним:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b cn=config dn | grep -i database
Enter LDAP Password:
dn: olcDatabase={-1}frontend,cn=config
dn: olcDatabase={0}config,cn=config
dn: olcDatabase={1}mdb,cn=config
dn: olcDatabase={2}monitor,cn=config
```

В разделе [Инициализация базы данных](3-linux.md#inicializaciya-bazy-dannykh) мы дали нашей учетной записи администратора очень широкие права, поэтому он может заглянуть в конфигурацию `cn=config`.

Теперь мы знаем, что требуется отредактировать ACL записи `olcDatabase={1}mdb,cn=config`.

Давайте проверим, какие ACL настроены для данного DN (вывод отформатирован):

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b olcDatabase={1}mdb,cn=config olcAccess
Enter LDAP Password:
dn: olcDatabase={1}mdb,cn=config
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external ,cn=auth" manage
  by * break
olcAccess: {1}to attrs=userPassword
  by self write
  by anonymous auth
olcAccess: {2}to *
  by self read
```

Мы должны немного изменить ACL, чтобы предоставить доступ пользователю **nssproxy** на чтение атрибутов учётных записей. Для этого создадим LDIF-файл `nssproxy.acl.ldif`:

```
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external ,cn=auth" manage
  by * break
-
add: olcAccess
olcAccess: {1}to attrs=userPassword
  by self write
  by anonymous auth
-
add: olcAccess
olcAccess: {2}to *
  by self read
  by dn.base="cn=nssproxy,ou=users,dc=kb,dc=edu" read
```

Загрузим LDIF с изменениями:

```bash
root@server:~# ldapmodify -xZZWD cn=admin,dc=kb,dc=edu -f nssproxy.acl.ldif
Enter LDAP Password:
modifying entry "olcDatabase={1}mdb,cn=config"
```

Проверим, можем ли мы просматривать информацию в каталоге от имени пользователя **nssproxy**. Для этого выполним запрос с использованием его учётных данных. Результатом запроса должно быть всё DIT (Directory Information Tree). Опустим его, для краткости:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=nssproxy,ou=users,dc=kb,dc=edu "(objectClass=*)"
Enter LDAP Password:
dn: dc=kb,dc=edu
...
```

Убедитесь, что другие учётные записи пользователей не имеют доступа к DIT:

<pre class="language-bash"><code class="lang-bash">root@server:~# ldapsearch -xZZLLLWD cn=<a data-footnote-ref href="#user-content-fn-6">mpyrev</a>,ou=users,dc=kb,dc=edu "(objectClass=*)"
Enter LDAP Password:
No such object (32)
</code></pre>

### Настройка аутентификации пользователей через OpenLDAP на клиенте

Где работаем: **client**

Перейдём к настройке клиентской рабочей станции. Для начала установим необходимые пакеты:

<pre class="language-bash"><code class="lang-bash"><strong>root@client:~# apt-get install libnss-ldapd libpam-ldapd
</strong></code></pre>

При установке нам будут предложены некоторые настройки . Далее мы всё равно внесём в конфигурацию изменения, но это нужно сделать, чтобы все пакеты нормально установились:

* URI сервера LDAP: `ldap://server.kb.edu`;
* База поиска сервера LDAP: `dc=kb,dc=edu`;
* Имена настраиваемых служб: `passwd`,`group`,`shadow`,`netgroup`.

Прежде чем делать запросы к LDAP-серверу, проверим параметры клиента в `/etc/ldap/ldap.conf`:

```
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE    dc=kb,dc=edu
URI     ldap://server.kb.edu

#SIZELIMIT      12
TIMELIMIT       15
TIMEOUT         20
#DEREF          never

# TLS certificates (needed for GnuTLS)
#TLS_CACERT     /etc/ssl/certs/ca-certificates.crt
TLS_CACERT      /etc/ssl/certs/rootca.crt
TLS_REQCERT     demand
TLS_CRLFILE     /etc/ssl/rootca.crl


```

Конечно, для того, чтобы всё заработало, сертификат нашего Certificate Authority (`rootca.crt`) и CRL-файл (`rootca.crl`) должны быть на месте.

Проверим работоспособность простым запросом. Результат опустим (Вы должны увидеть всё DIT):

```bash
root@client:~# ldapsearch -xZZLLLWD cn=nssproxy,ou=users,dc=kb,dc=edu
Enter LDAP Password:
dn: dc=kb,dc=edu
...
```

Поправим конфигурацию нашей локальной службы имён LDAP в файле `/etc/nslcd.conf`. Измените значение директивы `bindpw` на пароль записи `cn=nssproxy,ou=users,dc=kb,dc=edu`, который мы задавали ранее:

<pre class="language-cmake"><code class="lang-cmake"># /etc/nslcd.conf
# nslcd configuration file. See nslcd.conf(5)
# for details.

# The user and group nslcd should run as.
uid nslcd
gid nslcd

# The location at which the LDAP server(s) should be reachable.
uri ldap://server.kb.edu

# The search base that will be used for all queries.
base dc=kb,dc=edu

# The LDAP protocol version to use.
#ldap_version 3

# The DN to bind with for normal lookups.
#binddn cn=annonymous,dc=example,dc=net
#bindpw secret

binddn cn=nssproxy,ou=users,dc=kb,dc=edu
bindpw <a data-footnote-ref href="#user-content-fn-7">P@ssw0rdproxy</a>

# The DN used for password modifications by root.
#rootpwmoddn cn=admin,dc=example,dc=com
rootpwmoddn cn=admin,dc=kb,dc=edu

base group ou=groups,dc=kb,dc=edu
base passwd ou=users,dc=kb,dc=edu
base shadow ou=users,dc=kb,dc=edu

bind_timelimit 5
timelimit 10
idle_timelimit 60

# SSL options
#ssl off
ssl start_tls
#tls_reqcert demand
#tls_cacertfile /etc/ssl/certs/ca-certificates.crt
tls_reqcert allow
tls_cacertfile /etc/ssl/certs/rootca.crt

nss_initgroups_ignoreusers bin,daemon,games,lp,mail,nobody,nslcd,root,sshd,sync,uucp
nss_initgroups_ignoreusers sys,man,news,proxy,www-data,backup,list,irc,gnats,landscape

# The search scope.
#scope sub


</code></pre>

Краткое описание использованных директив:

* С помощью директивы `base` мы сообщаем демону **nslcd**, где в DIT искать ту или иную информацию;
* `bind_timelimit` ограничивает время на установление соединения с сервером пятью секундами;
* `timelimit` устанавливает максимальное время ожидания ответа от сервера в 10 секунд;
* `idle_timelimit` заставит **nslcd** разорвать соединение с сервером, если в течении минуты в соединении не было никакой активности;
* `ssl` заставляет клиента использовать TLS при подключении к серверу;
* `tls_reqcert` и `tls_cacertfile` подобны директивам `TLS_REQCERT` и `TLS_CACERT` в конфигурации _/etc/ldap/ldap.conf_ (необходимость проверки сертификата сервера и путь к корневому сертификату для выполнения проверки);
* `nss_initgroups_ignoreusers` описывает пользователей системы, поиск которых не нужно производить в DIT (чтобы мы могли работать с машиной при проблемах с доступом к серверу каталогов). В значение этой директивы следует внести имена всех общесистемных пользователей.

Поменяем права доступа к `nslcd.conf`, потому что теперь в нём хранится информация для аутентификации:

```bash
root@client:~# chmod 600 /etc/nslcd.conf
root@client:~# chown nslcd:nslcd /etc/nslcd.conf
```

Проверим содержимое `/etc/nsswitch.conf`:

```cmake
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         compat ldap
group:          compat ldap
shadow:         compat ldap

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis ldap

```

Убедимся в том, что демон **nslcd** запускается при старте системы и перезапустим его:

```bash
root@client:~# update-rc.d nslcd defaults
root@client:~# service nslcd restart
```

Убедимся, что в системе **client** **НЕТ** учётной записи с именем **nssproxy**. Следующая команда должна выполняться без результата:

```bash
root@client:~# grep nssproxy /etc/passwd
```

Убедимся так же, что кэширующий демон службы имён загружается при старте системы и перезапустим его:

```bash
root@client:~# update-rc.d nscd defaults
root@client:~# service nscd restart
```

Сделаем пару запросов к LDAP-серверу, используя настроенную нами систему:

```bash
root@client:~# getent passwd test.user
test.user:x:1101:1101:Test User:/home/test.user:/bin/bash
root@client:~# getent group test.group
test.group:*:1101:
root@client:~# getent shadow test.user
test.user:*:15140:0:99999:7:::0
```

Не беспокойтесь, хэш пароля мы видеть не должны.

Отлично, всё работает! Вышеприведённые результаты команд означают, что система **client** может осуществлять поиск по данным пользователей  и групп в нашем каталоге OpenLDAP.

Создадим домашний каталог нашего тестового пользователя и установим для него права доступа:

```bash
root@client:~# mkdir /home/test.user
root@client:~# chown test.user:test.group /home/test.user
```

Запустим в отдельном терминале вывод журнала аутентификации:

```bash
root@client:~# tail -f /var/log/auth.log
```

В другом терминале выполним:

<pre class="language-bash"><code class="lang-bash"><strong>mpyrev@client:~$ su - <a data-footnote-ref href="#user-content-fn-8">test.user</a>
</strong></code></pre>

Мы должны получить приглашение командной строки пользователя **test.user**.

Теперь попробуем зайти на машину **client** по сети с использованием **ssh** под учётной записью **test.user**.

Демон **ssh** в конфигурации по умолчанию должен работать с поддержкой PAM (и, соответственно, поддерживать аутентификацию через LDAP). Но на всякий случай приведём [рабочую конфигурацию](#user-content-fn-9)[^9] `/etc/ssh/sshd_config`:

<pre class="language-cmake"><code class="lang-cmake">#       $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
<strong>AddressFamily inet
</strong><strong>AllowGroups sysadmin test.group
</strong>#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
<strong>SyslogFacility AUTHPRIV
</strong>#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
<strong>#PasswordAuthentication yes
</strong>#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
<strong>AllowTcpForwarding no
</strong>#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
<strong>ClientAliveInterval 120
</strong><strong>ClientAliveCountMax 2
</strong>#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server


</code></pre>

Обратите внимание на строку с директивой `AllowGroups`. С помощью неё мы ограничиваем список групп пользователей, которые могут быть аутентифицированы через **ssh**. За информацией по остальным директивам обратитесь к документации.

Проверим работу с использованием какой-нибудь третьей машины. Например, выполним на нашей [хостовой машине запрос через PuTTY](#user-content-fn-10)[^10]:

```bash
login as: test.user
test.user@localhost's password:
Linux client 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Apr 29 12:09:06 2023 from 192.168.3.2
test.user@client:~$

```

Где работаем: **server**

Давайте заглянем в `/var/log/slapd.log` на нашем сервере. Мы можем обнаружить там следующие строки:

```
ldap-srv slapd[1304]: <= mdb_equality_candidates: (objectClass) not indexed
ldap-srv slapd[1304]: <= mdb_equality_candidates: (uid) not indexed
```

С помощью следующей команды мы можем убедиться, что в нашем каталоге пока нет никаких индексов:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b olcDatabase={1}mdb,cn=config olcDbIndex
Enter LDAP Password:
dn: olcDatabase={1}mdb,cn=config
```

Это значит, что в нашей базе данных надо создать индексы для атрибутов из журнала  `/var/log/slapd.log`. Поэтому создадим ещё один LDIF-файл `posixAccount.indexes.ldif` и запишем в него:

```
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: default pres,eq
-
add: olcDbIndex
olcDbIndex: uid
-
add: olcDbIndex
olcDbIndex: cn,sn pres,eq,sub
-
add: olcDbIndex
olcDbIndex: objectClass eq
-
add: olcDbIndex
olcDbIndex: memberUid eq
-
add: olcDbIndex
olcDbIndex: uniqueMember eq
-
add: olcDbIndex
olcDbIndex: uidNumber
-
add: olcDbIndex
olcDbIndex: gidNumber eq
```

Зачем мелочиться? Укажем побольше индексируемых атрибутов. И загрузим конфигурацию в наш каталог:

```bash
root@server:~# ldapadd -xZZWD cn=admin,dc=kb,dc=edu -f posixAccount.indexes.ldif
Enter LDAP Password:
modifying entry "olcDatabase={1}mdb,cn=config"
```

Проверим результат изменений:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=admin,dc=kb,dc=edu -b olcDatabase={1}mdb,cn=config olcDbIndex
Enter LDAP Password:
dn: olcDatabase={1}mdb,cn=config
olcDbIndex: default pres,eq
olcDbIndex: uid
olcDbIndex: cn,sn pres,eq,sub
olcDbIndex: objectClass eq
olcDbIndex: memberUid eq
olcDbIndex: uniqueMember eq
olcDbIndex: uidNumber
olcDbIndex: gidNumber eq
```

Отлично! Теперь в журнале `/var/log/slapd.log` не должно быть ошибок.

### Kerberos KDC с использованием OpenLDAP в качестве бэкэнда и аутентификацией SASL GSSAPI

В этом разделе мы научимся использовать OpenLDAP 2.4 в качестве хранилища принципалов (principals) [Kerberos](http://web.mit.edu/kerberos/) и разберем, как настраивать клиентские рабочие станции. [Kerberos](https://ru.wikipedia.org/wiki/Kerberos) — это сетевой протокол, который работает на основе билетов (tickets) и позволяет передавать данные через незащищённые сети для безопасной идентификации и аутентификации. Его дизайн преимущественно опирается на клиент-серверную модель и позволяет произвести взаимную аутентификацию субъекта и объекта доступа. Сообщения протокола устойчивы к прослушиванию и атакам повтора. Kerberos работает на основе криптографии с симметричным ключом и требует наличия доверенной третьей стороны, а так же может применяться с использованием криптографии с открытым ключом на некоторых этапах процесса аутентификации.

На текущем практическом занятии в роли сервера Kerberos будем использовать **server**. Но ничто не мешает Вам использовать для этого отдельную машину.

### Настройка сервера Kerberos <a href="#s1" id="s1"></a>

Где работаем: **server**

**Включите** [**NTP**](https://ru.wikipedia.org/wiki/NTP) **и убедитесь, что сервер и клиенты синхронизированы!** Мы не будем описывать, как настраивать [NTP](http://www.ntp.org/). Вы можете найти множество примеров в сети, если потребуется.

Теперь, когда наш сервер OpenLDAP настроен, мы можем приступить к конфигурированию сервера Kerberos. В этом разделе мы опишем, как развернуть центр распределения ключей Kerberos (KDC, Key Distribution Center) для области (realm) KB.EDU. Начнём с установки необходимых пакетов. В процессе будет так же установлено несколько пакетов-зависимостей. Пакет **wamerican** создаст файл `/usr/share/dict/words`, используемый **kadmind**. Вместо него можно использовать любой другой словарь или несколько словарей. Чтобы посмотреть их список, посмотрите содержимое метапакета **wordlist**.

```bash
root@server:~# apt-get install krb5-kdc krb5-pkinit krb5-admin-server wamerican libsasl2-modules-gssapi-mit
```

В разделе [Добавление наборов схем данных](3-linux.md#dobavlenie-naborov-skhem-dannykh) мы уже установили схему Kerberos для OpenLDAP сервера, но давайте лишний раз убедимся, что она на месте:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn | grep -i kerberos
dn: cn={12}kerberos,cn=schema,cn=config
```

Схема на месте. В ней содержится достаточно много объектов. Чтобы посмотреть их все, используйте следующий запрос:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b cn={12}kerberos,cn=schema,cn=config | grep NAME | cut -d' ' -f5 | sort
```

Эта команда должна вернуть список объектов. Это важно, потому что если новых атрибутов Kerberos LDAP нет, то **kdb5\_ldap\_util**(8) выдаст следующую ошибку:

```cmake
kdb5_ldap_util: Kerberos Container create FAILED: No such object while creating realm 'EXAMPLE.COM'
```

Итак, у нас есть набор схемы данных и объекты. Но есть ли у нас контейнер для принципалов Kerberos?

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b ou=services,dc=kb,dc=edu dn | grep -i kerberos
```

Нет, нету. Создадим его, а заодно — пользователя и группу, которые будут использоваться Kerberos для взаимодействия с сервером OpenLDAP. Используем для этой цели вот такой LDIF-файл `kerberos.ldif`:

```cmake
# Создадим контейнер для Kerberos
dn: cn=kerberos,ou=services,dc=kb,dc=edu
cn: kerberos
objectClass: top
objectClass: krbContainer

# Добавим группу krbadmin
dn: cn=krbadmin,ou=groups,dc=kb,dc=edu
objectClass: top
objectClass: posixGroup
cn: krbadmin
gidNumber: 800
description: Kerberos administrator's group.

# Добавим пользователя krbadmin
dn: cn=krbadmin,ou=users,dc=kb,dc=edu
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: top
cn: krbadmin
givenName: Kerberos Administrator
mail: kerberos.admin@kb.edu
sn: krbadmin
uid: krbadmin
uidNumber: 800
gidNumber: 800
homeDirectory: /home/krbadmin
loginShell: /bin/false
displayname: Kerberos Administrator
```

Загрузим этот файл в базу данных нашего сервера каталогов:

```bash
root@server:~# ldapadd -xZZWD cn=admin,dc=kb,dc=edu -f ~/ldap/kerberos.ldif
Enter LDAP Password:
adding new entry "cn=kerberos,ou=services,dc=kb,dc=edu"
adding new entry "cn=krbadmin,ou=groups,dc=kb,dc=edu"
adding new entry "cn=krbadmin,ou=users,dc=kb,dc=edu"
```

Зададим пароль для пользователя **krbadmin**. Сохраните пароль.

```bash
root@server:~# ldappasswd -xZZWSD "cn=admin,dc=kb,dc=edu" "cn=krbadmin,ou=users,dc=kb,dc=edu"
```

Создадим конфигурационный файл Kerberos `/etc/krb5.conf`:

```cmake
[logging]
        default = SYSLOG:INFO:LOCAL1
        kdc = SYSLOG:NOTICE:LOCAL1
        admin_server = SYSLOG:WARNING:LOCAL1

[libdefaults]
        default_realm = KB.EDU
        dns_lookup_realm = false
        dns_lookup_kdc = false
        ticket_lifetime = 24h
        renew_lifetime = 7d

# The following krb5.conf variables are only for MIT Kerberos.
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true

# The following encryption type specification will be used by MIT Kerberos
# if uncommented.  In general, the defaults in the MIT Kerberos code are
# correct and overriding these specifications only serves to disable new
# encryption types as they are added, creating interoperability problems.
#
# The only time when you might need to uncomment these lines and change
# the enctypes is if you have local software that will break on ticket
# caches containing ticket encryption types it doesn't know about (such as
# old versions of Sun Java).

#       default_tgs_enctypes = des3-hmac-sha1
#       default_tkt_enctypes = des3-hmac-sha1
#       permitted_enctypes = des3-hmac-sha1

# The following libdefaults parameters are only for Heimdal Kerberos.
        fcc-mit-ticketflags = true

[realms]
        KB.EDU = {
                kdc = server.kb.edu
                admin_server = server.kb.edu
                default_domain = kb.edu
                database_module = openldap_ldapconf
        }

[domain_realm]
        .kb.edu = KB.EDU
        kb.edu = KB.EDU

[appdefaults]
        pam = {
                debug = false
                ticket_lifetime = 36000
                renew_lifetime = 36000
                forwardable = true
                krb4_convert = false
        }

[dbmodules]
        openldap_ldapconf = {
                db_library = kldap
                ldap_kerberos_container_dn = cn=kerberos,ou=services,dc=kb,dc=edu
                ldap_kdc_dn = cn=krbadmin,ou=users,dc=kb,dc=edu
                 # этот объект должен иметь права на чтение контейнера области,
                 # контейнера принципала и поддеревьев области
                ldap_kadmind_dn = cn=krbadmin,ou=users,dc=kb,dc=edu
                 # этот объект должен иметь права на чтение контейнера области,
                 # контейнера принципала и поддеревьев области
                ldap_service_password_file = /etc/krb5.d/stash.keyfile
                ldap_servers = ldapi:///
                ldap_conns_per_server = 5
        }

```

Теперь создадим список контроля доступа (ACL) администратора Kerberos. Не путайте этот ACL с ACL OpenLDAP. Это не одно и то же. Чуть ниже мы поработаем с ACL для OpenLDAP. Создадим файл `/etc/krb5kdc/kadm5.acl` с таким содержимым:

```cmake
*/admin@KB.EDU *
```

Отредактируем `/etc/krb5kdc/kdc.conf`, конфигурационный файл службы аутентификации (AS, Authentication Service) и центра распределения ключей (KDC):

```cmake
[kdcdefaults]
    #kdc_ports = 750,88
    kdc_ports = 88
    kdc_tcp_ports = 88

[realms]
    KB.EDU = {
        #database_name = /var/lib/krb5kdc/principal
        #admin_keytab = FILE:/etc/krb5kdc/kadm5.keytab
        admin_keytab = /etc/krb5kdc/kadm5.keytab
        acl_file = /etc/krb5kdc/kadm5.acl
        dict_file = /usr/share/dict/words
        #key_stash_file = /etc/krb5kdc/stash
        #kdc_ports = 750,88
        #max_life = 10h 0m 0s
        #max_renewable_life = 7d 0h 0m 0s
        #master_key_type = des3-hmac-sha1
        #master_key_type = aes256-cts
        #supported_enctypes = aes256-cts:normal aes128-cts:normal
        supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal
        #default_principal_flags = +preauth
    }

```

В отдельном терминале включите просмотр журнала сервера OpenLDAP. Так мы сможем увидеть сообщения, генерируемые командой **kdb5\_ldap\_util**(8):

```bash
root@server:~# tail -F /var/log/slapd.log
```

Создайте каталог-тайник для пароля. В файле `/etc/krb5.conf` он указан в переменной `ldap_service_password_file`:

```bash
root@server:~# mkdir /etc/krb5.d
root@server:~# chmod u=rwx,g=,o= /etc/krb5.d
```

Извлеките пароль пользователя `cn=krbadmin,ou=users,dc=kb,dc=edu`, используя **kdb5\_ldap\_util**(8):

```bash
root@server:~# kdb5_ldap_util -D "cn=admin,dc=kb,dc=edu" stashsrvpw -f /etc/krb5.d/stash.keyfile cn=krbadmin,ou=users,dc=kb,dc=edu
```

Вам будет предложено ввести мастер-пароль базы данных.&#x20;

{% hint style="danger" %}
Очень важно, чтобы вы **не забыли** этот пароль!
{% endhint %}

В основном терминале выполните следующую команду для добавления записей Kerberos в базу данных OpenLDAP:

```bash
root@server:~# kdb5_ldap_util -D "cn=admin,dc=kb,dc=edu" create -subtrees "cn=kerberos,ou=services,dc=kb,dc=edu" -r KB.EDU -s
```

Вышеуказанная команда создаст несколько записей в `cn=kerberos,ou=services,dc=kb,dc=edu`. Чтобы увидеть их, выполните запрос:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b cn=kerberos,ou=services,dc=kb,dc=edu dn
```

Но может ли кто-нибудь кроме пользователя `cn=admin` увидеть нашу информацию Kerberos? Это было-бы не очень мудро. Поэтому давайте взглянем на ACL нашего сервера OpenLDAP:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b olcDatabase={1}mdb,cn=config olcAccess
```

Что нам надо сделать, так это дать права доступа на чтение/запись администратору Kerberos (`cn=krbadmin,ou=users,dc=kb,dc=edu`) к поддереву `cn=kerberos,ou=services,dc=kb,dc=edu`. Никакой другой пользователь не должен иметь доступа к этой информации (за исключением администратора каталога).

Для этого сформируем LDIF-файл `kerberos.acl.ldif`:

```cmake
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage
  by * break
olcAccess: {1}to attrs=userPassword,userPKCS12
  by self write
  by anonymous auth
olcAccess: {2}to attrs=shadowLastChange
  by self write
olcAccess: {3}to dn.subtree="cn=kerberos,ou=services,dc=kb,dc=edu"
  by dn.exact="cn=krbadmin,ou=users,dc=kb,dc=edu" manage
olcAccess: {4}to *
  by dn.base="cn=nssproxy,ou=users,dc=kb,dc=edu" read
  by self read
```

Загрузим данные в сервер каталогов:

```bash
root@server:~# ldapmodify -QY EXTERNAL -H ldapi:/// -f kerberos.acl.ldif
```

Убедитесь, что новые ACL работают. Суперпользователь и пользователь `cn=admin,dc=kb,dc=edu` должны иметь доступ к поддереву `cn=kerberos,ou=services,dc=kb,dc=edu`. Пользователь `cn=nssproxy,ou=users,dc=kb,dc=edu` не сможет обнаружить само существование контейнера Kerberos, а `cn=krbadmin,ou=users,dc=kb,dc=edu` должен не только видеть контейнер, но и иметь для него права на чтение/запись. Мы так же должны убедиться, что обычные пользователи всё ещё могут использовать сервер OpenLDAP для аутентификации и что они могут менять себе пароли.

Этот запрос возвращает поддерево `cn=kerberos,ou=services,dc=kb,dc=edu`:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=krbadmin,ou=users,dc=kb,dc=edu -b cn=kerberos,ou=services,dc=kb,dc=edu dn
```

Следующий запрос должен завершиться с ошибкой `No such object (32)`:

```bash
root@server:~# ldapsearch -xZZLLLWD cn=nssproxy,ou=users,dc=kb,dc=edu -b cn=kerberos,ou=services,dc=kb,dc=edu dn
```

Теперь с клиентской машины убедимся, что пользователь, учётная запись которого хранится на сервере каталогов, может менять свой пароль:

Где работаем: **client**

```bash
root@client:~# su - test.user
test.user@client:~$ passwd
(current) LDAP Password:
Новый пароль:
Повторите ввод нового пароля:
passwd: пароль успешно обновлён
```

Где работаем: **server**

Настало время настроить журналирование. Для начала добавим в конфигурацию **rsyslog** следующие строки (`/etc/rsyslog.conf`):

```systemd
...

# Пишем журнал демона krb5-kdc в /var/log/krb5kdc.log:
if $programname == 'krb5kdc' then /var/log/krb5kdc.log
& ~

# Пишем журнал демона krb5-admin-server в /var/log/kadmind.log:
if $programname == 'kadmind' then /var/log/kadmind.log
& ~
```

Создадим файлы журналов и зададим для них права доступа:

```bash
root@server:~# touch /var/log/{krb5kdc,kadmind}.log
root@server:~# chmod 0640 /var/log/{krb5kdc,kadmind}.log
root@server:~# chown root:adm /var/log/{krb5kdc,kadmind}.log
```

Настроим **logrotate**. Создадим два конфигурационных файла.

Файл `/etc/logrotate.d/krb5kdc` для демона **krb5-kdc**:

```systemd
/var/log/krb5kdc.log {
 daily
 missingok
 rotate 7
 compress
 delaycompress
 notifempty
}
```

Файл `/etc/logrotate.d/kadmind` для демона **krb5-admin-server**:

```systemd
/var/log/kadmind.log {
 daily
 missingok
 rotate 7
 compress
 delaycompress
 notifempty
}
```

Перезапустим демон **rsyslog**, чтобы конфигурация вступила в силу. Добавим демоны **krb5-kdc** и **krb5-admin-server** в автоматический запуск, затем запустим их:

```bash
root@server:~# service rsyslog restart
root@server:~# update-rc.d krb5-kdc defaults
root@server:~# update-rc.d krb5-admin-server defaults
root@server:~# service krb5-kdc start
root@server:~# service krb5-admin-server start
```

Загляните в журнал `/var/log/krb5kdc.log`. Вы должны увидеть там подобную строку:

```systemd
...
Dec  1 15:49:54 ldap-srv krb5kdc[1992]: commencing operation
```

Проверим, что наши демоны слушают необходимые порты. Порт 88 (**krb5kdc**), порты 464 и 749 (**kadmind**):

```bash
root@server:~# ss -tan | egrep ':88|:464|:749'
LISTEN     0      2                         *:749                      *:*
LISTEN     0      5                         *:464                      *:*
LISTEN     0      5                         *:88                       *:*
```

Поздравляю, теперь у Вас есть рабочий сервер аутентификации (AS) и сервер распределения ключей (KDC) MIT Kerberos 5!

Что дальше? В первую очередь нам необходимо создать принципал для локального сервера. Затем — принципал для пользователя **test.user**. Сделаем это так (разобьем вывод на части, чтобы добавить комментарии):

```bash
root@server:~# kadmin.local
Authenticating as principal root/admin@KB.EDU with password.
```

Здесь мы создаём принципал машины для текущего сервера (на котором залогинены). Далее выделен вводимый нами текст:

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  addprinc -randkey host/server.kb.edu@KB.EDU
</strong>WARNING: no policy specified for host/server.kb.edu@KB.EDU; defaulting to no policy
Principal "host/server.kb.edu@KB.EDU" created.
</code></pre>

После создания принципала сервера мы можем добавить его в набор ключей Kerberos (_/etc/krb5.keytab_):

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  ktadd host/server.kb.edu@KB.EDU
</strong>Entry for principal host/server.kb.edu@KB.EDU with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/server.kb.edu@KB.EDU with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/server.kb.edu@KB.EDU with kvno 2, encryption type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/server.kb.edu@KB.EDU with kvno 2, encryption type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
</code></pre>

Следующим шагом создадим принципал для персонального пользователя **mpyrev** и зададим для него пароль:

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  addprinc mpyrev@KB.EDU
</strong>WARNING: no policy specified for mpyrev@KB.EDU; defaulting to no policy
Enter password for principal "mpyrev@KB.EDU": 
Re-enter password for principal "mpyrev@KB.EDU":                             
Principal "mpyrev@KB.EDU" created.
</code></pre>

Теперь создадим принципал пользователя с правами администратора. Помните файл `/etc/krb5kdc/kadm5.acl`? Вот где он вступает в игру. С помощью этого файла пользователи с префиксом **/admin** имеют права администратора на доступ к области Kerberos. Это значит, что они могут создавать и удалять записи пользователей и политики доступа. Поэтому убедитесь, что Вы знаете этих пользователей и доверяете им!

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  addprinc mpyrev/admin@KB.EDU
</strong>WARNING: no policy specified for mpyrev/admin@KB.EDU; defaulting to no policy
Enter password for principal "mpyrev/admin@KB.EDU": 
Re-enter password for principal "mpyrev/admin@KB.EDU": 
Principal "mpyrev/admin@KB.EDU" created.
</code></pre>

Посмотрим список текущих принципалов в области:

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  get_principals
</strong>K/M@KB.EDU
krbtgt/KB.EDU@KB.EDU
kadmin/admin@KB.EDU
kadmin/server.kb.edu@KB.EDU
kiprop/server.kb.edu@KB.EDU
kadmin/changepw@KB.EDU
kadmin/history@KB.EDU
host/server.kb.edu@KB.EDU
mpyrev@KB.EDU
mpyrev/admin@KB.EDU
</code></pre>

А эта команда выведет подробную информацию о принципале **host/server.kb.edu@KB.EDU**:

<pre class="language-bash"><code class="lang-bash"><strong>kadmin.local:  getprinc host/server.kb.edu@KB.EDU
</strong>Principal: host/server.kb.edu@KB.EDU
Expiration date: [never]
Last password change: Сб апр 29 15:57:50 +05 2023
Password expiration date: [never]
Maximum ticket life: 1 day 00:00:00
Maximum renewable life: 0 days 00:00:00
Last modified: Сб апр 29 15:57:50 +05 2023 (root/admin@KB.EDU)
Last successful authentication: [never]
Last failed authentication: [never]
Failed password attempts: 0
Number of keys: 4
Key: vno 2, aes256-cts-hmac-sha1-96
Key: vno 2, aes128-cts-hmac-sha1-96
Key: vno 2, DEPRECATED:des3-cbc-sha1
Key: vno 2, DEPRECATED:arcfour-hmac
MKey: vno 1
Attributes:
Policy: [none]
<strong>kadmin.local:  exit
</strong></code></pre>

Обратите внимание, что был создан новый файл `/etc/krb5.keytab`. Вот почему мы запустили бинарник **kadmin.local** от имени суперпользователя. Иначе мы получили бы следующую ошибку:

```
mpyrev@server:~$ kadmin.local
Authenticating as principal mpyrev/admin@KB.EDU with password.
kadmin.local: Error reading password from stash: Permission denied while initializing kadmin.local interface
```

Только суперпользователь имеет доступ на чтение к файлу-тайнику (`/etc/krb5.d/stash.keyfile`).

Давайте посмотрим, что записано в `/etc/krb5.keytab`:

```bash
root@server:~# klist -ek /etc/krb5.keytab
Keytab name: FILE:/etc/krb5.keytab
KVNO Principal
---- --------------------------------------------------------------------------
   2 host/server.kb.edu@KB.EDU (aes256-cts-hmac-sha1-96)
   2 host/server.kb.edu@KB.EDU (aes128-cts-hmac-sha1-96)
   2 host/server.kb.edu@KB.EDU (DEPRECATED:des3-cbc-sha1)
   2 host/server.kb.edu@KB.EDU (DEPRECATED:arcfour-hmac)
```

Как мы можем видеть, это список ключей шифрования машины **server**. Неудивительно, что доступ предоставлен только суперпользователю!

У нас осталась ещё одна вещь на сервере, которую надо поправить. Это ошибка

```systemd
mdb_equality_candidates: (krbPrincipalName) not indexed
```

в журнале `/var/log/slapd.log`. Создадим LDIF-файл `kerberos.indexes.ldif`, который внесёт поправки в индексы:

```systemd
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: krbPrincipalName eq
-
add: olcDbIndex
olcDbIndex: ou eq
```

Добавим эти индексы в сервер каталогов:

```bash
root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f kerberos.indexes.ldif
```

Отлично! Теперь у нас есть настроенный KDC!

### Настройка клиента Kerberos <a href="#s2" id="s2"></a>

С действующим KDC мы можем заставить клиентские машины и сервисы использовать его. Обозначим основные цели по настройке клиентов, чтобы интегрировать их в инфраструктуру Kerberos:

* Аутентификация Kerberos для **sshd**.
* Аутентификация OpenLDAP с помощью SASL GSSAPI.

#### Аутентификация Kerberos для sshd

Где работаем: **client**

{% hint style="warning" %}
Клиенты Kerberos должны иметь возможность подключения к TCP портам KDC с номерами 88 и 749
{% endhint %}

Установим необходимые пакеты:

```bash
root@client:~# apt-get install krb5-user libpam-krb5 libsasl2-modules-gssapi-mit
```

Во время установки в качестве области Kerberos можете задать KB.EDU, а в качестве серверов, обслуживающих область, **server.kb.edu**.

Отредактируйте конфигурацию клиента в файле `/etc/krb5.conf` следующим образом:

```systemd
[logging]
        default = SYSLOG:INFO:LOCAL1
        kdc = SYSLOG:NOTICE:LOCAL1
        admin_server = SYSLOG:WARNING:LOCAL1

[libdefaults]
        default_realm = KB.EDU
        dns_lookup_realm = false
        dns_lookup_kdc = false
        ticket_lifetime = 24h
        renew_lifetime = 7d

# The following krb5.conf variables are only for MIT Kerberos.
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true

# The following encryption type specification will be used by MIT Kerberos
# if uncommented.  In general, the defaults in the MIT Kerberos code are
# correct and overriding these specifications only serves to disable new
# encryption types as they are added, creating interoperability problems.
#
# The only time when you might need to uncomment these lines and change
# the enctypes is if you have local software that will break on ticket
# caches containing ticket encryption types it doesn't know about (such as
# old versions of Sun Java).

#       default_tgs_enctypes = des3-hmac-sha1
#       default_tkt_enctypes = des3-hmac-sha1
#       permitted_enctypes = des3-hmac-sha1

# The following libdefaults parameters are only for Heimdal Kerberos.
        fcc-mit-ticketflags = true

[realms]
        KB.EDU = {
                kdc = server.kb.edu
                admin_server = server.kb.edu
                default_domain = kb.edu
        }

[domain_realm]
        .kb.edu = KB.EDU
        kb.edu = KB.EDU

[appdefaults]
        pam = {
                debug = false
                ticket_lifetime = 36000
                renew_lifetime = 36000
                forwardable = true
                krb4_convert = false
        }

```

Создайте новый принципал машины для этого хоста. Мы запускаем команду **kadmin** от имени суперпользователя, чтобы можно было записать результирующий файл `/etc/krb5.keytab`. Иначе мы получим не очень понятную ошибку `No such file or directory while adding key to keytab`.&#x20;

Мы так же должны использовать модификатор `-p`, чтобы дать понять команде **kadmin**, от имени какого принципала мы хотим подключиться. Если мы его не зададим, то получим ошибку `Client not found in Kerberos database while initializing kadmin interface`, потому что не создавали принципал **root/admin@KB.EDU**.&#x20;

{% hint style="warning" %}
Не создавайте этот принципал! Мы хотим знать, кто подключается с правами администратора (пользователи с префиксом **/admin**). Если создадим — не сможем различать администраторов между собой.
{% endhint %}

<pre class="language-bash"><code class="lang-bash"><strong>root@client:~# kadmin -p mpyrev/admin@KB.EDU
</strong>Authenticating as principal mpyrev/admin@KB.EDU with password.
Password for mpyrev/admin@KB.EDU:

<strong>kadmin:  addprinc -randkey host/client.kb.edu@KB.EDU
</strong>No policy specified for host/client.kb.edu@KB.EDU; defaulting to no policy
Principal "host/client.kb.edu@KB.EDU" created.
<strong>kadmin:  ktadd host/client.kb.edu@KB.EDU
</strong>Entry for principal host/client.kb.edu@KB.EDU with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/client.kb.edu@KB.EDU with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/client.kb.edu@KB.EDU with kvno 2, encryption type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/client.kb.edu@KB.EDU with kvno 2, encryption type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
<strong>kadmin:  exit
</strong></code></pre>

Эта команда создала файл `/etc/krb5.keytab`.

Теперь мы можем отредактировать `/etc/ssh/sshd_config` и включить аутентификацию Kerberos. Не забудьте добавить **test.group** в директиву `AllowGroups`. Иначе мы не сможем протестировать конфигурацию и увидим ошибку `User test.user from server.kb.edu not allowed because none of user's groups are listed in AllowGroups` в файле `/var/log/auth.log`.

<pre class="language-systemd"><code class="lang-systemd">#	$OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
AddressFamily inet
AllowGroups sysadmin test.group
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
SyslogFacility AUTHPRIV
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
#ChallengeResponseAuthentication no
<strong>ChallengeResponseAuthentication yes
</strong>
# Kerberos options
#KerberosAuthentication no
<strong>KerberosAuthentication yes
</strong><strong>#KerberosOrLocalPasswd yes
</strong><strong>#KerberosTicketCleanup yes
</strong>#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
<strong>GSSAPIAuthentication yes
</strong><strong>#GSSAPICleanupCredentials yes
</strong>#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
<strong>UsePAM yes
</strong>
#AllowAgentForwarding yes
#AllowTcpForwarding yes
AllowTcpForwarding no
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
ClientAliveInterval 120
ClientAliveCountMax 2
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none
<strong>Banner /etc/issue
</strong>
# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem	sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server


</code></pre>

Перезапустим демон:

```bash
root@client:~# service ssh restart
```

Запустите на клиентской рабочей станции отображение файла журнала:

```bash
root@client:~# tail -F /var/log/auth.log
```

А пока вернёмся на сервер.

Где работаем: **server**

Создадим принципал пользователя **test.user**:

<pre class="language-bash"><code class="lang-bash"><strong>root@server:~# kadmin -p mpyrev/admin@KB.EDU
</strong>Authenticating as principal mpyrev/admin@KB.EDU with password.
Password for mpyrev/admin@KB.EDU: 
<strong>kadmin:  addprinc test.user@KB.EDU
</strong>WARNING: no policy specified for test.user@KB.EDU; defaulting to no policy
Enter password for principal "test.user@KB.EDU": 
Re-enter password for principal "test.user@KB.EDU": 
Principal "test.user@KB.EDU" created.
<strong>kadmin:  exit
</strong></code></pre>

Возьмём билет Kerberos у **test.user** и прикинемся этим пользователем. Сначала мы уничтожим свои собственные билеты (если они есть), используя **kdestroy**, затем возьмём билет **test.user** с помощью **kinit**, а в итоге проверим, что он у нас есть с помощью **klist**:

```bash
root@server:~# kdestroy
kdestroy: No credentials cache found while destroying cache
root@server:~# kinit -p test.user@KB.EDU
Password for test.user@KB.EDU: 
root@server:~# klist 
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test.user@KB.EDU

Valid starting       Expires              Service principal
29.04.2023 16:28:10  30.04.2023 16:28:10  krbtgt/KB.EDU@KB.EDU
        renew until 29.04.2023 16:28:10

```

Теперь попробуем авторизоваться на клиенте без пароля, используя этот билет:

```bash
root@server:~# ssh test.user@client.kb.edu
The authenticity of host 'client.kb.edu (192.168.3.140)' can't be established.
ECDSA key fingerprint is SHA256:mMV0STktM2g+B1pAdJjKacLjXj3c/qjW3uJR6jj/4No.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'client.kb.edu,192.168.3.140' (ECDSA) to the list of known hosts.
Debian GNU/Linux 11 \n \l

Linux client 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Apr 29 12:36:37 2023 from 192.168.3.2
test.user@client:~$

```

Где работаем: **client**

В открытом нами журнале на клиенте `/var/log/auth.log` мы должны увидеть что-то подобное:

```systemd
Apr 29 15:00:55 client sshd[2170]: Authorized to test.user, krb5 principal test.user@KB.EDU (krb5_kuserok)
Apr 29 15:00:55 client sshd[2170]: Accepted gssapi-with-mic for test.user from 192.168.3.2 port 45995 ssh2
Apr 29 15:00:55 client sshd[2170]: pam_unix(sshd:session): session opened for user test.user by (uid=0)
```

Успех! Переходим к следующей цели.

#### Аутентификация OpenLDAP с помощью SASL GSSAPI <a href="#s2.2" id="s2.2"></a>

Где работаем: **server**

Для настройки [аутентификации SASL GSSAPI](http://pro-ldap.ru/tr/admin24/sasl.html#SASL%20Authentication) мы должны изменить конфигурацию сервера OpenLDAP таким образом, чтобы он знал о существовании нашей области Kerberos. После этого мы можем настроить клиентов.

Подключимся к KDC и создадим новый принципал. Мы по-прежнему запускаем **kadmin** от имени суперпользователя, потому что хотим создать новый набор ключей в файле `/etc/ldap/krb5.keytab`, чтобы наш демон **slapd** имел свой набор.

<pre class="language-bash"><code class="lang-bash">root@server:~# kadmin -p mpyrev/admin@KB.EDU
Authenticating as principal mpyrev/admin@KB.EDU with password.
Password for mpyrev/admin@KB.EDU:
<strong>kadmin:  addprinc -randkey ldap/server.kb.edu@KB.EDU
</strong>No policy specified for ldap/server.kb.edu@KB.EDU; defaulting to no policy
Principal "ldap/server.kb.edu@KB.EDU" created.
<strong>kadmin:  ktadd -k /etc/ldap/krb5.keytab ldap/server.kb.edu@KB.EDU
</strong>Entry for principal ldap/server.kb.edu@KB.EDU with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/etc/ldap/krb5.keytab.
Entry for principal ldap/server.kb.edu@KB.EDU with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab WRFILE:/etc/ldap/krb5.keytab.
Entry for principal ldap/server.kb.edu@KB.EDU with kvno 2, encryption type des3-cbc-sha1 added to keytab WRFILE:/etc/ldap/krb5.keytab.
Entry for principal ldap/server.kb.edu@KB.EDU with kvno 2, encryption type arcfour-hmac added to keytab WRFILE:/etc/ldap/krb5.keytab.
</code></pre>

Поменяем права доступа на этот файл, чтобы демон **slapd** смог его читать:

```bash
root@server:~# chown root:openldap /etc/ldap/krb5.keytab
root@server:~# chmod 640 /etc/ldap/krb5.keytab
```

Создадим LDIF-файл `sasl.ldif` с директивами SASL:

```systemd
dn: cn=config
changetype: modify
add: olcSaslSecProps
olcSaslSecProps: noanonymous,noplain
-
add: olcSaslHost
olcSaslHost: server.kb.edu
-
add: olcSaslRealm
olcSaslRealm: KB.EDU
```

И загрузим его в базу данных службы каталогов:

```bash
root@server:~# ldapadd -QY EXTERNAL -H ldapi:/// -f sasl.ldif
modifying entry "cn=config"

```

Сейчас проверим, загрузил ли демон **slapd** нашу конфигурацию SASL:

```bash
root@server:~# ldapsearch -QLLLY EXTERNAL -H ldapi:/// -b cn=config -s base | grep -i sasl
olcSaslSecProps: noanonymous,noplain
olcSaslHost: server.kb.edu
olcSaslRealm: KB.EDU
```

Хорошо! Теперь нам нужно добавить параметр `KRB5_KTNAME` в файл `/etc/default/slapd`:

<pre class="language-systemd"><code class="lang-systemd">SLAPD_CONF=
SLAPD_USER="openldap" 
SLAPD_GROUP="openldap"
SLAPD_PIDFILE=
SLAPD_SERVICES="ldap:/// ldapi:///"
SLAPD_SENTINEL_FILE=/etc/ldap/noslapd
SLAPD_OPTIONS="-4"
<strong>export KRB5_KTNAME=/etc/ldap/krb5.keytab
</strong></code></pre>

Для того, чтобы изменения вступили в силу, нам нужно перезапустить демон **slapd**:

```bash
root@server:~# service slapd restart
```

Демон **slapd** снова запущен — мы можем переходить к настройке клиента.

Где работаем: **client**

Получим билет от KDC:

```bash
mpyrev@client:~$ kdestroy
mpyrev@client:~$ kinit -p mpyrev/admin
mpyrev@client:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: mpyrev/admin@KB.EDU
 
Valid starting       Expires              Service principal
29.04.2023 14:16:24  30.04.2023 14:16:24  krbtgt/KB.EDU@KB.EDU
```

Все настройки для доступа на сервер у нас уже внесены в файл `/etc/ldap/ldap.conf`. Проверим, можем ли мы сделать запрос к LDAP-серверу:

```bash
mpyrev@client:~$ ldapwhoami
SASL/GSSAPI authentication started
SASL username: mpyrev/admin@KB.EDU
SASL SSF: 256
SASL data security layer installed.
dn:uid=mpyrev/admin,cn=kb.edu,cn=gssapi,cn=auth
```

Отлично. Теперь попробуем зайти с хостовой машины на **client** под пользователем test.user и поменять ему пароль.

```
login as: test.user
Pre-authentication banner message from server:
| Debian GNU/Linux 11 \n \l
|
End of banner message from server
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
Linux client 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Apr 29 16:29:21 2023 from 192.168.3.139
test.user@client:~$ passwd
Current Kerberos password:
Enter new Kerberos password:
Retype new Kerberos password:
passwd: пароль успешно обновлён
test.user@client:~$

```

Процесс обновления пароля изменился. Если мы попытаемся запустить команду **passwd** от имени пользователя, которого нет в локальном файле `/etc/passwd`, пароль будет изменен с использованием механизмов Kerberos.

Вот и всё! На этом танцы с бубном у Kerberos закончились. :blush:

За основу была взята статья: [https://www.opennet.ru/docs/RUS/openldap\_ubuntu](https://www.opennet.ru/docs/RUS/openldap\_ubuntu)

## 3.3         Практическое занятие. Настройка AppArmor

### Подготовка окружения

Клонировать код web сервера из репозитория на **server**:

```bash
mpyrev@server:~$ git clone https://github.com/yurichernyshov/web_server_for_OWASP.git
```

Cоздать виртуальное окружение, чтобы символическая ссылка заменилась на копию интерпретатора

```bash
mpyrev@server:~$ python3 –m venv --copies venv
```

Установить все необходимые пакеты в окружение

```bash
mpyrev@server:~$ source venv/bin/activate
(venv) mpyrev@server:~$ pip install wheel
(venv) mpyrev@server:~$ pip install –r web_server_for_OWASP/requirements.txt
```

Убедиться, что все пакеты установились успешно

### Демонстрация уязвимостей web сервера

Запустить web-сервер от имени текущего пользователя

```bash
(venv) mpyrev@server:~$ python3 web_server_for_OWASP/app.py 5555
```

Убедиться, что сервер успешно запустился

Сформировать URLEncoded строку (например, [https://www.urlencoder.org/](https://www.urlencoder.org/)) для кода

```bash
os.system("sudo cat /etc/shadow > ./shadow.bak")
```

{% hint style="info" %}
Выполнение этого кода создает копию файла `/etc/shadow` в рабочем каталоге сервере (откуда его запустили)
{% endhint %}

Перейти по URL `<IP адрес сервера>/command?command=<URLencoded строка кода>`. Убедиться, что, что команда успешно выполнена (отображается `result 0`).

Сформировать URLEncoded строку для кода

```java
open("shadow.bak",'r').readlines()
```

Перейти по URL `<IP адрес сервера>/command?command=<URLencoded строка кода>`, убедиться, что отображается содержимое файла `/etc/shadow`

### Настройка безопасности сервера

Создать специальную учетную запись для web-сервера с помощью команды:

```bash
mpyrev@server:~$ sudo useradd -d `pwd` -M -r -s /usr/sbin/nologin flask-run
```

{% hint style="info" %}
Рекомендуется перед выполнением команды перейти в каталог, содержащий файл `app.py`
{% endhint %}

Значения флагов команд:

* создается учетная запись без полноценного домашнего каталога (с помощью опции `–d` задается явный домашний каталог, в данном случае совпадающий с текущим каталогом, опция `–M` говорит, что каталог не нужно инициализировать);
* создается системная учетная запись (т.е. uid менее 1000 – чистое украшательство), что задается опцией `–r` ;
* у учетной записи отсутствует пароль и шелл (опция `–s` задает специальный «шелл» `/usr/sbin/nologin`) – это делает невозможным обычный вход от имени этой учетной записи (что локальный, что по SSH).

Сменить владельца каталога с файлами web-сервера:

```bash
mpyrev@server:~$ sudo chown -R flask-run:flask-run web_server_for_OWASP
```

Показать, что просто так перейти в сеанс учетной записи невозможно:

```bash
mpyrev@server:~$ sudo su flask-run # выполнится с ошибкой
mpyrev@server:~$ sudo su –l flask-run –s /bin/bash # позволит перейти в сеанс flask-run
```

Запустить web-сервер из под учетной записи `flask-run` (в сеанс перешли в предыдущей демонстрации), команды предполагают, что текущий каталог это `/home/mpyrev`, а не `web_server_for_OWASP`:

```bash
mpyrev@server:~$ source venv/bin/activate
(venv) mpyrev@server:~$ python3 web_server_for_OWASP/app.py 5555
```

Повторно пробудем прочитать файл `/etc/shadow` ([демонстрация](3-linux.md#demonstraciya-uyazvimostei-web-servera)), убеждаемся, что прочитать не удалось.

Для удобства управления сервером создаем скрипты для запуска и останова сервера:

Скрипт `start.sh`:

```bash
source /home/mpyrev/venv/bin/activate
SCRIPT_PATH="/home/mpyrev/web_server_for_OWASP"
start-stop-daemon -S -b --make-pidfile -u flask-run --pidfile /tmp/flask-run.pid -d $SCRIPT_PATH -a `which python3` -c flask-run:flask-run -- $SCRIPT_PATH/app.py 5555
```

Скрипт `stop.sh`:

```bash
start-stop-daemon -K --pidfile /tmp/flask-run.pid
```

{% hint style="info" %}
start-stop-daemon – утилита, которая берет на себя рутину по «демонизации» любых процессов, поэтому удобно ее использовать для запуска различных серверов.&#x20;
{% endhint %}

Пояснения по синтаксису:

`-S` – команда запуска

`-K` – команда останова

`-b` – запуск в режиме демона (освобождение консоли)

`--make-pidfile` – указывает что `start-stop-daemon` самостоятельно позаботиться о создании PID файла

`--pidfile` – указывает имя PID файла, которое надо использовать

`-u flask-run` – способ проверки, что демон еще не запущен (в данном случае по имени пользователя – если уже будет процесс `python3` от имени `flask-run`, то `start-stop-daemon` ничего не запустит)

`-d` – указывает на домашний каталог демона (обязательно, иначе сервер не найдет свои файлы – БД, шаблоны и пр.)

`-a` – что, собственно, запускать (здесь сделано через подстановку результата работы утилиты which для универсальности, но можно просто задать полное имя)

`-c` – от чьего имени запускать

После `--` идут опции, которые передаются запускаемому демону

Опционально можно убедиться, что через web сервер мы прекрасно можем прочитать файл `start.sh` (да и любой другой, к которому у flask-run есть доступ на чтение)

### Помещение сервера в изолированный профиль AppArmor

Информацию про Linux Security Modules (механизм реализации альтернативной модели безопасности в Linux) можно прочитать в лекциях (презентации Лекция\_13 и Лекция\_16)

#### Создание нового профиля AppArmor

Останавливаем web сервер (`stop.sh`)

Устанавливаем утилиты для работы с профилями AppArmor:

```bash
mpyrev@server:~$ sudo apt-get install apparmor-utils
```

Запускаем интерактивное формирование профиля для сервера:

```bash
mpyrev@server:~$ sudo aa-genprof /home/mpyrev/venv/bin/python3
```

Во второй консоли запускаем web сервер (`start.sh`), выполняем все легитимные действия с сервером (`all, one, create, update, predict`).

Останавливаем web сервер (`stop.sh`).

В первой консоли (где работает `aa-genprof`) нажимаем `s` (`(S)can system log for AppArmor events`).

Отвечаем на вопросы утилиты, руководствуясь следующими соображениями:

* всегда выбираем конкретный каталог сервера и виртуального окружения (утилита будет пытаться маскировать имя пользователя в пути по умолчанию)
* используем `(G)lob` для путей, где предполагаются массовые однотипные действия с файлами (типа `site-packages, templates, dev/shm`)
* соглашаемся на инклюды системных политик
* После завершения вопросов утилиты сохраняем полученный профиль (можно предварительно его отобразить на экране)

#### Проверка профиля AppArmor

Проводим проверку профиля

```bash
mpyrev@server:~$ sudo aa-complain /home/ubuntu/venv/bin/python3
```

Во второй консоли также запускаем web сервер и снова выполняем все легитимные действия. Можно сделать что-то запрещенное, убедиться, что все получилось, но запомнить, что сделали, чтобы не добавить потом это действие в разрешения профиля.

{% hint style="info" %}
Если сервер перестал работать, возможно скорректировать профиль по пути `/etc/apparmor.d` в данной папке возможно найти профиль `home.mpyrev.venv.bin.python3` и проверить что запрещено (строка начинается с`deny`)

После сохранение профиля необходимо перезапустить сервис Apparmor командой&#x20;

`systemctl restart apparmor`
{% endhint %}

Останавливаем web-сервер во второй консоли

Запускаем утилиту сканирования журнала:

```bash
mpyrev@server:~$ sudo aa-logprof
```

Анализируем вывод утилиты (аналогичен `aa-genprof`). Если видно, что запрашиваются какие-то нужные права – подтверждаем. Всякие сомнительные – либо `Ignore`, либо `Deny` (отличие в том, что `Deny` больше не попадет в логи вообще, а так оба варианта запрещают действие, так как политика AppArmor строится на принципе запрещено все, что не разрешено).

#### Боевой запуск профиля AppArmor

Переводим профиль в боевой режим:

```bash
mpyrev@server:~$ sudo aa-enforce /home/ubuntu/venv/bin/python3
```

Запускаем web сервер, проводим проверку легитимных действий (должно все получиться)

Пробуем сделать что-то нелегитимное (например, прочитать `start.sh`) – убеждаемся, что больше это не получается.

{% hint style="info" %}
Прочитать сам `app.py` получится, так как мы разрешили это в профиле (иначе скрипт не запустился бы)
{% endhint %}

### Настройка контроля целостности

Устанавливаем пакет AIDE

```bash
mpyrev@server:~$ sudo apt-get install aide –y
```

Прячем все стандартные политики AIDE, чтобы они не использовались далее:

```bash
mpyrev@server:~$ sudo mkdir /etc/aide/aide.conf.d/aide_default_rules.d
mpyrev@server:~$ sudo mv /etc/aide/aide.conf.d/* /etc/aide/aide.conf.d/aide_default_rules.d
```

Создаем новый файл с правилами для нашего сервера:

```bash
mpyrev@server:~$ sudo nano /etc/aide/aide.conf.d/10_aide_my_webserverubuntu
```

Содержимое файла:

```cmake
/home/mpyrev/venv Full
/home/mpyrev/web_server_for_OWASP Full
!/home/mpyrev/web_server_for_OWASP/database.db
!/home/mpyrev/web_server_for_OWASP/database.db-journal
```

Пояснение по содержимому: первые две сроки задают каталоги для контроля целостности и способ контроля. `Full` – это макрос, который говорит контролировать все, использовать все поддерживаемые контрольные суммы. Кому интересно, может найти определение макроса в `/etc/aide/aide.conf` (там сильная вложенность макросов).

Последние две строки задают исключения для контроля целостности.

Создаем первичный снимок каталогов:

```bash
mpyrev@server:~$ sudo aide --init –c /etc/aide/aide.conf
```

Выполняем тестовую проверку:

```bash
mpyrev@server:~$ sudo aide --check –c /etc/aide/aide.conf
```

В выводе должна присутствовать строка:&#x20;

```cmake
AIDE found NO differences between database and filesystem. Looks okay!!
```

Проверяем работу AIDE: модифицируем файл `app.py` (произвольным образом)

Запускаем проверку

```bash
mpyrev@server:~$ sudo aide --check –c /etc/aide/aide.conf
```

Теперь в выводе должны присутствовать строки вида:

```cmake
Changed entries:
---------------------------------------------------
d =.... mc.. .. .: /home/ubuntu/web_server_for_OWASP
f >.... mci.C.. .: /home/ubuntu/web_server_for_OWASP/app.py
```

При необходимости обновляем конфигурацию.

## 3.4         Практическое занятие. Настройка окружения SELinux на примере LAMP-сервера

### Подготовка

Установим стандартный комплект софта для LAMP:

```bash
[root@lamp ~]# yum install -y httpd mariadb-server php-fpm php-mysqlnd
```

Минимально настроим софт:

<details>

<summary>/etc/php-fpm.d/www.conf</summary>

```cmake
[www]
listen = 127.0.0.1:9009
user = apache
group = apache
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
pm.status_path = /status
request_terminate_timeout = 10s
request_slowlog_timeout = 1s
slowlog = /var/log/php-fpm/www-slow.log
security.limit_extensions = .php
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_admin_value[memory_limit] = 128M
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session
```

</details>

<details>

<summary>/etc/httpd/conf.d/userdir.conf</summary>

<pre class="language-cmake"><code class="lang-cmake">&#x3C;IfModule mod_userdir.c>
<strong>    UserDir enabled
</strong><strong>    UserDir www
</strong>&#x3C;/IfModule>
&#x3C;Directory "/home/*/www">
<strong>    AllowOverride FileInfo AuthConfig Limit Indexes
</strong>    Options MultiViews Indexes
    Require method GET POST OPTIONS
    DirectoryIndex index.html index.htm index.php
    &#x3C;FilesMatch "\.php$">
        &#x3C;If "-f %{REQUEST_FILENAME}">
            SetHandler "proxy:fcgi://127.0.0.1:9009"
        &#x3C;/If>
<strong>    &#x3C;/FilesMatch>
</strong>&#x3C;/Directory>
</code></pre>

</details>

Отключим временно SELinux:

```bash
[root@lamp ~]# setenforce 0
```

Запустим все необходимые сервисы:

```bash
[root@lamp ~]# systemctl enable httpd mariadb php-fpm
[root@lamp ~]# systemctl start httpd mariadb php-fpm
```

Включим обратно SELinux:

```bash
[root@lamp ~]# setenforce 1
```

Добавим какого-нибудь пользователя, например `phpbb`:

```bash
[root@lamp ~]# useradd -Z user_u -m -g apache phpbb
[root@lamp ~]# chmod 750 /home/phpbb
```

И создадим простой тестовый файл с `phpinfo()`:

```bash
[phpbb@lamp ~]$ mkdir www
[phpbb@lamp ~]$ echo "<?php phpinfo(); ?>" > www/info.php
```

Перейдем по ссылке…

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

… и получим именно то, что получают все :blush:

#### Разбираемся с ошибками

В отличие от других мануалов, где следующим шагом идет «отключите SELinux», мы сейчас узнаем, почему так получилось и что можно сделать.\
\
Для начала — установим консольные утилиты для управления политиками SELinux:

```bash
[root@lamp ~]# yum install -y policycoreutils-python-utils policycoreutils-newrole policycoreutils-restorecond setools-console
```

А потом — включим нужные нам модули ( командой `semodule` ):

```bash
[root@lamp ~]# semodule -e apache
[root@lamp ~]# semodule -e mysql
```

Давайте посмотрим, с какими именно проблемами столкнулся apache при открытии этой страницы?

<details>

<summary>audit2allow -lb -t httpd_t</summary>

```cmake
#============= httpd_t ==============

#!!!! This avc can be allowed using one of the these booleans:
#     httpd_enable_homedirs, httpd_unified
allow httpd_t httpd_user_content_t:file getattr;
```

</details>

{% hint style="info" %}
Все верно: папка _www_ (а так-же папки _web_ и _public\_html_) внутри домашней директории пользователя автоматически получает тип **`httpd_user_content_t`**, что и указано в правилах:
{% endhint %}

<details>

<summary>sesearch -T -s user_t -c dir -dt | grep user_home_dir_t | grep httpd_user_content_t</summary>

```cmake
type_transition user_t user_home_dir_t:dir httpd_user_content_t public_html;
type_transition user_t user_home_dir_t:dir httpd_user_content_t web;
type_transition user_t user_home_dir_t:dir httpd_user_content_t www;
```

</details>

{% hint style="info" %}
Лечение указано в выводе `audit2allow`, установка переменных выполняется командой **`setsebool`**(или **`semanage boolean`**).
{% endhint %}

```bash
[root@lamp ~]# cd /etc/httpd
[root@lamp httpd]# setsebool -P httpd_enable_homedirs=1
```

Обновляем страницу и получаем:

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Смотрим логи:

<details>

<summary>/var/log/httpd/error_log</summary>

```cmake
[Sun May 28 22:28:07.097784 2023] [proxy:error] [pid 37774:tid 37917] (13)Permission denied: AH00957: FCGI: attempt to connect to 127.0.0.1:9009 (*:80) failed
[Sun May 28 22:28:07.097863 2023] [proxy_fcgi:error] [pid 37774:tid 37917] [client 127.0.0.1:58310] AH01079: failed to make connection to backend: 127.0.0.1
```

</details>

Все ясно: httpd не может коннектиться куда попало, httpd может ходить только куда нужно. Это логично: если веб-сервер вдруг соединяется по ssh, то явно происходит что-то странное.\
Давайте посмотрим, куда веб-серверу ходить можно?

<details>

<summary>sesearch -A -s httpd_t -c tcp_socket -p name_connect</summary>

```cmake
allow daemon auth_port_t:tcp_socket name_connect; [ daemons_use_tcp_wrapper ]:True
allow httpd_t cobbler_port_t:tcp_socket name_connect; [ httpd_can_network_connect_cobbler ]:True
allow httpd_t ephemeral_port_type:tcp_socket name_connect; [ httpd_can_connect_ftp ]:True
allow httpd_t ephemeral_port_type:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t ephemeral_port_type:tcp_socket name_connect; [ httpd_use_openstack ]:True
allow httpd_t ftp_port_t:tcp_socket name_connect; [ httpd_can_connect_ftp ]:True
allow httpd_t ftp_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t gds_db_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t gopher_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t http_cache_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t http_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t http_port_t:tcp_socket name_connect; [ httpd_graceful_shutdown ]:True
allow httpd_t ldap_port_t:tcp_socket name_connect; [ httpd_can_connect_ldap ]:True
allow httpd_t memcache_port_t:tcp_socket name_connect; [ httpd_can_network_memcache ]:True
allow httpd_t memcache_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t mongod_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t mssql_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t mysqld_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t mythtv_port_t:tcp_socket name_connect; [ httpd_can_connect_mythtv ]:True
allow httpd_t oracle_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t osapi_compute_port_t:tcp_socket name_connect; [ httpd_use_openstack ]:True
allow httpd_t pop_port_t:tcp_socket name_connect; [ httpd_can_sendmail ]:True
allow httpd_t port_type:tcp_socket name_connect; [ httpd_can_network_connect ]:True
allow httpd_t postgresql_port_t:tcp_socket name_connect; [ httpd_can_network_connect_db ]:True
allow httpd_t smtp_port_t:tcp_socket name_connect; [ httpd_can_sendmail ]:True
allow httpd_t squid_port_t:tcp_socket name_connect; [ httpd_can_network_relay ]:True
allow httpd_t tcs_port_t:tcp_socket name_connect; [ httpd_use_opencryptoki ]:True
allow httpd_t zabbix_port_t:tcp_socket name_connect; [ httpd_can_connect_zabbix ]:True
allow nsswitch_domain dns_port_t:tcp_socket { name_connect recv_msg send_msg };
allow nsswitch_domain dnssec_port_t:tcp_socket name_connect;
allow nsswitch_domain ephemeral_port_t:tcp_socket name_connect; [ nis_enabled ]:True
allow nsswitch_domain kerberos_port_t:tcp_socket name_connect; [ kerberos_enabled ]:True
allow nsswitch_domain ldap_port_t:tcp_socket name_connect; [ authlogin_nsswitch_use_ldap ]:True
allow nsswitch_domain ocsp_port_t:tcp_socket name_connect; [ kerberos_enabled ]:True
allow nsswitch_domain port_t:tcp_socket name_connect; [ nis_enabled ]:True
allow nsswitch_domain reserved_port_type:tcp_socket name_connect; [ nis_enabled ]:True
allow nsswitch_domain unreserved_port_t:tcp_socket name_connect; [ nis_enabled ]:True

```

</details>

В квадратных скобках указаны переменные, которые отвечают за работу этого правила.

Итого: нужно добавить порт 9009 в один из типов, к которым разрешен коннект, а затем установить переменную `httpd_can_network_relay` в 1.

Новый порт добавляется при помощи команды `semanage port`:

```
[root@lamp httpd]# semanage port -a -t http_cache_port_t -p tcp 9009
[root@lamp httpd]# setsebool -P httpd_can_network_relay=1
```

Обновляем страницу и видим:

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Что-то посложнее

Давайте теперь усложним задачу и поставим phpbb на этот хост.

```bash
[phpbb@lamp ~]$ curl https://download.phpbb.com/pub/release/3.3/3.3.10/phpBB-3.3.10.zip -O
[phpbb@lamp ~]$ unzip phpBB-3.3.10.zip
[phpbb@lamp ~]$ mv phpBB3/* www/
[phpbb@lamp ~]$ restorecon -R www/
```

Создадим базу данных:

```bash
[root@lamp www]# mysql -u root
```

<pre class="language-sql"><code class="lang-sql"><strong>MariaDB [(none)]> CREATE USER 'admin'@'localhost' IDENTIFIED BY 'P@ssw0rd';
</strong>MariaDB [(none)]> CREATE DATABASE phpbb;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON phpbb.* TO 'admin'@'localhost';
</code></pre>

Пробуем зайти в инсталляшку phpbb:

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Почему так? Потому что httpd не может изменять пользовательские данные. Давайте узнаем, какие же он изменять может?

<details>

<summary>sesearch -A -s httpd_t -p write -t user -rt</summary>

```cmake
allow httpd_t httpd_user_ra_content_t:dir { add_name write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_ra_content_t:dir { add_name write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:dir { add_name create ioctl link lock read remove_name rename reparent rmdir setattr unlink watch watch_reads write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:dir { add_name ioctl lock read remove_name write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:dir { add_name ioctl lock read remove_name write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:dir { add_name ioctl lock read remove_name write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:file { append create getattr ioctl link lock open read rename setattr unlink watch watch_reads write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:lnk_file { append create getattr ioctl link lock read rename setattr unlink watch watch_reads write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpd_user_rw_content_t:sock_file { append getattr open read write }; [ httpd_builtin_scripting ]:True
allow httpd_t httpdcontent:dir { add_name create ioctl link lock read remove_name rename reparent rmdir setattr unlink watch watch_reads write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:file { append create getattr ioctl link lock open read rename setattr unlink watch watch_reads write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:lnk_file { append create getattr ioctl link lock read rename setattr unlink watch watch_reads write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t user_devpts_t:chr_file { append getattr ioctl lock read write }; [ httpd_tty_comm ]:True
allow httpd_t user_tmp_t:dir { add_name ioctl lock read remove_name write };
allow httpd_t user_tmp_t:file { map write };
allow httpd_t user_tty_device_t:chr_file { append getattr ioctl lock read write }; [ httpd_tty_comm ]:True
```

</details>

Устанавливаем `php-xml` и `php-mbstring` включаем `httpd_builtin_scripting` и назначаем контекст `httpd_user_rw_content_t` на указанные файлы и папки (командой `chcon`):

<pre class="language-bash"><code class="lang-bash">[root@lamp httpd]# cd /etc/httpd/
[root@lamp httpd]# setsebool -P httpd_builtin_scripting=1
<strong>[root@lamp httpd]# exit
</strong>[phpbb@lamp ~]$ cd www/
[phpbb@lamp www]$ chmod 660 config.php
[phpbb@lamp www]$ chcon -t httpd_user_rw_content_t cache/ store/ files/ config.php images/avatars/upload/ phpbb/filesystem/filesystem.php 
</code></pre>

Получаем:

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Устанавливаем phpBB дальше, удаляем install и получаем работающий форум:

<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Меняем контекст конфига обратно:

```bash
[phpbb@lamp www]$ chcon -t httpd_user_content_t config.php
```

Наслаждаемся безопасным форумом :blush:

<details>

<summary>Небольшой cheatsheet</summary>



Команды.

* semodule — управляет списком модулей
* sestatus — текущий статус SELinux
* setenforce 1/0 — включить/выключить enforcing
* audit2allow — утилита для генерации правил ( и для подсказок )
* sesearch — утилита для поиска правил в политике
* seinfo — показывает информацию о типах, ролях, атрибутах итд
* semanage — позволяет вносить изменения в политики
* chcon — позволяет менять контекст на ФС
* restorecon — востанавливает контекст по-умолчанию
* setsebool — устанавливает переменную в on/off. С -P — пишет на диск
* getsebool — получает переменную. -a — посмотреть все

Изменения политики

* semanage port -a/-d -t httpd\_port\_t -p tcp 8044 — добавить/удалить номер порта к контексту
* semanage fcontext -a/-d -t httpd\_cache\_t "/srv/http/cache(/.\*)?" — добавить/удалить контекст для этой маски
* semanage permissive -a/-d httpd\_t — включить/выключить режим permissive для httpd\_

Аргументы к командам

* id -Z — показывает контекст текущего пользователя
* ls -Z — показывает контекст файлов
* ps -Z — показывает контекст процессов
* netstat -Z — показывает контекст соединений
* usermod/useradd -Z связать пользователя с SELinux-пользователем
* ausearch -m AVC — показывает нарушения политик

</details>

Источники для дополнительного ознакомления:

{% embed url="https://habr.com/ru/articles/320100/" %}

{% embed url="https://habr.com/ru/articles/322904/" %}

[^1]: Обозначение SUID

[^2]: Обозначение, что бит на исполнение (x) отсутствует

[^3]: Обозначение sticky bit

[^4]: Обозначение, что бит на исполнение (x) отсутствует

[^5]: Может быть выбран студентом самостоятельно

[^6]: Мы ранее не настраивали пароль для персонального пользователя, рекомендуется предварительно воспользоваться утилитой ldappasswd&#x20;

[^7]: Пароль пользователя nssproxy

[^8]: Мы ранее не настраивали пароль для персонального пользователя, рекомендуется предварительно воспользоваться утилитой ldappasswd&#x20;

[^9]: Места на которые стоит обратить внимание выделены

[^10]: Как пробросить 22 (ssh) порт при NAT подключении: \
    статья\
    [http://guruadmin.ru/page/razreshaem-dostup-k-virtualnoj-mashine-vmware-nat-s-drugix-kompyuterov/](http://guruadmin.ru/page/razreshaem-dostup-k-virtualnoj-mashine-vmware-nat-s-drugix-kompyuterov/)\
    видео\
    [https://youtu.be/CICf-0md2gM?t=646](https://youtu.be/CICf-0md2gM?t=646)\
    \
    При подключении по PuTTY выбираем localhost:\[порт на которые пробросили 22 от ВМ]
