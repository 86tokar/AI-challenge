# Task 3 — Telegram Learning Bot on n8n: Development Report

## Tools and techniques used

- **n8n Cloud (free trial)** — единственный «бэкенд». Никаких внешних сервисов и подписок.
- **Telegram Bot API** — UI бота: команды `/start`, `/learn <url>`, `/quiz` плюс inline-кнопки для выбора темы и вариантов ответа.
- **OpenAI `gpt-4o-mini`** через узлы `@n8n/n8n-nodes-langchain.chainLlm` + `lmChatOpenAi`. Бесплатных кредитов n8n trial хватает.
- **Code-узлы (JavaScript)** для роутинга апдейтов, чистки HTML, парсинга JSON от модели, валидации ответов и стейт-машины квиза.
- **HTTP Request** на `https://api.telegram.org/bot<token>/sendMessage` для отправки сообщений (вместо нативного Telegram-узла — почему, см. ниже).
- **`$getWorkflowStaticData('global')`** как встроенное хранилище: два словаря `materials[userId]` и `quizzes[userId]`, переживают рестарты.

## Что работало

- Разделение ролей **Teacher / Examiner** через две разные цепочки `chainLlm` с разными промптами и схемами JSON.
- Хранение состояния в `workflowStaticData` с разбивкой по `userId` — никакой внешней БД не понадобилось.
- Одна точка отправки сообщений: все ветки сходятся в `Send Telegram Message`, добавление новой команды = одна Code-нода + одна ветка Switch.
- Inline-клавиатуры через `callback_data` с префиксами `topic:` / `ans:` — `Route` различает их одним switch'ем.
- Валидация ответа по ключу опции + резервная сверка по тексту, чтобы не зависеть от точного совпадения.

## Что не работало (реальные грабли по ходу)

Делал вместе с Copilot, и почти на каждом шаге что-то не запускалось с первого раза. По хронологии:

1. **`/start` молчал, хотя execution был зелёный.**
   Первая версия `Send Telegram Message` была HTTP-нодой с `{{ $credentials.telegramApi.accessToken }}` в URL. В моей версии n8n это выражение возвращает пустую строку, URL получался битый, Telegram отвечал 404, но `neverError: true` гасил ошибку и нода светилась успешно. Фикс: заменил на нативный `Telegram` node.

2. **«Node does not have any credentials set».**
   После каждого `Import from File` нативный Telegram-узел терял привязку credential (n8n матчит credential по ID, а в JSON стоит placeholder). Фикс: после каждого re-import вручную выбирать credential в трёх нодах: `Send Telegram Message`, `OpenAI (Teacher)`, `OpenAI (Examiner)`.

3. **`/learn` падал с `The connection was aborted, perhaps the server is offline`.**
   Узел `Fetch URL` шёл без User-Agent и без следования редиректам, многие сайты резали такие запросы. Фикс: добавил браузерный `User-Agent`, `Accept`, `Accept-Language`, включил `followRedirects` (до 5), поднял timeout до 30 с.

4. **К ответам подклеивалось `This message was sent with n8n…`.**
   Автоприписка n8n на бесплатном плане. Фикс: `appendAttribution: false` (когда был нативный Telegram-узел); после перехода на HTTP Request — проблема исчезла сама.

5. **`/quiz` отвечал «You have no saved materials yet», хотя `/learn` отрабатывал и присылал summary.**
   Самый мучительный пункт. По очереди подозревали:
   - что `staticData` режется n8n Cloud (нет),
   - что разные `userId` у message vs callback (нет),
   - что хранилище в принципе сбрасывается между webhook-исполнениями.

   На самом деле причина была банальная: **workflow не был Published**. В новом UI n8n кнопка активации называется **Publish**, и без неё бот ловит апдейты только через одноразовый `webhook-test` при ручном «Execute Workflow». А там staticData реально не сохраняется между запусками. Фикс: нажать **Publish**, выставить в Settings **Save execution progress = Save**. После этого `/learn` → `/quiz` начал нормально показывать список тем.

   В процессе отладки добавил debug-строку `[debug] your uid=..., known uids=[...]` в «no materials»-ответ — оставил её в коде, реально помогает.

6. **Под сообщением «Your saved topics» не появлялись кнопки.**
   В моей версии n8n у нативного Telegram-узла поле `additionalFields.reply_markup` молча игнорировалось — в Output `Reply Markup` оставался `None`. Фикс: вернулся к HTTP Request и сам собираю тело `sendMessage` (с `reply_markup.inline_keyboard` только когда кнопки реально есть). Это надёжнее всех вариантов с нативным узлом.

7. **После замены на HTTP-ноду снова 404, теперь даже на `/start`.**
   Попытался хранить токен через `predefinedCredentialType: telegramApi` и `$credentials.telegramApi.accessToken` в URL — не подставилось. Фикс: оставил в URL плейсхолдер `PASTE_YOUR_BOT_TOKEN_HERE` и просто вписываю реальный токен прямо в URL ноды. Не идеально с точки зрения секретов, но это единственный способ, который стабильно работает на free-плане без Variables.

8. **Markdown ломал отправку.**
   Заголовки и key points часто содержат `_`, `(`, `*`, и Telegram отдавал `400 can't parse entities`. Фикс: убрал `parse_mode: Markdown`, шлю plain text — читается всё равно нормально, зато никогда не падает.

9. **`Execution Succeeded`, а в Telegram пусто.**
   Из-за `neverError: true` у HTTP-ноды любой ответ Telegram считался успехом, в том числе 404. Понял это, только когда полез в Executions → узел Send Telegram Message → **Output JSON** и увидел там `{ "ok": false, "error_code": 404, "description": "Not Found" }`. С тех пор первое, что проверяю при «бот молчит» — именно Output этой ноды.

## Notable decisions

- **Хранилище — `workflowStaticData`, не внешняя БД.** Ноль настройки, переживает рестарты, разделено по `userId`. Если проект вырастет — следующим шагом мигрировать на встроенные **n8n Data Tables** (структура `materials` / `quizzes` останется той же).
- **Namespace по Telegram `userId`, а не `chatId`** — данные пользователя переходят за ним между DM, группами и темами.
- **HTTP Request вместо нативного Telegram-узла** — единственный способ гарантированно прокинуть `reply_markup` через все ветки на этой версии n8n.
- **Одна `Send Telegram Message` нода на весь workflow** — все ветки сходятся в неё через объект `{ chatId, text, reply_markup }`. Добавление команды = одна Code-нода + одно правило Switch.
- **Жёсткое ограничение в 5 вопросов в коде** (`.slice(0, 5)`) поверх промпта — на случай если модель решит выдать 3 или 8.
- **`gpt-4o-mini`** — качество для «суммаризируй статью» и «напиши 5 MCQ» более чем достаточно, латентность <3 с, бесплатных кредитов n8n хватает.
- **Workflow надо Publish, а не «Execute Workflow».** Очевидно задним числом, но в новом UI это совсем не очевидно — держу это явно в README и здесь.
