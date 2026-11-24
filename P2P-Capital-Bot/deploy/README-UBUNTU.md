# Запуск P2P Capital на Ubuntu VPS

Бот уже использует Telegram Long Polling. Для VPS менять его на Webhook не нужно. `systemd` будет автоматически запускать бота после перезагрузки сервера и перезапускать процесс при аварийном завершении.

## 1. Подготовка

Загрузите папку `P2P-Capital-Bot` на Ubuntu-сервер. Убедитесь, что в ней есть `bot.py`, `requirements.txt`, `assets/` и `data/`.

Подключитесь по SSH и перейдите в папку проекта, затем выполните:

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
sudo mkdir -p /opt/p2p-capital
sudo cp -a P2P-Capital-Bot /opt/p2p-capital/
cd /opt/p2p-capital/P2P-Capital-Bot
python3 -m venv .venv
.venv/bin/pip install --upgrade pip
.venv/bin/pip install -r requirements.txt
```

Если вы уже находитесь внутри самой папки проекта, копируйте её содержимое в `/opt/p2p-capital/P2P-Capital-Bot`, а не создавайте вложенную папку с таким же названием.

## 2. Настройка токена

Откройте service-файл:

```bash
sudo cp deploy/p2p-capital.service /etc/systemd/system/p2p-capital.service
sudo nano /etc/systemd/system/p2p-capital.service
```

Замените `PASTE_YOUR_BOT_TOKEN_HERE` в строке `Environment=BOT_TOKEN=...` на токен от BotFather. Не публикуйте этот файл и не отправляйте токен в чаты. Ограничьте доступ к файлу:

```bash
sudo chmod 600 /etc/systemd/system/p2p-capital.service
```

В service-файле указан пользователь `ubuntu`. Если ваш сервер использует другое имя пользователя, замените `User=ubuntu` на имя существующего пользователя Linux.

## 3. Запуск 24/7 и автоперезапуск

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now p2p-capital
sudo systemctl status p2p-capital
```

Посмотреть журнал:

```bash
sudo journalctl -u p2p-capital -f
```

Перезапустить после изменения кода:

```bash
sudo systemctl restart p2p-capital
```

Остановить:

```bash
sudo systemctl stop p2p-capital
```

## 4. Важные замечания

- Не запускайте одновременно вторую копию этого же бота с тем же токеном: Telegram polling может конфликтовать.
- База SQLite находится в `data/p2p_capital.db`. Делайте резервные копии этой папки.
- Этот service-файл автоматически перезапускает процесс, но не может восстановить сервер, если сам хостинг выключил или удалил виртуальную машину.
- Бесплатные VPS не дают безусловной гарантии 24/7. Перед использованием проверьте актуальные условия выбранного провайдера.
