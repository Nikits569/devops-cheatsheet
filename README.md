Диагностика ресурсов 

top / htop - показывает утилизацию железа в реальном врмени в htop можно использовать интерфейс
метрики: 
  load average - средняя нагрузка за 1, 5, 15 минут если число выше количества ядер CPU то система перегружена 
  VIRT - сколько виртуальной памяти выделено под процес 
  RES - реальна оп память 
  SHR - разделяемая памят 

strace - перехватыет системные вызовы которая програма делает к ядру 
-p <PID> - подключится к запущеному процесу 
-c - статистика сколько времени ушло и CPU ушло на каждый вызов
-f - трасирвока всех дочерних потоков и процесов 
-s <размер> - макс длина вывода строк нарпимер 1024
-o /path/info.log - сбор в логи
-e trace=network/file/process - фильтрация 

Анализ логов
/var/log/syslog
/var/log/auth.log
/var/log/nginx

journalctl -u nginx.service - показать конкретый лог 
-f - реальное время 
-e - промотать в самый конец 
-b - логи начиная с текущего запуска 
-p alert/emerg/crit/err - показывать только что то конкретное 

Права доступа и безопасность 
chmod [опции] [код]  [файл или путь] - меняем права на каталог или файл 
-R - рекурсивный флаг для дерикторий который применяет опции и для папок внутри 
chown [пользователь]:[группа] файл

umask - определяет, какие права будут забраны у новых файлов/директорий при создании

apt update - подтянуть свежие индексы репозиториев из интернета
apt upgrade - обновить установленные пакеты до актуальных версий
apt install <package> - установить пакет
apt remove <package> - удалить (конфиги остаются)

systemctl status nginx - проверить состояние (работает/упал/ошибки)
systemctl restart nginx - перезапустить
systemctl reload nginx - перечитать конфиги без обрыва текущих

df -h — проверить свободное место на дисках

ip a - просмотреть сетевые интерфейсы и IP адреса 
ip route - показать таблицу маршрутизации 

ss - показ сокетов, открытых портов и активные подключания 
ss -tulpn - показать все слушающие порты 
-t - TCP
-u - UDP
-l - listen
-p - process
-n - не резолвить в IP 
ss -s - общая статистика по сокетам 

traceroute - Показывает маршрут пакетов до целевого хоста
traceroute google.com 

tcpdump - Перехватывает сетевые пакеты «на лету». Спасает, когда другие методы бессильны
tcpdump -i eth0 -nn -s0 -v port 80 — слушать интерфейс eth0 на 80 порту

