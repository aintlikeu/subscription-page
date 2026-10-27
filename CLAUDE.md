# Форк Remnawave Subscription Page (vpn1)

Копия [remnawave/subscription-page](https://github.com/remnawave/subscription-page) с нашими правками интерфейса
поверх официальных релизов. Развёртывание описано в нашем инфраструктурном репозитории (`newvpn`, шаг 07).

## Ветки и теги

- `main` — зеркало `upstream/main`. Сюда не коммитим.
- `vpn1` — наша рабочая ветка поверх **релизного тега** апстрима (сейчас `8.0.0`).
- Выкатываем только по тегам вида `<тег апстрима>-vpn1.<N>`, например `8.0.0-vpn1.1`. По тегу работает откат.
- `upstream` — только для чтения: push туда отключён (`NO_PUSH_TO_UPSTREAM`).
- GitHub Actions в форке выключены: workflow апстрима публикуют образы в его реестр. Включать только осознанно.

## Обновление на новый релиз апстрима

```bash
git fetch upstream --tags
git push origin 'refs/tags/*:refs/tags/*'
git rebase --onto X.Y.Z <старый тег апстрима> vpn1
# проверки и сборка — ниже
git tag X.Y.Z-vpn1.1 && git push origin vpn1 --force-with-lease --tags
```

Перед rebase посмотрите изменения апстрима в `frontend/src/widgets/main/subscription-link` и
`frontend/src/pages/main/ui/components`: наши правки живут там.

## Наши изменения

- Переключатель языка перенесён из низа страницы в шапку, между кнопкой «Получить ссылку» и кнопкой поддержки
  (`subscription-link.widget.tsx`, `main.page.component.tsx`). Меню открывается `bottom-end`, чтобы не вылезать
  за правый край экрана.

## Правила изменений

- Дифф с апстримом держим минимальным, тогда rebase проходит без конфликтов.
- Всё, что настраивается конфигом страницы в панели Remnawave (брендинг, приложения, инструкции, переводы),
  меняем там, а не в коде.
- `Dockerfile` апстрима не трогаем.

## Проверки и сборка

`Dockerfile` копирует уже собранный `frontend/dist`, фронтенд собирается отдельно (как в CI апстрима, Node 24):

```bash
docker run --rm -v "$PWD/frontend:/w" -w /w node:24.22-trixie-slim \
    sh -c 'npm ci && npm run typecheck && npm run lint && npm run start:build'
docker build -t subscription-page-local:<тег> .
```

- `npm run lint` выдаёт одно предупреждение апстрима (`setState` в effect) — оно не наше.
- `oxfmt --check` не проходит и на чистом апстриме. Файлы апстрима не переформатируем: это раздувает дифф.

## Лицензия

GNU AGPL-3.0 (`LICENCE`). Изменённая версия доступна пользователям по сети, поэтому раздел 13 требует давать им
исходный код: этот репозиторий должен быть **публичным**. Секретов, адресов серверов и деталей инфраструктуры сюда
не кладём.
