<div align="center">
  <br>
  <img src="https://i.ibb.co/DkYw4wF/logo.png" alt="logo" border="0"><br><br>
  МИНОБРНАУКИ РОССИИ<br>
  Федеральное государственное бюджетное<br>
  образовательное учреждение высшего образования
  
#### **«МИРЭА – Российский технологический униврситет»**
  
  Лабораторная работа по дисциплине:<br>
  «Программные средства оперативно-аналитического поиска»
</div>
<br><br><br><br><br>
<div align="right">
  <br>
  Выполнил:<br>
  Студент 4 курса<br>
  Специальности 10.05.05<br>
  Группа ББСО-02-16<br>
  Антипов К.О<br>
  Проверил:<br>
  Захарчук И.И.<br>
</div>
<br><br><br><br><br><br><br>

<p align="center">
  Москва 2020
</p>
<br><br><br>

<div align="center">

#### **Лабораторная работа № 3**

</div>

<p>

#### **Цель работы**

Исследование возможностей автоматизации сбора данных о доменах
</p>

<p>

#### **Исходные данные**

| Наименование устройства  | Программа виртуализации   | Операционная система  |
| ------------------------ | ------------------------- | --------------------- |
| XIAOMI MI NOTEBOOK PRO   | VMware Workstation Pro 15 | Ubuntu 18.04          |
</p>

<p>

#### **Исследуемые домены**

