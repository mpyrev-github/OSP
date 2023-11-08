# 🎨 5       Творческая работа (Linux)

**Deadline **<mark style="color:red;">**00:00**</mark>**:00 **<mark style="color:red;">**29.04**</mark>**.2023**

Перед началом выполнения необходимо ввести команду&#x20;

```bash
export HISTTIMEFORMAT='%F %T '
```

Командой `history 5` проверить, что дата и время команд отображается.

Представьте, что вы оказались в стартапе. У вас есть только один Linux сервер для обеспечения работы маленькой компании. Придумайте несколько ролей (не менее трех) для шести (или более) пользователей. У каждого пользователя есть не менее 2 папок, как минимум с одним файлом.\
Необходимо продумать логику работы команды и права доступа каждого из сотрудников.  В ходе выполнения задания необходимо использовать все механизмы, изученные [в разделе 3.1](3-linux.md#3.1-prakticheskoe-zanyatie.-prava-dostupa.-atributy-faila.-posix-acl)

> Источники для дополнительного изучения:\
> [https://help.ubuntu.ru/wiki/access\_control\_list](https://help.ubuntu.ru/wiki/access\_control\_list)\
> [https://www.opennet.ru/docs/RUS/posixacl/posixacls5.html](https://www.opennet.ru/docs/RUS/posixacl/posixacls5.html)

В ходе выполнения задания рекомендуется сделать несколько скриншотов на основных, по вашему мнению, пунктах. После выполнения задания необходимо выполнить команду&#x20;

```bash
history > ФИО_KB_Linux1.txt
```

или&#x20;

```bash
history | grep "дата выполнения" > ФИО_KB_Linux1.txt 
```

Результатом работы станет отчет с описанием идеи разграничения доступа и файл _ФИО\_KB\_Linux1.txt_