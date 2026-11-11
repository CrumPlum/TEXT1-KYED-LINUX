# Встановлення на іншому Linux

Пакет використовує keyd. Базові клавіатурні шари працюють на Linux, але інтеграція пошуку, перекладу, запуску програм і фокусування вікон у комплекті налаштована для Wayland + KDE Plasma 6. Перед встановленням змініть прив'язки пристрою та параметри користувача, описані нижче.

## 1. Встановіть залежності

Встановіть `keyd`, `wl-clipboard` (команди `wl-copy`, `wl-paste`), Python 3 та `kdotool` з пакетного менеджера вашого дистрибутива. Для запуску програм з цих прикладів потрібні відповідні програми; поточні помічники розраховані на Flatpak Chrome і Obsidian. KWin-пакети додатково потребують інструментів KDE Plasma 6.

Запустіть службу keyd за інструкціями свого дистрибутива. Не копіюйте конфіг, доки не перевірили ID своєї клавіатури.

## 2. Визначте клавіатуру та ім'я користувача

```sh
sudo keyd monitor
```

Знайдіть пристрій клавіатури та його ID у форматі `vvvv:pppp`. Відредагуйте `outputs/default.conf`: замініть ID у секції `[ids]`. Можна видалити `[ids]` разом із рядком ID, щоб застосовувати конфіг до всіх клавіатур, але тоді переназначення діятиме також на вбудованій/інших клавіатурах.

У helper-файлах `keyd-insert-text.sh`, `keyd-google-search.sh`, `keyd-google-translate.sh`, `keyd-obsidian.sh`, `keyd-raise-chrome.sh` знайдіть `user=sandr` і замініть `sandr` на ім'я вашого Linux-користувача. У `keyd-insert-text.sh` шлях `/home/sandr` також має відповідати домашній папці користувача. Поточні команди запуску Flatpak (`com.google.Chrome`, `md.obsidian.Obsidian`) замініть на ID ваших програм або відповідні команди запуску.

> Команди запуску та передавання фокусу розраховані на графічний сеанс Wayland. В X11/іншому середовищі ці helper-и можуть потребувати адаптації.

## 3. Встановіть конфіг і helper-и

З кореня розпакованого пакета:

```sh
sudo install -Dm644 outputs/default.conf /etc/keyd/default.conf
sudo install -Dm755 outputs/keyd-insert-text.sh /usr/local/bin/keyd-insert-text
sudo install -Dm755 outputs/keyd-google-search.sh /usr/local/bin/keyd-google-search
sudo install -Dm755 outputs/keyd-google-translate.sh /usr/local/bin/keyd-google-translate
sudo install -Dm755 outputs/keyd-obsidian.sh /usr/local/bin/keyd-obsidian
sudo install -Dm755 outputs/keyd-raise-chrome.sh /usr/local/bin/keyd-raise-chrome
sudo keyd check
sudo systemctl enable --now keyd
sudo systemctl restart keyd
```

Назви сервісу/команди можуть відрізнятися у вашому дистрибутиві. Перевірте `systemctl is-active keyd`.

## 4. Встановіть інтеграцію KDE (необов'язково)

Для активації вікон Obsidian/Chrome та CapsLock+= відкрийте термінал у графічній сесії KDE і виконайте від звичайного користувача:

```sh
./outputs/install-kwin-activate-launches.sh
./outputs/install-kwin-toggle-keep-above.sh
```

Ці скрипти призначені для Plasma 6, не запускайте їх через `sudo`. На інших робочих середовищах пропустіть цей крок і приберіть непотрібні відповідні прив'язки з конфіга.

## 5. Перевірте та адаптуйте

- Довідник комбінацій: [GUIDE.md](GUIDE.md).
- Перевірка синтаксису: `sudo keyd check`.
- Перегляд фізичних і віртуальних натискань: `sudo keyd monitor`.
- Перевірка журналу: `sudo journalctl -u keyd -b --no-pager`.
- Аварійне вимкнення keyd: Backspace+Escape+Enter.

Комбінації CapsLock та російські літери залежать від того, чи є `CapsLock` шаром у вашому конфігу. Якщо бажаєте лишити системну поведінку CapsLock, вилучіть `capslock = layer(caps)` і секцію `[caps]` або змініть модифікатор на свій.

Символи, email, дата та російські літери вставляються через Wayland clipboard, тому потрібен активний користувацький графічний сеанс і `wl-clipboard`. Попередній текстовий буфер відновлюється після вставлення; нетекстові/форматовані clipboard-дані можуть не відновитися. Інтеграційні shell-скрипти мають налаштовуватися під власне середовище.
