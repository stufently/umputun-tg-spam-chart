# umputun-tg-spam-chart



Helm chart для развертывания tg-spam бота.

## Использование

### Добавление репозитория

```bash
helm repo add umputun-tg-spam https://ORG.github.io/umputun-tg-spam-chart
helm repo update
```

Замените `ORG` на имя вашей GitHub организации или пользователя.

### Поиск чартов

```bash
helm search repo umputun-tg-spam
```

### Установка

```bash
helm install my-tg-spam umputun-tg-spam/tg-spam -f values.yaml --version 0.1.0
```

### Обновление

```bash
helm upgrade my-tg-spam umputun-tg-spam/tg-spam -f values.yaml --version 0.1.0
```

## Разработка

### Локальная разработка

```bash
# Упаковка чарта
helm package tg-spam-chart

# Создание индекса
helm repo index . --url https://ORG.github.io/umputun-tg-spam-chart
```

### Публикация

Чарт автоматически публикуется в GitHub Pages при пуше изменений в `tg-spam-chart/**`
в ветку `main` или `master` через GitHub Actions workflow.

Для ручного запуска:
1. Перейдите в раздел Actions в GitHub
2. Выберите workflow "Release Helm Chart"
3. Нажмите "Run workflow"

⚠️ **ВСЕГДА бампайте `version` в `Chart.yaml` вместе с правкой чарта.** Workflow
перезаписывает `.tgz` под той же версией, а потребители (ArgoCD-приложения
`tg-spam-*` в кластере romtk3s) стоят на `targetRevision: <версия>` с
`syncPolicy.automated.selfHeal` — то есть публикация ИЗМЕНЁННОГО чарта под УЖЕ
используемой версией немедленно перекатит все деплойменты антиспама. С бампом
версии публикация безопасна: старая версия продолжает обслуживать кластер, пока
`targetRevision` не поднимут вручную в `romtk3s`.

⚠️ **`[skip ci]` в коммите по `tg-spam-chart/**` = чарт НЕ опубликован.** Грабля
2026-07-26: коммит «Pin busybox digest, chart 0.8.0 [skip ci]» поднял `version` в
`Chart.yaml`, но workflow не запустился, поэтому в `index.yaml` максимум остался
`0.7.0`, а `targetRevision` в `romtk3s` продолжал (корректно) указывать на `0.7.0`.
Расхождение «локальный Chart.yaml новее опубликованного index» безопасно, но легко
принять за поломку. **Перед сменой `targetRevision` в `romtk3s` всегда проверяй
факт публикации:** `curl -s https://stufently.github.io/umputun-tg-spam-chart/index.yaml`
— нужной версии там нет, значит её нет и для ArgoCD.

### Версия образа tg-spam

Тег апстрим-образа `ghcr.io/umputun/tg-spam` берётся из `Chart.appVersion`
(`tgspam.image.tag` пустой). Порядок апгрейда апстрима:

1. `appVersion` → новый тег, `version` → бамп (обязательно, см. выше);
2. push БЕЗ `[skip ci]`, дождаться workflow и появления версии в `index.yaml`;
3. только после этого поднять `targetRevision` в
   `romtk3s/tg-spam/tg-spam-appset.yaml` — ArgoCD (automated+selfHeal, strategy
   `Recreate`) сам перекатит все 10 деплойментов.

### Пины (воспроизводимость)

- `tgspam.initContainer.image` — образ initContainer'а, по умолчанию
  `busybox:1.38.0@sha256:fd8d9aa63ba2f0982b5304e1ee8d3b90a210bc1ffb5314d980eb6962f1a9715d`
  (раньше в шаблоне был захардкожен `busybox:latest`). initContainer работает
  под `runAsUser: 0` и делает `chown -R` на PVC, поэтому плавающий тег там —
  лишний неконтролируемый root-образ. Ссылка «тег + дайджест»: тег для читаемости,
  дайджест для неизменяемости (апстрим может переписать и сам тег `1.38.0`).
  Это ровно тот образ, что крутится в кластере (взят из
  `.status.initContainerStatuses` боевого `tg-spam-adpattaya`, 2026-07-25);
  дайджест мультиарховый, arm64/amd64 не ломаются.
  Переопределяется через values, если нужен другой образ.
- `azure/setup-helm` в workflow запинен по commit-SHA (`# v3.5`), версия helm —
  `v4.2.2` вместо `latest` (столько скачал последний успешный релиз чарта).

## Настройка GitHub Pages

1. Перейдите в Settings → Pages вашего репозитория
2. Убедитесь, что источник установлен на "GitHub Actions" или "gh-pages" branch
3. После первого запуска workflow репозиторий будет доступен по адресу:
   `https://ORG.github.io/umputun-tg-spam-chart`
