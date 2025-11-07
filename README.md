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

Чарт автоматически публикуется в GitHub Pages при пуше изменений в ветку `main` или `master` через GitHub Actions workflow.

Для ручного запуска:
1. Перейдите в раздел Actions в GitHub
2. Выберите workflow "Release Helm Chart"
3. Нажмите "Run workflow"

## Настройка GitHub Pages

1. Перейдите в Settings → Pages вашего репозитория
2. Убедитесь, что источник установлен на "GitHub Actions" или "gh-pages" branch
3. После первого запуска workflow репозиторий будет доступен по адресу:
   `https://ORG.github.io/umputun-tg-spam-chart`
