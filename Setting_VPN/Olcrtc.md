### OLCRTC, проект созданный для обхода белых списков

Этот имбовый webRTC инструмент, изначально создавался для обхода белых спиков и по сей день разработчики решают разные вопросы и проблемы с которыми сталкиваются. Но +- в мае 2026 года вышло маштабное обновление в котором этот проект стало возможным использовать и для обхода простых блокиров (чёрных списков), а что именно добавилось я расскажу ниже.
# Что это и для чего это?

Этот полностью Open Source-ный проект был опубликован в общий доступ в апреле 2026 года, сразу же вышла статья в которой автор рассказывает как работает olcrtc, хоть и статья уже полностью не актуальна, так как проект очень сильно поменялся, основное количество людей пришло в проект именно после этой статьи.

---

Так вот...
# Настройка olcrtc

Так вот про настройку, есть кучу панелей быстрых скриптов и т.д, но всё это делают другие разрабы и обновляются они не сразу, да и настроить базовым способ супер просто (я бы для этого отдельную статью не создавал чтобы показывать как настроить какието панели, в которых и так всё расписано. Но так как я понимаю что обычный пользователь, как и я 3 месяца назад, вообще не знает ничего, я решил выпустить эту статью, ну а ещё мне лень искать постоянно старые ответы от нейронок, так что я решил сюда просто поместить, чтобы в будующем пользаваться клавишами Ctrl+C и Ctrl+V). Ну короче мы будем делать так как это делает разраб.

---

## Первый этап настройки:

### Swap (ОЗУ)

Если у вас меньше 4ГБ оперативной памяти, сборка может вылетать. Обязательно включите SWAP:

```
sudo fallocate -l 4G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile
```

### Что нужно установить

---

### Шаг 1: Установить git

Debian   / Ubuntu  / Mint

```
apt install git       
```

Arch    / CachyOS / Manjaro

```
pacman -S git
```

Fedora / RHEL   / CentOS

```
dnf install git
```

### Шаг 2: Установить Go 1.26+

Arch / Fedora (всё просто)

```
pacman -S go    # Arch    / CachyOS / Manjaro
dnf install go  # Fedora / RHEL   / CentOS
```

Debian / Ubuntu:

Скачать официальный архив: 
```
curl -OL https://go.dev/dl/go1.26.0.linux-amd64.tar.gz
```

Распаковать его в чистую папку:

```
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.26.0.linux-amd64.tar.gz
```

Должно быть вот так:

[установка go]()

### Шаг 3: Установить mage

Mage - система сборки для Go-проектов, аналог make.

```
go install github.com/magefile/mage@latest
```

Проверка:

```
mage --version
```

Добавь ~/go/bin в PATH:

```
echo 'export PATH="$HOME/go/bin:$PATH"' >> ~/.bashrc
```

```
source ~/.bashrc
```

Должно быть вот так:

[установка mage]()

### Шаг 4: Скачать репозиторий

```
git clone https://github.com/openlibrecommunity/olcrtc
```

Перейдём в скаченную директорию (репозиторий):

```
cd olcrtc
```

Вот так:

[скачивание репозитория]()

### Шаг 5: Собрать

```
mage build
```

Результат:

```
build/olcrtc-linux-amd64
```

[результат build]()

## Второй этап:

### Если вы не хотите сильно заморачиваться, но так же хотите обходить белые списки вам достаточно будет воспользоваться редактором nano и создать следующий файл:

### Но перед этим сгенерируем ключ шифрования:

```
openssl rand -hex 32
```

