# Как добавить SSH-ключ в GitHub

**Справочник студента** · «Программирование и алгоритмизация» (C++), 1 курс

SSH-ключ позволяет выполнять `git push` и `git pull` без ввода логина и токена при каждой операции. Настройка занимает около 10 минут и делается один раз на каждом компьютере.

## Как это работает

SSH-ключ — это пара файлов:

| Файл | Что это | Что с ним делать |
|---|---|---|
| `~/.ssh/id_ed25519` | **Закрытый (приватный) ключ** | Хранится только на вашем компьютере. Никому не отправляйте, не загружайте на сайты, не коммитьте в репозиторий |
| `~/.ssh/id_ed25519.pub` | **Открытый (публичный) ключ**, расширение `.pub` | Загружается на GitHub. Его можно показывать кому угодно |

При подключении GitHub проверяет, что у вас есть закрытый ключ, соответствующий загруженному открытому, — сам закрытый ключ по сети не передаётся. `~` обозначает домашний каталог: `/home/<имя>` в Linux и WSL, `/Users/<имя>` в macOS, `C:\Users\<имя>` в Windows.

## Где выполнять команды

| Система | Где | Примечание |
|---|---|---|
| Linux | Терминал | — |
| macOS | Терминал (Terminal) | — |
| Windows + WSL | Терминал Ubuntu (WSL) | Если git запускается внутри WSL, ключ создаётся там же |
| Windows без WSL | **Git Bash** (устанавливается вместе с Git for Windows) | Команды такие же, как в Linux. Отличия для PowerShell указаны отдельно |

WSL и Windows — разные окружения со своими каталогами `~/.ssh`. Ключ, созданный в WSL, используется только git внутри WSL, а ключ из Git Bash — только git в Windows. Если вы работаете с git и там, и там, создайте ключ в каждом окружении и добавьте на GitHub оба.

---

## Шаг 1. Проверить, нет ли ключа

```bash
ls -la ~/.ssh
```

| Результат | Что делать |
|---|---|
| `No such file or directory` или в списке нет файлов `.pub` | Ключа нет — переходите к шагу 2 |
| Есть `id_ed25519.pub` (или `id_rsa.pub`), и вы знаете, что это ваш ключ | Можно использовать его — переходите к шагу 4 |
| Файлы есть, но вы не знаете, откуда они | Не удаляйте и не перезаписывайте их: создайте новый ключ с другим именем (см. примечание в шаге 2) |

## Шаг 2. Создать ключ

```bash
ssh-keygen -t ed25519 -C "ivan@example.com"
```

| Часть команды | Значение |
|---|---|
| `ssh-keygen` | Программа создания ключей (входит в OpenSSH, есть в Linux, macOS, Git Bash и Windows 10/11) |
| `-t ed25519` | Тип ключа Ed25519 — современный, короткий и надёжный; его рекомендует GitHub |
| `-C "ivan@example.com"` | Комментарий к ключу — укажите почту от аккаунта GitHub, чтобы потом понимать, чей это ключ |

Программа задаст три вопроса:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ivan/.ssh/id_ed25519):    ← нажмите Enter
Enter passphrase (empty for no passphrase):                          ← придумайте пароль
Enter same passphrase again:                                         ← повторите пароль
Your identification has been saved in /home/ivan/.ssh/id_ed25519
Your public key has been saved in /home/ivan/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:… ivan@example.com
```

1. **Куда сохранить ключ** — нажмите Enter, чтобы принять путь по умолчанию.
2. **Пароль (passphrase)** — защищает ключ, если кто-то получит доступ к вашему компьютеру или файлу ключа. Рекомендуется задать; чтобы не вводить его при каждом `git push`, ключ добавляют в ssh-agent (шаг 3). При вводе символы не отображаются — это нормально. Можно оставить пустым, нажав Enter дважды, но тогда любой, кто скопирует файл ключа, получит доступ к вашему GitHub.
3. **Повтор пароля.**

Если появился вопрос `/home/ivan/.ssh/id_ed25519 already exists. Overwrite (y/n)?` — ответьте `n`: у вас уже есть ключ, и перезапись сломает доступ везде, где он используется. Чтобы создать дополнительный ключ, укажите другое имя файла: `ssh-keygen -t ed25519 -C "ivan@example.com" -f ~/.ssh/id_ed25519_github`.

**На общих компьютерах** (компьютерный класс) ключ лучше не создавать. Если без него не обойтись — обязательно с паролем, а после окончания курса удалите ключ с компьютера и из настроек GitHub.

## Шаг 3. Добавить ключ в ssh-agent

ssh-agent — фоновая программа, которая запоминает расшифрованный ключ, чтобы пароль не приходилось вводить при каждой операции. Если вы не задали пароль на шаге 2, этот шаг можно пропустить.

### Linux, WSL, Git Bash

```bash
eval "$(ssh-agent -s)"          # запустить агент; ответ вида «Agent pid 59566»
ssh-add ~/.ssh/id_ed25519       # добавить ключ; один раз спросит пароль
```

| Команда | Что делает |
|---|---|
| `eval "$(ssh-agent -s)"` | Запускает ssh-agent и сообщает терминалу, как к нему обращаться |
| `ssh-add ~/.ssh/id_ed25519` | Добавляет закрытый ключ в агент (указывается файл **без** `.pub`) |
| `ssh-add -l` | Показывает ключи, загруженные в агент |

В Linux и WSL агент работает, пока открыт терминал; в новом окне эти две команды нужно повторить. В большинстве графических окружений Linux агент запускается автоматически, и достаточно `ssh-add`.

### macOS

Создайте или откройте файл настроек SSH:

```bash
touch ~/.ssh/config
open -e ~/.ssh/config
```

Добавьте в него строки (если пароля у ключа нет, строку `UseKeychain` не пишите):

```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