1. [TechRadar](https://www.techradar.com)
2. [WIRED](https://www.wired.com)
3. [Lifewire](https://www.lifewire.com)
4. [PCmag](https://www.pcmag.com)
5. [TechCrunch](https://www.techcrunch.com)
6. [Ars Technica](https://www.arstechnica.com)
7. [PCWorld](https://www.pcworld.com)
8. [Laptop Mag](https://www.laptopmag.com)
9. [Slashdot](https://www.slashdot.org)
10. [Computerworld](https://www.computerworld.com)
11. [The Register](https://www.theregister.co.uk)
12. [CIO](https://www.cio.com)
13. [Tehmeme](https://www.techmeme.com)
14. [ExtremeTech](https://www.extremetech.com)
15. [Techdirt](https://www.techdirt.com)
</p>

<p>

#### **Используемые утилиты**

1. Утилиты 
    * [IPWhois](https://github.com/secynic/ipwhois)<br>
    * [BuiltWith](https://bitbucket.org/richardpenman/builtwith/src/default/)<br>
    * [Shodan](shodan.io)<br>
2. [Rstudio IDE](https://rstudio.com/)
</p>

<p>

#### **Варианты решения задачи**

1. Собрать информацию вручную с помощью веб-браузера, инструментов whois, dig, nmap и т.д.
2. Использоавть интегрированные инструменты такие как SpiderFoot, Maltego CE, Datasploit, Recon-ng
3. Самостоятельно разработать (для образовательных целей) автоматизированное решение для сбора информации.

В данной работе используется третий вариант решения задачи.

</p>

<p>

#### **Общий план выполнения работы**

Общий ход выполнения работы

1. Написание функции/скрипта для сбора требуемой информации
2. Сбор информации по компаниям
    
</p>


#### **Разработка средства сбора информации**


```{r setup, include=FALSE, echo=TRUE}
library(reticulate)
use_python("/usr/bin/python3.6", required = T)
py_config()
```

```{python}

from ipwhois import IPWhois
from socket import socket, gethostbyname, AF_INET, SOCK_STREAM
from tabulate import tabulate
import builtwith
import shodan

def phone_clean(phone):
    try:
        just_phone = phone.strip("\n\r\t abcdefghijklmnopqrstuvwxyz:;")
    except:
        just_phone = phone
    return just_phone

short_urls = ['techradar.com', 'wired.com', 'lifewire.com', 'pcmag.com', 'techcrunch.com', 'arstechnica.com',
                  'pcworld.com', 'laptopmag.com', 'slashdot.org', 'computerworld.com', 'theregister.co.uk', 'cio.com',
                  'techmeme.com', 'extremetech.com', 'techdirt.com']
full_urls = []
for i in short_urls:
    url_element = 'https://'
    full_urls.append(str(url_element) + str(i))

information_about_all_domain = []
domain_technologies = []
chek_port = [21, 22, 80, 443]
open_ports = []
n = 0
number = 1

for i in full_urls:
    domain = i
    find_ip = gethostbyname(short_urls[n])
    who_is = IPWhois(find_ip)
    whois_results = who_is.lookup_whois()
    rdap_results = who_is.lookup_rdap(depth=1)
    ip_netblock = whois_results['nets'][0]['cidr']
    country = whois_results['nets'][0]['country']
    city = whois_results['nets'][0]['city']
    address = whois_results['nets'][0]['address']
    api = shodan.Shodan('gmKRrUgQFEmMTgeN5UWGWv7etjuScv7V')    
    info = api.host(find_ip) 
    try:
        if info['data'] is not None:
            for item in info['data']:
                if 'isp' in item and item['isp'] is not None:
                    isp = item['isp']
    except:
        isp = 'None'
                
    web_parser = builtwith.parse(i)  
    try:
        for key, value in web_parser.items():
            domain_technologies.append("{0}: {1}".format(key, *value))
    except:
        domain_technologies = 'None'
    try:
        if rdap_results['objects'] is not None:
            for a in rdap_results['objects']:
                telephone = rdap_results['objects'][a]['contact']['phone']
                if telephone is not None:
                    phone = str(phone_clean(telephone[0]['value']))
    except:
        phone = "None"

    for port_element in chek_port:
        s = socket(AF_INET, SOCK_STREAM)
        s.settimeout(0.08)
        verification_result = s.connect_ex((find_ip, port_element))
        if (verification_result == 0):
            status = 'open'
            open_ports.append(str(port_element) + " " + str(status))
        s.close()

    information_about_domain = {
        "Номер": number,
        "Домен": domain,
        "IP-адрес": find_ip,
        "IP Netblock": ip_netblock,
        "Страна": country,
        "Город" : city,
        "Адрес" : address,
        "Телефон": phone,
        "Хостинг": isp,
        "Порты": '\n'.join(open_ports),
        "Web-технологии" :'\n'.join(domain_technologies)
    }

    information_about_all_domain.append(information_about_domain)
    n += 1
    open_ports = []
    domain_technologies = []
    number += 1

print(tabulate(information_about_all_domain, headers='keys', tablefmt="pipe"))

```

|   Номер | Домен                     | IP-адрес        | IP Netblock                    | Страна   | Город         | Адрес                        | Телефон                | Хостинг                          | Порты    | Web-технологии                                         |
|--------:|:--------------------------|:----------------|:-------------------------------|:---------|:--------------|:-----------------------------|:-----------------------|:---------------------------------|:---------|:-------------------------------------------------------|
|       1 | https://techradar.com     | 185.113.25.50   | 185.113.24.0/22                | GB       |               | Quay House, The Ambury       | None                   | Future Publishing Ltd            | 80 open  | analytics: comScore                                    |
|         |                           |                 |                                |          |               | BA1 1UA                      |                        |                                  | 443 open |                                                        |
|         |                           |                 |                                |          |               | Bath                         |                        |                                  |          |                                                        |
|         |                           |                 |                                |          |               | UNITED KINGDOM               |                        |                                  |          |                                                        |
|       2 | https://wired.com         | 151.101.130.194 | 151.101.0.0/16                 | US       | San Francisco | PO Box 78266                 | +1-415-404-9374        | Fastly                           | 80 open  | tag-managers: Google Tag Manager                       |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | javascript-frameworks: Prototype                       |
|       3 | https://lifewire.com      | 151.101.2.114   | 151.101.0.0/16                 | US       | San Francisco | PO Box 78266                 | +1-415-404-9374        | Fastly                           | 80 open  | advertising-networks: DoubleClick for Publishers (DFP) |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | tag-managers: Google Tag Manager                       |
|       4 | https://pcmag.com         | 52.201.108.115  | 52.192.0.0/11                  | US       | Seattle       | 410 Terry Ave N.             | +1-206-266-4064        | Amazon.com                       |          |                                                        |
|       5 | https://techcrunch.com    | 152.195.50.33   | 152.176.0.0/12, 152.192.0.0/13 | US       | Ashburn       | 22001 Loudoun County Parkway | +1-800-900-0241        | ANS Communications               |          | web-servers: Nginx                                     |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-graphics: D3                                |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: Prototype                       |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | cms: WordPress                                         |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | programming-languages: PHP                             |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | blogs: PHP                                             |
|       6 | https://arstechnica.com   | 3.130.45.15     | 3.128.0.0/9                    | US       | Seattle       | 410 Terry Ave N.             | +1-206-266-4064        | Amazon.com                       |          | web-servers: Nginx                                     |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | advertising-networks: DoubleClick for Publishers (DFP) |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | tag-managers: Google Tag Manager                       |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | cms: WordPress                                         |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | programming-languages: PHP                             |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | blogs: PHP                                             |
|       7 | https://pcworld.com       | 151.101.130.165 | 151.101.0.0/16                 | US       | San Francisco | PO Box 78266                 | +1-415-404-9374        | Fastly                           | 80 open  | tag-managers: Google Tag Manager                       |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | analytics: comScore                                    |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: jQuery                          |
|       8 | https://laptopmag.com     | 185.113.25.56   | 185.113.24.0/22                | GB       |               | Quay House, The Ambury       | None                   | Future Publishing Ltd            | 80 open  | analytics: comScore                                    |
|         |                           |                 |                                |          |               | BA1 1UA                      |                        |                                  | 443 open |                                                        |
|         |                           |                 |                                |          |               | Bath                         |                        |                                  |          |                                                        |
|         |                           |                 |                                |          |               | UNITED KINGDOM               |                        |                                  |          |                                                        |
|       9 | https://slashdot.org      | 216.105.38.15   | 216.105.32.0/20                | US       | San Diego     | 9725 Scranton Road           | +1-888-327-8375;ext501 | American Internet Services, LLC. |          | web-servers: Nginx                                     |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | analytics: Cross Pixel                                 |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: Handlebars                      |
|      10 | https://computerworld.com | 151.101.194.165 | 151.101.0.0/16                 | US       | San Francisco | PO Box 78266                 | +1-415-404-9374        | Fastly                           | 80 open  | tag-managers: Google Tag Manager                       |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | analytics: comScore                                    |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: jQuery                          |
|      11 | https://theregister.co.uk | 104.18.235.86   | 104.16.0.0/12                  | US       | San Francisco | 101 Townsend Street          | +1-650-319-8930        | Cloudflare                       | 80 open  | cdn: CloudFlare                                        |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open |                                                        |
|      12 | https://cio.com           | 151.101.194.165 | 151.101.0.0/16                 | US       | San Francisco | PO Box 78266                 | +1-415-404-9374        | Fastly                           | 80 open  | tag-managers: Google Tag Manager                       |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | analytics: comScore                                    |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: jQuery                          |
|      13 | https://techmeme.com      | 75.126.78.226   | 75.126.78.224/29               | US       | Menlo Park    | 405 El Camino Real #607      | +1-415-404-9374        | SoftLayer Technologies           |          | web-servers: Apache                                    |
|      14 | https://extremetech.com   | 52.216.10.82    | 52.192.0.0/11                  | US       | Seattle       | 410 Terry Ave N.             | +1-206-266-4064        | Amazon.com                       |          |                                                        |
|      15 | https://techdirt.com      | 104.25.96.73    | 104.16.0.0/12                  | US       | San Francisco | 101 Townsend Street          | +1-650-319-8930        | Cloudflare                       | 80 open  | cdn: CloudFlare                                        |
|         |                           |                 |                                |          |               |                              |                        |                                  | 443 open | analytics: Chartbeat                                   |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | font-scripts: Google Font API                          |
|         |                           |                 |                                |          |               |                              |                        |                                  |          | javascript-frameworks: jQuery                          |

#### **Оценка результата** 

В результате выполнения данной работы получилось сделать универсальное решение по сбору информации о доменах.

#### **Вывод**

В данной лабораторной работе я научился автоматизировать процесс сбора информации о доменах, используя язык программирования Python, а также применять библиотеки типа Shodan, Builtwith, IPWhois.