Для примера я буду исользовать jitsi, но если вы хотите гнать трафик не только через jitsi, перейдите [в репозиторий к разрабу](https://github.com/openlibrecommunity/olcrtc/blob/master/docs/manual.md#wbstream--vp8channel-%D0%B0%D0%BB%D1%8C%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D1%82%D0%B8%D0%B2%D0%B0)

```
nano server.yaml
```

```
# server.yaml
mode: srv
auth:
  provider: jitsi
room:
  # Используйте meet1.arbitr.ru или meet.cryptopro.ru - тот, что работает в вашей сети
  id: "https://meet.handyweb.org/test"
crypto:
  key: "ключ шифрования который вы сгенерировали"
net:
  transport: datachannel
  dns: "8.8.8.8:53"
data: data

   # olcrtc://jitsi?datachannel@https://meet.handyweb.org/test#ключ шифрования который вы сгенерировали$Описание
```

тут вам нужно заменить id (комнату) Jitsi Meet, и key (ключ шифрования), так же тут я добавил от себя последнюю строку, это коментарий который вам понадобится в клиенте olcbox, он является ссылкой для этого и других клиентов (как vless:// в xray), сделал я это для удобства, её тоже нужно изменить

### Формат этой ссылки следующий:

```text
olcrtc://<Auth>?<Transport>@<RoomID>#<EncryptionKey>$<MIMO>
olcrtc://<Auth>?<Transport><key=value&key=value>@<RoomID>#<EncryptionKey>$<MIMO>
```

Блок `<key=value&...>` - payload параметров транспорта в угловых скобках, идёт сразу после имени транспорта. Если параметры транспорту не нужны или используются defaults - блок опускается целиком.

Поля

| Поле | Значение |
|------|----------|
| `<Auth>` | Имя auth-провайдера, например `telemost`, `wbstream`, `jitsi` |
| `<Transport>` | Имя транспорта, например `datachannel`, `vp8channel`, `seichannel`, `videochannel` |
| payload | Параметры транспорта в `<key=value&...>`. Ключи совпадают с YAML полями. Блок опускается если используются defaults |
| `<RoomID>` | Идентификатор комнаты или auth-specific room URL/ID |
| `<EncryptionKey>` | Ключ шифрования в hex, обычно 64 символа (`32` байта) |
| `<MIMO>` | Свободный комментарий для UI/метаданных, например `RU / olc free sub / IPv6` |

### Запустим

Для запуска воспользуемся командой:

```
./build/olcrtc-linux-amd64 server.yaml
```

Результат:

[результат запуска]()

Но как только мы закроем терминал, всё перестанет по этому переходим к следующему этапу

## Третий этап. Создаем основной шаблонный сервис

Ну короче и после того как мы всё сделали нам нужно как то это протестировать, да и чтоб работало оно 24/7, с этим нам поможет systemctl.

Чтобы проверить есть ли вообще эта служба на нашем сервере воспользуемся командой:

```
which systemctl
```

Если утилита есть, команда вернет путь к файлу:

[](/image/Рисунок10.png)

Ну и если вы захотите протестировать сразу несколько сервисов, вы создадитенесколько .yaml конфигов, а для них потребуется отдельно кучу сервисов, мы же создадим следующий файл в директории путь к которой мы знали командой ```which systemctl```

```
nano /etc/systemd/system/olcrtc@.service
```

```
[Unit]
Description=OLCRTC Proxy Server (%i)
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/olcrtc
# Здесь мы обновили путь к конфигурационному файлу
ExecStart=/root/olcrtc/build/olcrtc-linux-amd64 /root/olcrtc/%i.yaml # Меняйте, если у вас другие пути
Restart=always
RestartSec=5
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=olcrtc-%i

[Install]
WantedBy=multi-user.target
```

Проверяем:
введём две следующие команды:

```
systemctl start olcrtc@server
```

```
systemctl enable olcrtc@server
```

А введя команду:

```
systemctl status olcrtc@server
```

Мы увидим следующее:

[статус сервиса systemctl]()

В строке Active обязательно должно быть active (running), как на скрине

---

Дальше мы создаем скрипт и таймер перезапуска, он нам нужен чтообы сервер сам перезапускал наши конфиги, сделал я это для того чтобы если сервер выкинуло из румы в стриминговом сервисе он перезапустился и вошёл туда обратно (ну точнее это мои догатки, так как иногда я сталкивался с проблемой кто конфиг не работает так как сервера просто нету в руме, неисключено что это не помагает):

```
cat >/etc/systemd/system/olcrtc-restart-all.service <<'EOF'
[Unit]
Description=Restart all olcrtc instances

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'systemctl restart olcrtc@*'
EOF
```

```
cat >/etc/systemd/system/olcrtc-restart-all.timer <<'EOF'
[Unit]
Description=Restart olcrtc every 2 hours

[Timer]
OnBootSec=10min
OnUnitActiveSec=2h
Unit=olcrtc-restart-all.service

[Install]
WantedBy=timers.target
EOF
```

## Четвёртый этап. Проверка.

Переходите [сюда](https://github.com/alananisimov/olcbox), скачиваете приложуху и тестируете.

## Пятый этап. Развёртка self-hosted jitsi-meet







## Этап ...

Чтобы просматреть все запущенные процессы, напишем следующую команду:

```
systemctl list-units --type=service --all
```

Тут мы увидим все наши процессы systemctl, если хотим узнать только запущенные пропишем следующую команду:

```
systemctl list-units --type=service --state=running
```

Нам нужно просмтреть только наши процессы, вы их сразу увидите:

[картинка]()

Полная перезагрузка systemctl:

```
sudo systemctl daemon-reexec
```

