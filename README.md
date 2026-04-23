# Scrider Editor

WYSIWYG-редактор с архитектурой **Framework-Agnostic Core + Framework Wrappers**.

Ядро (`@scrider/editor-core`) — чистый TypeScript без зависимостей от DOM и фреймворков.
React-обёртка (`@scrider/editor-react`) — тонкий мост к contentEditable.

---

## Архитектура

```
@scrider/editor-core        ← Данные + чистые функции (State → State)
                               Не знает о DOM, React, Vue
                               Тестируется без браузера, работает в SSR/Worker

@scrider/editor-react        ← React-компонент <ScriderEditor />
                               contentEditable + события + toolbar
                               Зависит от editor-core

demo/                        ← Тестовый стенд (Vite)
                               Интеграционное тестирование, примеры расширений
```

### Разделение ответственности

|                  | `@scrider/editor-core`            | `@scrider/editor-react`            |
|------------------|-----------------------------------|-------------------------------------|
| **Знает о**      | Delta, Selection, Commands        | DOM, React, contentEditable         |
| **Зависимости**  | `@scrider/delta`, `@scrider/formatter` | `@scrider/editor-core`, `react` |
| **Состояние**    | EditorState (иммутабельные данные)| React state + refs                  |
| **Функции**      | Чистые: `State → State`          | Event handlers, DOM-манипуляции     |
| **DOM**          | Нет                               | Да                                  |
| **Тестирование** | Unit-тесты без браузера           | Интеграционные тесты с DOM          |
| **SSR/Worker**   | Работает                          | Только браузер                      |

> **Это НЕ Parchment.** Core не создаёт дерева объектов, не зеркалирует DOM.
> Это набор данных и чистых функций — продолжение функциональной парадигмы Scrider.

---

## Возможности

### Форматирование
- **12 inline-форматов:** bold, italic, underline, strike, code, link, color, background, mark, subscript, superscript, kbd
- **11 block-форматов:** header (h1–h6), blockquote, code-block, list (bullet/ordered/checklist), align, indent, table
- **Embed:** image, video, formula (KaTeX/MathJax), divider, diagram
- **Font Family и Font Size** — inline-атрибуты Delta с dropdown в toolbar

### Ввод и редактирование
- **Two-Mode Architecture:** browser-driven (нативный ввод без перерисовки DOM) + editor-driven (форматирование/вставка с полной перерисовкой)
- **Input Rules:** автозамена при вводе (`# ` → заголовок, `- ` → список, `1. ` → нумерованный список)
- **Inline Markdown Triggers:** `**text**` → bold, `*text*` → italic, `` `code` `` → code, `~~text~~` → strikethrough
- **LaTeX/Markdown Paste:** при Ctrl+V автоматическая конвертация `\(...\)` → `$...$`, `\[...\]` → `$$...$$`, markdown-разметки
- **Undo/Redo** через OT (invert + compose)
- **Ctrl+A** — выделение только содержимого редактора

### Toolbar
- JSON-конфигурация с именованными группами
- Адаптивная верстка (overflow menu при уменьшении ширины)
- Light/dark темы через CSS Custom Properties
- Кастомные кнопки и dropdown через Module/Plugin API

### Экспорт
- **PDF** — через pdfkit с поддержкой шрифтов (Times New Roman и др.), inline KaTeX-формул (MathJax SVG), justify-выравнивания, красной строки, межстрочного интервала
- **DOCX** — через docx.js
- **HTML, Markdown, Plain Text, Delta JSON**

### Расширяемость
- **ScriderRegistry** — глобальный реестр модулей и плагинов
- **ModuleRegistry** — расширения toolbar (кнопки, dropdown, группы)
- **PluginRegistry** — команды с side effects, post-render hooks, lifecycle
- **CodeBlockRenderer** — паттерн для code-block рендереров (Prism, KaTeX, Mermaid, PlantUML)
- **Декларативный merge** (`extends`) — patch-конфигурация на любом уровне