Затем добавьте ключ в агент с сохранением пароля в связке ключей macOS:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

### Windows: PowerShell и встроенный OpenSSH

Если вы работаете не в Git Bash, а в PowerShell. В окне PowerShell, **запущенном от имени администратора**:

```powershell
Get-Service -Name ssh-agent | Set-Service -StartupType Manual   # разрешить запуск службы агента
Start-Service ssh-agent                                         # запустить агент
```

Затем в обычном окне PowerShell:

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

Последняя команда заставляет git использовать системный SSH Windows — иначе Git for Windows использует собственный SSH, который не видит агент Windows, и пароль будет спрашиваться при каждом `git push`.

## Шаг 4. Скопировать открытый ключ

Копируется **только файл `.pub`**.

| Где | Команда |
|---|---|
| Linux (или вручную в любой системе) | `cat ~/.ssh/id_ed25519.pub`, затем выделить строку и скопировать |
| macOS | `pbcopy < ~/.ssh/id_ed25519.pub` |
| Git Bash | `clip < ~/.ssh/id_ed25519.pub` |
| WSL | `cat ~/.ssh/id_ed25519.pub \| clip.exe` |
| PowerShell | `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub \| Set-Clipboard` |

Команды `pbcopy`, `clip` и `clip.exe` помещают ключ сразу в буфер обмена, ничего не выводя на экран.

Правильный открытый ключ — **одна строка** вида:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI… ivan@example.com
```

Она начинается с `ssh-ed25519` и заканчивается вашим комментарием. Если скопированный текст начинается с `-----BEGIN OPENSSH PRIVATE KEY-----`, вы открыли закрытый ключ — не вставляйте его никуда и скопируйте файл с расширением `.pub`.

## Шаг 5. Добавить ключ на GitHub

1. Войдите на github.com.
2. Нажмите на свой аватар в правом верхнем углу → **Settings**.
3. В левом меню, в разделе **Access**, выберите **SSH and GPG keys**.
4. Нажмите **New SSH key**.
5. **Title** — название, по которому вы узнаете компьютер: «Ноутбук», «Домашний ПК», «WSL на ноутбуке».
6. **Key type** — оставьте **Authentication Key** (ключ для входа; вариант Signing Key нужен для подписи коммитов).
7. **Key** — вставьте скопированную строку без лишних пробелов и переносов.
8. Нажмите **Add SSH key**. GitHub может попросить подтвердить действие паролем от аккаунта или кодом двухфакторной аутентификации.

Ключ появится в списке с отпечатком вида `SHA256:…`. Свой отпечаток можно посмотреть командой `ssh-keygen -lf ~/.ssh/id_ed25519.pub` и сверить — это помогает, когда на GitHub несколько ключей и непонятно, какой из них какой.

## Шаг 6. Проверить подключение

```bash
ssh -T git@github.com
```

При первом подключении SSH покажет отпечаток сервера и спросит, доверять ли ему:

```
The authenticity of host 'github.com (…)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Сверьте отпечаток** с официальным списком GitHub (страница «GitHub's SSH key fingerprints» в документации GitHub):

| Тип ключа сервера | Отпечаток |
|---|---|
| Ed25519 | `SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU` |
| ECDSA | `SHA256:p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM` |
| RSA | `SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s` |

Если отпечаток совпадает — наберите `yes` полностью и нажмите Enter. Если не совпадает — **не подключайтесь**: вы соединяетесь не с GitHub (например, в сети стоит перехватывающий прокси). Отпечатки взяты из документации GitHub в октябре 2026 года; при сомнениях сверьтесь с актуальной страницей.

Успешный результат:

```
Hi ivan-petrov! You've successfully authenticated, but GitHub does not provide shell access.
```

Фраза «does not provide shell access» — это нормально: GitHub не даёт командную строку на своих серверах, а только принимает git-операции. Код возврата этой команды при успехе равен 1, это тоже нормально. Главное — в ответе ваше имя пользователя.

## Шаг 7. Работать с репозиториями через SSH

У каждого репозитория на GitHub два адреса:

| Протокол | Вид адреса |
|---|---|
| HTTPS | `https://github.com/ivan-petrov/prog-labs.git` |
| SSH | `git@github.com:ivan-petrov/prog-labs.git` |

SSH-ключ работает только с адресом второго вида.

**Склонировать репозиторий.** На странице репозитория нажмите зелёную кнопку **Code** → вкладка **SSH** → скопируйте адрес:

```bash
git clone git@github.com:ivan-petrov/prog-labs.git
```

**Перевести уже склонированный репозиторий с HTTPS на SSH:**

```bash
git remote -v                                                      # посмотреть текущий адрес (https://…)
git remote set-url origin git@github.com:ivan-petrov/prog-labs.git # заменить на SSH-адрес
git remote -v                                                      # проверить: теперь git@github.com:…
git push                                                           # работает без ввода логина и токена
```

**Связать новый локальный репозиторий с пустым репозиторием на GitHub:**

```bash
git remote add origin git@github.com:ivan-petrov/prog-labs.git
git branch -M main
git push -u origin main
```

---

## Несколько компьютеров

Создавайте **отдельный ключ на каждом компьютере** и добавляйте каждый на GitHub под понятным названием. Не копируйте закрытый ключ с одного компьютера на другой: если компьютер потерян или продан, достаточно удалить на GitHub только его ключ (Settings → SSH and GPG keys → Delete), не трогая остальные.

## GitLab

Порядок тот же, отличается только шаг 5: аватар → **Edit profile** (или **Preferences**) → **SSH Keys** → **Add new key**; поле **Expiration date** можно очистить, чтобы ключ не истекал. Проверка подключения — `ssh -T git@gitlab.com`, для GitLab вуза — `ssh -T git@<адрес GitLab вуза>`.

---

## Частые проблемы

| Сообщение или симптом | Причина | Решение |
|---|---|---|
| `Permission denied (publickey)` | GitHub не принял ни один ключ: ключ не добавлен на сайт, добавлен не тот, агент не знает ключа или ключ создан в WSL, а git запускается в Windows (или наоборот) | Проверить ключи в агенте: `ssh-add -l`; посмотреть, какие ключи пробует SSH: `ssh -vT git@github.com` (строки `Offering public key`); сверить отпечаток `ssh-keygen -lf ~/.ssh/id_ed25519.pub` с отпечатком на странице SSH and GPG keys |
| `ssh: connect to host github.com port 22: Connection timed out` или `Connection refused` | В сети (часто в вузах и офисах) закрыт порт 22 | Подключаться через порт 443 — см. ниже |
| `Host key verification failed` или `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` | Сохранённый отпечаток сервера не совпадает с текущим — например, остался старый RSA-ключ GitHub, заменённый в марте 2023 года | `ssh-keygen -R github.com`, затем снова `ssh -T git@github.com` и сверка отпечатка по таблице из шага 6 |
| `Could not open a connection to your authentication agent` | ssh-agent не запущен | `eval "$(ssh-agent -s)"`, затем `ssh-add` |
| Пароль от ключа спрашивается при каждом `git push` | Ключ не добавлен в агент | Шаг 3; в Windows — также команда `git config --global core.sshCommand …` |
| GitHub при добавлении пишет `Key is already in use` | Этот открытый ключ уже привязан к другому аккаунту GitHub | Создать новый ключ (шаг 2) |
| `WARNING: UNPROTECTED PRIVATE KEY FILE!` | У файла закрытого ключа слишком широкие права (часто после копирования из Windows в WSL) | `chmod 700 ~/.ssh` и `chmod 600 ~/.ssh/id_ed25519` |
| `Bad owner or permissions on ~/.ssh/config` | Слишком широкие права у файла настроек | `chmod 600 ~/.ssh/config` |
| PowerShell: `The '<' operator is reserved for future use` | PowerShell не поддерживает `<` | Команда из таблицы шага 4 для PowerShell |
| `git push` по-прежнему спрашивает логин и пароль | У репозитория HTTPS-адрес | `git remote -v`, затем `git remote set-url` (шаг 7) |
| Забыли пароль от ключа | Восстановить его невозможно | Создать новый ключ, добавить на GitHub, старый удалить в настройках |

### Если закрыт порт 22: SSH через порт 443

GitHub принимает SSH-подключения на порту 443 (тот же порт, что у HTTPS), который в сетях обычно открыт. Проверьте:

```bash
ssh -T -p 443 git@ssh.github.com
```

Если пришёл ответ «Hi …!», добавьте в файл `~/.ssh/config` (создайте его, если нет):

```
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
```

После этого все команды git с адресами `git@github.com:…` пойдут через порт 443 без каких-либо изменений в репозиториях. При первом подключении снова потребуется подтвердить отпечаток — он тот же, что в таблице шага 6.

---

## Коротко

```bash
ls -la ~/.ssh                                   # 1. есть ли ключ
ssh-keygen -t ed25519 -C "ivan@example.com"     # 2. создать ключ (Enter, пароль, пароль)
eval "$(ssh-agent -s)"                          # 3. запустить агент
ssh-add ~/.ssh/id_ed25519                       #    и добавить в него ключ
cat ~/.ssh/id_ed25519.pub                       # 4. скопировать открытый ключ
#                                                 5. GitHub → Settings → SSH and GPG keys → New SSH key
ssh -T git@github.com                           # 6. проверить, сверить отпечаток, yes
git remote set-url origin git@github.com:<пользователь>/<репозиторий>.git   # 7. перейти на SSH
```
