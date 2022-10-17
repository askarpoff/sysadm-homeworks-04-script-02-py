# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Никакое, т.к. в одной строка, а в другой число, ошибка  |
| Как получить для переменной `c` значение 12?  | сделать a='1', но мы получим строковую '12'  |
| Как получить для переменной `c` значение 3?  | сделать b=2, оставить а=1, получим численное значение  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
bash_command = ["cd ~/netology/sysadm-homeworks","pwd"]
directory_os = os.popen(' && '.join(bash_command)).read().replace("\n","")
print(f'Отслеживаемая директория {directory_os}')
bash_command = ["cd ~/netology/sysadm-homeworks","git status"]
result_os = os.popen(' && '.join(bash_command)).read()
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = directory_os+'/'+result.replace('\tmodified:   ', '')
        print(f'Изменился файл {prepare_result}')
```

### Вывод скрипта при запуске при тестировании:
```
debian@debian:~$ /bin/python3 /home/debian/1.py
Отслеживаемая директория /home/debian/netology/sysadm-homeworks
Изменился файл /home/debian/netology/sysadm-homeworks/test1.txt
Изменился файл /home/debian/netology/sysadm-homeworks/test2.txt
Изменился файл /home/debian/netology/sysadm-homeworks/test3.txt
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import sys
if len(sys.argv)<2:
    print('Не указан каталог');
    sys.exit(0);
bash_command = ["cd "+sys.argv[1],"pwd"]
directory_os = os.popen(' && '.join(bash_command)).read().replace("\n","")
if "cd: can't cd to" in directory_os:
    print(f'Нет такого каталога: {sys.argv[1]}');
    sys.exit(0);

print(f'Отслеживаемая директория {directory_os}')

bash_command = ["cd "+sys.argv[1],"git status"]
result_os = os.popen(' && '.join(bash_command)).read()
if len(result_os)==0:
    print(f'Это не репозиторий: {sys.argv[1]}');
    sys.exit(0);

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = directory_os+'/'+result.replace('\tmodified:   ', '')
        print(f'Изменился файл {prepare_result}')
```

### Вывод скрипта при запуске при тестировании:
```
debian@debian:~/netology/sysadm-homeworks$ /bin/python3 /home/debian/2.py /home/debian/netology/
Отслеживаемая директория /home/debian/netology
fatal: not a git repository (or any of the parent directories): .git
Это не репозиторий": /home/debian/netology/
debian@debian:~/netology/sysadm-homeworks$ /bin/python3 /home/debian/2.py /home/debian/netology/
Отслеживаемая директория /home/debian/netology
fatal: not a git repository (or any of the parent directories): .git
Это не репозиторий: /home/debian/netology/
debian@debian:~/netology/sysadm-homeworks$ /bin/python3 /home/debian/2.py ./netology/sysadm-homeworks
/bin/sh: 1: cd: can't cd to ./netology/sysadm-homeworks
Отслеживаемая директория
/bin/sh: 1: cd: can't cd to ./netology/sysadm-homeworks
Это не репозиторий: ./netology/sysadm-homeworks
debian@debian:~/netology/sysadm-homeworks$ /bin/python3 /home/debian/2.py /home/debian/netology/sysadm-homeworks/
Отслеживаемая директория /home/debian/netology/sysadm-homeworks
Изменился файл /home/debian/netology/sysadm-homeworks/test1.txt
Изменился файл /home/debian/netology/sysadm-homeworks/test2.txt
Изменился файл /home/debian/netology/sysadm-homeworks/test3.txt

```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```