---

## Структура репозитория

```
scrider-editor/
├── editor-core/         @scrider/editor-core (TypeScript, 0 DOM-зависимостей)
│   ├── src/
│   │   ├── state/       EditorState, HistoryState, SelectionRange
│   │   ├── commands/    inline, block, structure, clipboard, registry
│   │   ├── queries/     getActiveFormats, isFormatActive, canUndo/canRedo
│   │   ├── keymap/      KeyMap типы + дефолтный маппинг
│   │   ├── input-rules/ InputRule типы + дефолтные правила
│   │   └── registry/    ScriderRegistry, ModuleRegistry, PluginRegistry
│   └── tests/
│
├── editor-react/        @scrider/editor-react (React 18+)
│   ├── src/
│   │   ├── components/  ScriderEditor, Toolbar, ToolbarConnected
│   │   ├── hooks/       useScriderEditor
│   │   └── styles/      CSS с Custom Properties
│   └── tests/
│
├── demo/                Vite-приложение (тестовый стенд)
│   └── src/
│       ├── pages/       EditorPage (основная демо-страница)
│       └── extensions/  export-pro, inline-ext и др.
│
└── docs/                Документация (концепт, план, архитектура)
```

> Это **не** монорепо. Каждый пакет — автономный проект со своим `package.json` и `pnpm-lock.yaml`.
> Пакеты связаны через `link:` для локальной разработки.

---

## Быстрый старт

### Требования
- Node.js 18+
- pnpm

### Установка и запуск

```bash
# Установка зависимостей (каждый пакет отдельно)
cd editor-core && pnpm install
cd ../editor-react && pnpm install
cd ../demo && pnpm install

# Сборка библиотек
cd ../editor-core && pnpm build
cd ../editor-react && pnpm build

# Запуск демо
cd ../demo && pnpm dev
```

### Тесты и линтер

```bash
# editor-core
cd editor-core
pnpm test:run    # unit-тесты (Vitest)
pnpm lint        # ESLint

# editor-react
cd ../editor-react
pnpm test:run
pnpm lint
```

---

## Использование

```tsx
import { ScriderEditor, defaultToolbar } from '@scrider/editor-react';
import '@scrider/editor-react/styles.css';

function App() {
  return (
    <ScriderEditor
      toolbar={defaultToolbar}
      theme="light"
      placeholder="Начните вводить текст..."
      onChange={(delta) => console.log(delta)}
    />
  );
}
```

---

## Стек технологий

| Пакет | Технологии |
|-------|------------|
| `editor-core` | TypeScript strict, tsup (ESM + CJS), Vitest, ESLint |
| `editor-react` | React 18+, TypeScript, tsup, Vitest + jsdom |
| `demo` | Vite, React, KaTeX, Prism.js, pdfkit, docx.js |
| Общее | pnpm, Husky, lint-staged, Prettier |

### Зависимости экосистемы Scrider
- [`@scrider/delta`](https://www.npmjs.com/package/@scrider/delta) — OT Delta (Operational Transformation)
- [`@scrider/formatter`](https://www.npmjs.com/package/@scrider/formatter) — Delta ↔ HTML/Markdown конверсия, схема форматов

---

## Фазы разработки

| Phase | Название | Статус |
|-------|----------|--------|
| 0 | Setup + Demo | ✅ |
| 1 | Core: State & Commands | ✅ |
| 2 | React: базовый редактор | ✅ |
| 3 | Toolbar | ✅ |
| 4 | Темы и полировка | ✅ |
| 5 | Архитектура реестров | ✅ |
| 6 | Plugin-система + Export + Font | ✅ |
| 7 | Архитектура расширений + пилотные плагины | — |
| 8 | Полнота контента (все 31 формат) | — |
| 9 | Polish & Publish | — |

---

## Лицензия

Proprietary. Все права защищены.
