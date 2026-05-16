# Домашнее задание к занятию «Защита сети» - Куприянов Ю.В.

------

### Подготовка к выполнению заданий
- `IP адрес атакующей машины: 192.168.0.107`
- `IP адрес защищающейся машины: 192.168.0.108`


1. Подготовка защищаемой системы:

- установите **Suricata**,

![Скриншот-00](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img00.png)

`Вносим настройки в файл **/etc/suricata/suricata.yaml**`

```
EXTERNAL_NET: “any” # указываем "any"

af-packet: # заменяем имя интерфейса в разделе af-packet 
interface: enp0s3

default-rule-path: /var/lib/suricata/rules/ # указываем путь к правилам Suricata
rule-files:
- suricata.rules
```

- установите **Fail2Ban**.

![Скриншот-01](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img01.png)


2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

![Скриншот-02](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img02.png)

------

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

---
### ОТВЕТ на Задание 1.

`1.1 Включаем защиту Suricata.`

```
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s3
```

`1.2 Запускаем сканирование на Kali (атакующей) машине.`

![Скриншот-11](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img11.png)

`1.3 Просматриваем отчет Suricata.`

![Скриншот-12](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img12.png)

`1.4 Вывод`

#### С помощью утилиты `nmap` был проведён комплексный анализ хоста с IP-адресом `192.168.0.108`: использовались различные методы сканирования — `ACK, SYN, connect`, а также определение версий запущенных сервисов. На защищаемой машине система обнаружения вторжений Suricata зафиксировала соответствующие сетевые активности, распознав их как попытки сканирования портов.

------

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.

---
### ОТВЕТ на Задание 2.

`2.1 Содаём пользователя dupe-user с паролем 1234.`
```
sudo adduser dupe-user
...
...
New password: 1234
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 1234
```
![Скриншот-21](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img21.png)

`2.2 Создаём файлы users.txt и pass.txt на машиен Kali.`

```
users.txt

admin
administrator
god
andrey
superman
dupe-user
root
```

```
pass.txt

1234
god
qwerty
abcd
QJt#90_@root
god
root
```
![Скриншот-22](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img22.png)

`2.3 Тестируем Hydra при отключенной защите.`

`Успешный подбор пароля.`

![Скриншот-23](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img23.png) 

`Отображение в лог файлах попытки подбора логинов и паролей и успешная авторизация dupe-user`

![Скриншот-231](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img231.png)


`2.4 Включаем защиту SSH для Fail2Ban.`

`В файле /etc/fail2ban/jail.conf, в секции sshd установливаем enabled = true.`

![Скриншот-24](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img24.png)

`Перезапускаем сервис Fail2Ban.`

```
sudo systemctl stop fail2ban.service
```

`2.5 Запускае Hydra при включенной защите.`

![Скриншот-25](https://github.com/Yuriykup/Netology_13-03-hw/blob/main/img/img25.png)

`Вывод: После активации защиты fail2ban и настройки конфигурационного файла, атака была успешно заблокирована.`
---

*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*
