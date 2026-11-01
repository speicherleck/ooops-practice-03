# ooops-practice-03 — git: удалённый репозиторий, конфликты, ветки

Практическая работа №3: локальный git-репозиторий, привязка к пустому репозиторию
на GitHub по SSH-ключу, отправка изменений, разрешение конфликта слияния и работа
с веткой `FEAT-123`.

Учебный репозиторий, в котором выполнялась работа:
[`speicherleck/devops_lab3`](https://github.com/speicherleck/devops_lab3).

## Содержимое репозитория

| Путь | Описание |
| --- | --- |
| [`screenshots/`](screenshots/) | Скриншоты, подтверждающие выполнение каждого шага |
| `README.md` | Описание работы |

## Стенд

| Параметр | Значение |
| --- | --- |
| ОС | Linux Mint 22.3 (хост-машина) |
| git | 2.43.0 |
| Аккаунт GitHub | [`speicherleck`](https://github.com/speicherleck) |
| Доступ к GitHub | SSH-ключ `~/.ssh/devops_lab3` (ed25519) |
| Каталог репозитория | `~/Документы/devops/devops_lab3` |

## Ход работы

### 1. Локальный репозиторий

Каталог с репозиторием, имя и почта автора коммитов (локально, только для этого
репозитория) и первый коммит:

```bash
mkdir ~/Документы/devops/devops_lab3
cd ~/Документы/devops/devops_lab3
git init
git config user.name "speicherleck"
git config user.email "speicherleck@gmail.com"
echo "# devops_lab3" > README.md
git add README.md
git commit -m README
git log --oneline
```

В git 2.43 ветка по умолчанию — `master`, как и в методичке.

![git init, config и первый коммит](screenshots/01-git-init-commit.png)

### 2. SSH-ключ

Ключ сохраняется в `~/.ssh/devops_lab3` (закрытый) и `~/.ssh/devops_lab3.pub` (открытый):

```bash
ssh-keygen -t ed25519 -f ~/.ssh/devops_lab3 -C "devops_lab3"
cat ~/.ssh/devops_lab3.pub
```

![ssh-keygen и открытый ключ](screenshots/02-ssh-keygen.png)

Содержимое `.pub` добавлено в профиль GitHub: *Settings → SSH and GPG keys → New SSH key*.

![Ключ в профиле GitHub](screenshots/03-github-ssh-key.png)

### 3. Пустой репозиторий на GitHub

Репозиторий `devops_lab3` создан без README, лицензии и `.gitignore`. GitHub показывает
команды для нового и для существующего локального репозитория — нужен второй вариант
(*…or push an existing repository from the command line*).

![Пустой репозиторий на GitHub](screenshots/04-github-empty-repo.png)

### 4. Привязка удалённого репозитория и push

В ssh-agent хоста загружено несколько ключей от разных аккаунтов GitHub, поэтому
репозиторию явно указан ключ лабораторной (`core.sshCommand`):

```bash
ssh -i ~/.ssh/devops_lab3 -o IdentitiesOnly=yes -T git@github.com
git config core.sshCommand "ssh -i ~/.ssh/devops_lab3 -o IdentitiesOnly=yes"
git remote add origin git@github.com:speicherleck/devops_lab3.git
git branch -M master
git push -u origin master
```

![remote add и git push](screenshots/05-remote-push.png)

Файл появился на сервере:

![README на GitHub](screenshots/06-github-readme.png)

### 5. Конфликт изменений

Изменение локальной копии README и коммит:

```bash
echo "Локальная правка" >> README.md
git commit -am "local change"
git log --oneline
```

![Локальная правка](screenshots/07-local-change.png)

В конец README на GitHub через веб-редактор дописана другая строка
(«Правка на сервере») и закоммичена в `master`:

![Правка в веб-редакторе GitHub](screenshots/08-web-edit.png)

Попытка отправить локальный коммит отклоняется (`rejected … fetch first`): на сервере
есть коммит, которого нет локально. Для `git pull` при разошедшихся ветках git просит
выбрать стратегию — выбрано слияние, после чего `pull` повторён:

```bash
git push
git pull
git config pull.rebase false
git pull
git status
```

Изменения с сервера скачаны, но автоматически не слились — `CONFLICT (content)`
в `README.md`, `git status` показывает файл в *Unmerged paths*:

![push rejected, git pull и конфликт](screenshots/09-push-rejected-conflict.png)

### 6. Разрешение конфликта

В файле обе версии разделены маркерами `<<<<<<<`, `=======`, `>>>>>>>`:

```bash
cat README.md
```

![Маркеры конфликта в README.md](screenshots/10-conflict-markers.png)

Маркеры удалены, в файле оставлены обе правки — локальная и серверная. Слияние
завершено обычным коммитом и отправлено на сервер, теперь push принимается:

```bash
nano README.md
cat README.md
git add README.md
git commit -m "merge origin/master"
git push
git log --oneline --graph
```

![Разрешённый конфликт и успешный push](screenshots/11-conflict-resolved.png)

### 7. Ветка FEAT-123

Новая ветка, коммит в неё, публикация на сервере с созданием одноимённой ветки,
затем слияние в `master`:

```bash
git checkout -b FEAT-123
echo XXX >> README.md
git commit -am feature
git push -u origin FEAT-123
git checkout master
git merge FEAT-123
git push
git log --oneline --graph --all
```

![Ветка FEAT-123 и слияние в master](screenshots/12-branch-merge.png)

На сервере видны обе ветки:

![Ветки на GitHub](screenshots/13-github-branches.png)

## Результат

* создан локальный git-репозиторий, к нему по SSH-ключу привязан репозиторий
  `devops_lab3` на GitHub, изменения отправлены командой `git push`;
* создан и вручную разрешён конфликт слияния между локальной и серверной правкой
  одного места в `README.md`, итоговая версия отправлена на сервер;
* создана ветка `FEAT-123`, опубликована на сервере через `git push -u` и слита в `master`.
