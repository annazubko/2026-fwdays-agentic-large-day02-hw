# MCP Server: Playwright Browser — Setup & Demo

## Чому Playwright MCP

З трьох варіантів (GitHub, Context7, Browser) — **Playwright MCP** найнаочніший для цього проекту:

- Не потребує токенів або OAuth
- Миттєво демонструє роботу: Claude відкриває браузер, робить скріншот, аналізує UI
- Прямо релевантний для Excalidraw: можна перевірити live app на localhost:3001
- Встановлення — одна команда, нульова конфігурація

---

## Налаштування (зроблено)

### 1. Project-level `.mcp.json` (вже у корені проекту)

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

Цей файл зберігається в репозиторії і підхоплюється автоматично усіма членами команди.

### 2. Альтернатива — user-level (особисте, для всіх проектів)

```bash
claude mcp add --scope user --transport stdio playwright -- npx -y @playwright/mcp@latest
```

### 3. Перевірка що сервер підключено

```bash
# У терміналі (поза сесією Claude Code):
claude mcp list

# Очікуваний вивід:
# playwright: npx -y @playwright/mcp@latest (stdio)
```

---

## Demo-сценарій

### Передумова

Запустити dev-сервер Excalidraw:

```bash
yarn start
# → http://localhost:3001
```

### Запит до Claude Code

```
Use the playwright browser to open http://localhost:3001, take a screenshot,
and tell me what UI elements are visible on the canvas toolbar.
```

### Що відбудеться

1. Claude запускає Playwright browser (Chromium headless)
2. Відкриває `http://localhost:3001`
3. Робить скріншот
4. Аналізує UI і перераховує елементи тулбару

### Очікуваний результат

```
I can see the Excalidraw canvas with the following toolbar elements:
- Selection tool (arrow icon)
- Rectangle tool
- Diamond/rhombus tool
- Ellipse tool
- Arrow tool
- Line tool
- Freehand draw tool
- Text tool
- Image tool
- Eraser tool
...
```

### Додаткові demo-запити

```
# Перевірити що build працює в браузері
Use playwright to navigate to http://localhost:5001 (production build)
and check if the page loads without console errors.

# Знайти проблему в UI
Use playwright to open http://localhost:3001, click the rectangle tool,
draw a rectangle on the canvas, and take a screenshot of the result.

# Перевірити responsive
Use playwright to open http://localhost:3001 at mobile viewport (375x812)
and screenshot the toolbar — check if it's responsive.
```

---

## Як показати проверяющему що MCP реально працює

1. **Показати `.mcp.json`** у корені репо — конфіг зафіксований
2. **Запустити `yarn start`** щоб dev-сервер був живий
3. **Написати в Claude Code:**
   ```
   Use playwright to open http://localhost:3001 and take a screenshot
   ```
4. **Claude відкриє браузер** і поверне скріншот + опис UI
5. **Опціонально** — показати в терміналі:
   ```bash
   claude mcp list
   # playwright: stdio (running)
   ```

---

## Що це дає для проекту Excalidraw

| Сценарій | Як використовувати MCP |
|----------|----------------------|
| Регресійне тестування UI | Screenshot before/after змін в App.tsx |
| Перевірка production build | Відкрити localhost:5001 після `yarn build` |
| Debug collaboration flow | Відкрити два вікна, ввести room URL |
| Accessibility audit | Playwright accessibility snapshot тулбару |
| Visual diff компонентів | Скріншот до та після зміни LayerUI.tsx |