A - IPv4 
AAAA - IPv6
CNAME - алиас (псевдоним) с одного домена на другой (например, [www.example.com](https://www.example.com) -> example.com)
MX (Mail Exchange) - указывает серверы, принимающие почту для домена (с приоритетами: чем меньше цифра, тем выше приоритет)
TXT - произвольный текст. Используется для подтверждения прав на домен (Google/GitHub/SSL) и для почтовых политик
SPF (внутри TXT) - список IP-адресов и серверов, которым разрешено отправлять почту от имени твоего домена
DKIM (внутри TXT)- криптографическая подпись писем. Почтовый сервер получателя проверяет подпись по открытому ключу в DNS
DMARC (внутри TXT) - правило для почтовиков (Gmail, Mail.ru), что делать с письмами, которые не прошли SPF или DKIM (пропустить, отправить в спам или отклонить)

dig example.com - дефолтный запрос (по умолчанию ищет A-запись)
dig MX example.com - проверить почтовые шлюзы
dig TXT _dmarc.example.com - проверить DMARC-запись

Как устроен процесс отправки почты:

1 Твой скрипт/приложение отправляет письмо через SMTP (порт 25 или 587) на почтовый сервер (Postfix, Exim и т.д.).
2 Почтовый сервер проверяет аутентификацию (если порт 587/465) и смотрит MX-записи получателя.
3 Почтовый сервер подключается по SMTP (порт 25) к серверу получателя (например, серверу Google) и передает письмо.
4 Получатель проверяет SPF, DKIM, DMARC. Если всё ок — письмо падает во «Входящие», если нет — в спам или заворачивается с ошибкой.

nc 127.0.0.1 25
И дальше пишем по шагам команды протокола SMTP:

EHLO localhost
MAIL FROM:<sender@example.com>
RCPT TO:<recipient@gmail.com>
DATA
Subject: Test mail from CLI
From: sender@example.com
To: recipient@gmail.com

Hello, this is a manual test email!
.
QUIT


Bash 

#!/bin/bash
set -euo pipefail

#!/bin/bash - интерпретатор, который будет выполнять скрипт
set -e - немедленно завершить скрипт, если любая команда вернет ненулевой код ошибки (предотвращает каскадные сбои)
set -u - считать ошибкой обращение к несуществующей переменной (защита от опечаток типа $userr вместо $user)
set -o pipefail - если в конвейере (cmd1 | cmd2) упадет первая команда, то весь конвейер вернет ошибку

APP_NAME="backend-api"
PORT=8000

$0 - имя самого скрипта
$1, $2, ... - аргументы командной строки, переданные скрипту
$# — количество переданных аргументов
$@ - все аргументы разом
$? - код возврата последней выполненной команды (0 — успех, от 1 до 255 — ошибка)

if [ "$ENV" = "production" ]; then
    echo "Внимание! Продакшн-окружение."
elif [ "$ENV" = "staging" ]; then
    echo "Тестовый стенд."
else
    echo "Локальная разработка."
fi

Числа:

-eq (equal) — равно ([ "$a" -eq 10 ])
-ne (not equal) — не равно
-gt / -lt — больше / меньше (>) (<)
-ge / -le — больше или равно / меньше или равно

Строки:

=, != — точное равенство / неравенство строк.
-z "$var" — строка пуста (длина 0).
-n "$var" — строка не пуста.

Файлы и директории:

-f "path" — файл существует и это обычный файл.
-d "path" — директория существует.
-x "path" — файл существует и исполняемый.
-s "path" — файл существует и его размер больше нуля.

# Обход фиксированного списка
for service in nginx postgresql redis; do
    systemctl restart $service
done

# Обход результатов команды
for ip in $(cat servers.txt); do
    echo "Пингуем $ip..."
    ping -c 1 $ip
done

# Чтение файла построчно без потери пробелов
while IFS= read -r line; do
    echo "Обрабатываю строку: $line"
done < config.env

log_info() {
    local message="$1" # Ключевое слово local делает переменную локальной для функции
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') - $message"
}

# Вызов функции
log_info "Деплой успешно завершен"

> — запись в файл с перезаписью (echo "hello" > file.txt)
>> — добавление в конец файла (echo "world" >> file.txt)

2>&1 — перенаправление потока ошибок (stderr, дескриптор 2) в стандартный поток вывода (stdout, дескриптор 1). Часто пишется в конце логов: python manage.py runserver > app.log 2>&1 &
/dev/null — «черная дыра» (мусорное ведро). Если нужно скрыть вывод команды: rm temp.txt 2>/dev/null.

grep (Поиск по шаблону):
grep -rn "API_KEY" /app/ — рекурсивно найти строку во всех файлах каталога с указанием номеров строк (-n).
grep -E "ERROR|CRITICAL" app.log — поиск по расширенному регулярному выражению (ИЛИ).
sed (Потоковый редактор, замена на лету):
sed -i 's/host = localhost/host = db.internal/g' settings.ini — заменить текст во всем файле (-i перезаписывает файл на месте).
sed -i '/^#/d' config.txt — удалить все строки, начинающиеся с комментария (#).
awk (Анализ по колонкам):
ps aux | awk '{print $1, $2, $11}' — вывести только пользователя, PID и имя запущенного процесса (первая, вторая и одиннадцатая колонки).

roles/
  └── webserver/
      ├── tasks/
      │   └── main.yml      # Основной список задач роли
      ├── handlers/
      │   └── main.yml      # Обработчики (вызываются через notify)
      ├── templates/
      │   └── nginx.conf.j2 # Шаблоны конфигураций (Jinja2)
      ├── files/
      │   └── id_rsa.pub    # Статические файлы (копируются as-is)
      ├── vars/
      │   └── main.yml      # Переменные роли (высокий приоритет)
      ├── defaults/
      │   └── main.yml      # Переменные по умолчанию (низкий приоритет)
      └── meta/
          └── main.yml      # Метаданные (зависимости от других ролей)

- name Some Name free
- hosts: web_servers
  become: yes          # Выполнять задачи с повышенными привилегиями (sudo)
  vars_files:
    - vars/global.yml  # Подключение глобальных переменных
  roles:
    - common
    - webserver
- name: Создать рабочую директорию
  file:
    path: /var/www/app
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'

# Скопировать статический файл
- name: Скопировать SSH ключ
  copy:
    src: files/id_rsa.pub
    dest: /home/deploy/.ssh/authorized_keys
    mode: '0600'

# Сгенерировать конфиг из Jinja2 шаблона
- name: Развернуть конфигурацию Nginx
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/default
    mode: '0644'
  notify: Reload Nginx  # Триггерит handler при изменении файла

# Установка пакетов (Ubuntu/Debian)
- name: Установить системные пакеты
  apt:
    name:
      - curl
      - git
      - ufw
    state: present
    update_cache: yes  # Аналог apt update

# Управление службой
- name: Запустить и включить Nginx
  systemd:
    name: nginx
    state: started     # Возможные: started, stopped, restarted, reloaded
    enabled: yes       # Добавить в автозагрузку

# Добавить или изменить конкретную строку в файле
- name: Настроить параметры ядра
  lineinfile:
    path: /etc/sysctl.conf
    line: 'net.core.somaxconn = 1024'
    create: yes

# Вставить целый блок текста между маркерами
- name: Добавить правила в конфиг
  blockinfile:
    path: /app/.env
    block: |
      DATABASE_URL=postgres://user:pass@localhost/db
      DEBUG=False

- name: Перезапустить Apache (только для RedHat)
  systemd:
    name: httpd
    state: restarted
  when: ansible_os_family == "RedHat" and env == "production"

- name: Создать системные группы
  group:
    name: "{{ item }}"
    state: present
  loop:
    - developers
    - sysadmins
    - deployers

# Цикл по списку словарей
- name: Создать пользователей с разными шеллами
  user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
  loop:
    - { name: 'alice', shell: '/bin/bash' }
    - { name: 'bob', shell: '/bin/sh' }
  loop_control:
    label: "{{ item.name }}" # Скрывает мусор в логах выполнения

handlers:
  - name: Reload Nginx
    systemd:
      name: nginx
      state: reloaded

Проверить синтаксис плейбука:
ansible-playbook --syntax-check site.yml

Dry Run (Сухой прогон / симуляция): Показывает, что изменится, ничего реально не трогая на сервере.
ansible-playbook -i inventory/hosts.ini site.yml --check

Запустить плейбук для конкретного тега (удобно для деплоя отдельных частей):
ansible-playbook -i inventory/hosts.ini site.yml --tags "nginx,deploy"

Одноразовая команда без написания плейбука (Ad-Hoc):
ansible all -i "192.168.1.50," -u root -m ping

Настрйока сервера nginx, gunicron 

sudo apt update && sudo apt upgrade -y
sudo apt install nginx python3-pip python3-venv git curl -y

sudo nano /etc/nginx/sites-available/config

server {
    server_name sniperlead.sk www.sniperlead.sk;

    location /static/ {
        alias /var/www/sniperlead/site/sniper_lead/sniper_lead/staticfiles/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/sniperlead.sk/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/sniperlead.sk/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.sniperlead.sk) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = sniperlead.sk) {
        return 301 https://$host$request_uri;
    } 


    listen 80;
    server_name sniperlead.sk www.sniperlead.sk;
    return 404; 


}

дальше юникорн 

sudo nano /etc/systemd/system/config.service

[Unit]
Description=SniperLead Django site
After=network.target

[Service]
User=root
WorkingDirectory=/var/www/sniperlead/site/sniper_lead
EnvironmentFile=/var/www/sniperlead/site/sniper_lead/.env

ExecStart=/var/www/sniperlead/site/sniper_lead/venv/bin/gunicorn sniper_lead.wsgi:application --bind 127.0.0.1:8000 --access-logfile - --error-logfile -

Restart=always

[Install]
WantedBy=multi-user.target

