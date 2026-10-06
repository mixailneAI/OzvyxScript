
# OzvyxScript

**OzvyxScript** — язык программирования общего назначения со статической типизацией, безопасностью памяти без сборщика мусора и мгновенной диагностикой ошибок в редакторе.

**English** · [Русский](#русская-версия)

---

## English

### Overview

OzvyxScript is a general-purpose programming language designed for systems programming, application development, backend services, embedded systems, and scripting. It combines static typing with full type inference, memory safety without a garbage collector, an expressive but minimal syntax, and live error diagnostics in the editor through a built-in Language Server Protocol implementation.

The language is designed around a simple idea: a program should be easy to read, safe by default, fast at runtime, and pleasant to write. These goals are not new on their own — the design of OzvyxScript is an attempt to combine them without compromise, drawing on the strongest ideas from existing languages while avoiding their historical baggage.

- **Version:** 1.0.0
- **License:** MIT
- **File extension:** `.yx`
- **Compiler:** OzvyxKompilator
- **Command:** `ozvk`
- **Author:** [mixailneAI](https://github.com/mixailneAI)
- **Repository:** [github.com/mixailneAI/OzvyxScript](https://github.com/mixailneAI/OzvyxScript)

### Design goals

1. **Memory safety without a garbage collector by default.** Ownership and borrow checking, with lifetimes inferred automatically in the vast majority of code. An optional garbage-collected mode (`--gc`) is available for prototyping and scripting.

2. **Static typing with full type inference.** Hindley–Milner inference with bidirectional checking. Type annotations are optional in most cases but always available when clarity demands them.

3. **Live diagnostics.** Errors are shown while you type, not only at compile time. This is provided by a Language Server Protocol server that lives inside the compiler itself.

4. **One tool for everything.** `ozvk` is the compiler, package manager, REPL, formatter, test runner, documentation generator, and LSP server in a single binary. No separate tools, no plugin ecosystems, no version drift.

5. **Stability.** The core syntax, keywords, and type system are frozen. Code written against version 1.0.0 is intended to compile against every future version without modification.

6. **Self-hosting.** The compiler is intended to be written in OzvyxScript itself. This places a high bar on the language's expressiveness and ensures that the language remains usable for large, complex software.

7. **Readability over brevity.** Keywords are words, not punctuation. Explicit is better than implicit. Code should be understandable to a reader who has never seen the project before.

8. **No null, no exceptions.** The absence of a value is expressed with `Option[T]`. Failures are expressed with `Result[T, E]`. All error handling is part of the type system and visible in function signatures.

### Syntax

OzvyxScript uses curly braces for blocks, newlines for statement separation, and parentheses only where they add clarity. There are no mandatory semicolons. There is no `end` keyword. There is no significant whitespace.

```ozvyx
fn fizzbuzz(n: Int) -> String {
    match (n % 3, n % 5) {
        when (0, 0) => "FizzBuzz",
        when (0, _) => "Fizz",
        when (_, 0) => "Buzz",
        when _     => n.to_string(),
    }
}

for i in 1..=100 {
    println(fizzbuzz(i))
}
```

### Keywords

Core keywords:

```
fn  let  mut  ref  if  else  while  for  in  loop
break  continue  return  match  when  struct  enum
trait  impl  mod  use  pub  const  type  true  false
self  Self  as  async  await  and  or  not
```

Extended keywords:

```
where  dyn  move  copy  unsafe  extern
```

### Type system

- Static typing with full type inference (Hindley–Milner with bidirectional checking).
- Nominal types for `struct` and `enum`; structural types for `trait`.
- No implicit type coercions.
- Generic types with monomorphization.
- Sum types (`enum`) with payloads and exhaustive pattern matching.
- `Option[T]` for absence, `Result[T, E]` for fallibility.
- Traits with associated functions and operator overloading.

### Memory model

- Ownership by default, with borrow checking.
- Lifetimes inferred automatically in the vast majority of code.
- Explicit lifetime annotations only when the inference cannot resolve a case.
- Optional garbage-collected mode (`--gc`) for prototyping.
- Zero-cost abstractions in the default mode.
- FFI via `extern "C"`.

### Concurrency and asynchronous code

- `async fn` and `await` are first-class language constructs.
- Structured concurrency with task handles.
- No built-in runtime — the async executor is a library module, not part of the language.
- Threads and channels provided by the standard library.

### Error handling

- No exceptions. No `try`/`catch`.
- Errors are values, expressed as `Result[T, E]`.
- The `?` operator propagates errors upward.
- Compiler diagnostics use stable error codes (`OZX0001` … `OZX9999`) that can be searched in the documentation.

### A first example

```ozvyx
println("Hello, World!")
```

Top-level code is the implicit entry point. For larger programs, an explicit `fn main()` is used.

### More examples

Reading a file and processing lines:

```ozvyx
use std::fs

fn count_non_empty(path: String) -> Result[U64, IoError] {
    let text = fs::read_to_string(path)?
    let mut count = 0
    for line in text.lines() {
        if line.trim().len() > 0 {
            count += 1
        }
    }
    Ok(count)
}
```

Async HTTP:

```ozvyx
async fn fetch_title(url: String) -> Result[String, HttpError] {
    let body = await http::get_string(url)?
    let title = html::select_first(body, "title")?
    Ok(title)
}
```

Pipeline and higher-order functions:

```ozvyx
let result = users
    |> filter(fn(u) => u.age >= 18)
    |> map(fn(u) => u.name)
    |> sort()
    |> join(", ")
```

Pattern matching on enums:

```ozvyx
enum Shape {
    Circle(Float),
    Rect(Float, Float),
    Point,
}

fn area(s: Shape) -> Float {
    match s {
        when Shape::Circle(r)  => 3.14159 * r * r,
        when Shape::Rect(w, h) => w * h,
        when Shape::Point      => 0.0,
    }
}
```

Traits and generics:

```ozvyx
trait Drawable {
    fn draw(self)
}

fn render_all[T: Drawable](items: List[T]) {
    for item in items {
        item.draw()
    }
}
```

### The compiler — OzvyxKompilator

The compiler is delivered as a single binary called `ozvk`. It includes everything needed to build, run, test, document, and publish OzvyxScript projects.

Commands:

```
$ ozvk new my_project      create a new project
$ ozvk build               build the project
$ ozvk run                 build and run
$ ozvk repl                interactive session
$ ozvk test                run tests
$ ozvk add http            add a dependency
$ ozvk remove http         remove a dependency
$ ozvk update              update dependencies
$ ozvk lsp                 start the language server
$ ozvk fmt                 format source files
$ ozvk doc                 generate documentation
$ ozvk check               type-check without building
$ ozvk publish             publish a package
$ ozvk --version           print version
$ ozvk --help              print help
```

Compilation pipeline:

```
.yx source
   │
   ▼
Lexer → Parser → AST → HIR → Type Checker → MIR → Borrow Checker → Optimizer → Codegen
                                                                                    │
                                                                                    ▼
                                                                    LLVM IR | C | WASM | ASM
```

Each stage is a pure function returning either a result or a list of diagnostics. The same pipeline is used by the command-line interface, the REPL, and the LSP server, which is why diagnostics are identical in all three contexts.

Incremental compilation: the compiler uses a query-based architecture with memoization. When a file changes, only the queries that depend on it are invalidated and recomputed. If a recomputed result is identical to the previous one, dependent queries are not invalidated at all.

Live diagnostics: the LSP server inside `ozvk` uses the same query layer as the command-line compiler. Diagnostics are published to the editor as the user types, with the same error codes, positions, and suggestions that would appear in a terminal build.

Target backends:

| Backend | Purpose |
|---------|---------|
| LLVM IR | Primary backend for production builds |
| C | Bootstrap backend; portability to platforms without LLVM |
| WASM | WebAssembly target |
| ASM | Assembly output for inspection |

Package manager: the package manager is built into `ozvk`. A project is described by a manifest file (`ozvk.toml`) and a lockfile (`ozvk.lock`). Dependencies are downloaded from the community registry. Publication is one command: `ozvk publish`.

### Standard library

The standard library is written in OzvyxScript itself and covers:

- `io` — input and output streams
- `fs` — filesystem access
- `math` — numeric operations
- `string` — string manipulation and Unicode
- `collections` — `List`, `Map`, `Set`, and related types
- `net` — TCP, UDP, and HTTP
- `time` — clocks, durations, dates
- `sync` — threads, channels, mutexes
- `result` — utilities for `Result` and `Option`
- `json` — JSON parsing and serialization

### Editor support

- Visual Studio Code — extension provided in the repository
- Neovim — via the built-in LSP server
- JetBrains IDEs — plugin planned
- Syntax highlighting — TextMate grammar and Tree-sitter grammar

### Documentation

- `docs/spec/philosophy.md` — design philosophy in detail
- `docs/spec/language.md` — complete language specification
- `docs/spec/grammar.ebnf` — formal EBNF grammar
- `docs/OZVYXKOMPILATOR.md` — compiler documentation
- `docs/roadmap.md` — development roadmap
- `docs/faq.md` — frequently asked questions
- `docs/tutorials/` — step-by-step guides

### Project status

Version 1.0.0 — design phase.

The language foundation is being specified. The compiler is not yet implemented. The repository currently contains the specification, the architecture, and the design documentation.

### Contributing

Contributions are welcome in any of the following areas:

- Compiler development — Rust, compiler theory, LLVM
- Standard library — writing `std` in OzvyxScript
- Documentation — specifications, tutorials, guides
- Editor integrations — VS Code, Neovim, JetBrains, Emacs
- Test suites — specification cases, runtime tests, UI tests
- Design — logo, website, branding

Before contributing, please read the philosophy and the language specification. Substantial changes are discussed in issues or discussions first. All contributions are expected to follow the project's stability guarantees — do not propose changes that break existing syntax or semantics.

### License

MIT License. See [LICENSE](LICENSE) for details.

### Support the project

- Star the repository — it helps the project be discovered.
- Read the documentation and share feedback.
- Report issues or unclear parts of the specification.
- Join discussions — language design proposals are welcome.
- Contribute code, documentation, tests, or design.

Repository: [github.com/mixailneAI/OzvyxScript](https://github.com/mixailneAI/OzvyxScript)

With love from mixailneAI.

---

## Русская версия

### Обзор

OzvyxScript — язык программирования общего назначения для системного программирования, разработки приложений, backend-сервисов, встраиваемых систем и написания скриптов. Он сочетает статическую типизацию с полным выводом типов, безопасность памяти без сборщика мусора, выразительный, но минимальный синтаксис и мгновенную диагностику ошибок в редакторе через встроенную реализацию Language Server Protocol.

Язык проектируется вокруг простой идеи: программа должна быть легко читаемой, безопасной по умолчанию, быстрой во время исполнения и приятной в написании. По отдельности эти цели не новы — дизайн OzvyxScript представляет собой попытку соединить их без компромиссов, опираясь на сильнейшие идеи существующих языков и избегая их исторического багажа.

- **Версия:** 1.0.0
- **Лицензия:** MIT
- **Расширение файлов:** `.yx`
- **Компилятор:** OzvyxKompilator
- **Команда:** `ozvk`
- **Автор:** [mixailneAI](https://github.com/mixailneAI)
- **Репозиторий:** [github.com/mixailneAI/OzvyxScript](https://github.com/mixailneAI/OzvyxScript)

### Цели дизайна

1. **Безопасность памяти без сборщика мусора по умолчанию.** Владение и заимствование, времена жизни выводятся автоматически в подавляющем большинстве кода. Опциональный режим со сборщиком мусора (`--gc`) доступен для прототипирования и написания скриптов.

2. **Статическая типизация с полным выводом.** Вывод типов по Хиндли–Милнеру с двунаправленной проверкой. Аннотации типов необязательны в большинстве случаев, но всегда доступны, когда нужна ясность.

3. **Живая диагностика.** Ошибки видны в момент набора кода, а не только при компиляции. Это обеспечивается сервером Language Server Protocol, встроенным в сам компилятор.

4. **Один инструмент для всего.** `ozvk` — это компилятор, пакетный менеджер, REPL, форматтер, тест-раннер, генератор документации и LSP-сервер в одном бинарнике. Никаких отдельных инструментов, никаких экосистем плагинов, никакого расхождения версий.

5. **Стабильность.** Базовый синтаксис, ключевые слова и система типов заморожены. Код, написанный против версии 1.0.0, должен собираться с любой будущей версией без изменений.

6. **Самохостинг.** Компилятор пишется на самом OzvyxScript. Это задаёт высокую планку выразительности языка и гарантирует, что язык остаётся пригодным для крупного, сложного программного обеспечения.

7. **Читаемость важнее краткости.** Ключевые слова — это слова, а не знаки пунктуации. Явное лучше неявного. Код должен быть понятен читателю, который никогда не видел проект раньше.

8. **Никакого null, никаких исключений.** Отсутствие значения выражается через `Option[T]`. Ошибки — через `Result[T, E]`. Вся обработка ошибок входит в систему типов и видна в сигнатурах функций.

### Синтаксис

OzvyxScript использует фигурные скобки для блоков, переводы строк для разделения операторов и круглые скобки только там, где они добавляют ясность. Обязательных точек с запятой нет. Ключевого слова `end` нет. Значимых отступов нет.

```ozvyx
fn fizzbuzz(n: Int) -> String {
    match (n % 3, n % 5) {
        when (0, 0) => "FizzBuzz",
        when (0, _) => "Fizz",
        when (_, 0) => "Buzz",
        when _     => n.to_string(),
    }
}

for i in 1..=100 {
    println(fizzbuzz(i))
}
```

### Ключевые слова

Основные:

```
fn  let  mut  ref  if  else  while  for  in  loop
break  continue  return  match  when  struct  enum
trait  impl  mod  use  pub  const  type  true  false
self  Self  as  async  await  and  or  not
```

Расширенные:

```
where  dyn  move  copy  unsafe  extern
```

### Система типов

- Статическая типизация с полным выводом (Хиндли–Милнер с двунаправленной проверкой).
- Номинальные типы для `struct` и `enum`; структурные для `trait`.
- Никаких неявных приведений типов.
- Дженерики с мономорфизацией.
- Суммарные типы (`enum`) с payload и исчерпывающим pattern matching.
- `Option[T]` для отсутствия значения, `Result[T, E]` для возможности ошибки.
- Трейты с ассоциированными функциями и перегрузкой операторов.

### Модель памяти

- Владение по умолчанию с проверкой заимствований.
- Времена жизни выводятся автоматически в подавляющем большинстве кода.
- Явные аннотации времён жизни нужны только там, где вывод не справляется.
- Опциональный режим со сборщиком мусора (`--gc`) для прототипирования.
- Zero-cost абстракции в основном режиме.
- FFI через `extern "C"`.

### Параллелизм и асинхронность

- `async fn` и `await` — первоклассные конструкции языка.
- Структурный параллелизм с хендлами задач.
- Встроенного рантайма нет — async-исполнитель это библиотечный модуль, а не часть языка.
- Потоки и каналы предоставляются стандартной библиотекой.

### Обработка ошибок

- Никаких исключений. Никаких `try`/`catch`.
- Ошибки — это значения, выраженные через `Result[T, E]`.
- Оператор `?` пробрасывает ошибку наверх.
- Диагностики компилятора используют стабильные коды ошибок (`OZX0001` … `OZX9999`), которые можно искать в документации.

### Первый пример

```ozvyx
println("Hello, World!")
```

Код верхнего уровня — неявная точка входа. Для больших программ используется явный `fn main()`.

### Больше примеров

Чтение файла и обработка строк:

```ozvyx
use std::fs

fn count_non_empty(path: String) -> Result[U64, IoError] {
    let text = fs::read_to_string(path)?
    let mut count = 0
    for line in text.lines() {
        if line.trim().len() > 0 {
            count += 1
        }
    }
    Ok(count)
}
```

Асинхронный HTTP:

```ozvyx
async fn fetch_title(url: String) -> Result[String, HttpError] {
    let body = await http::get_string(url)?
    let title = html::select_first(body, "title")?
    Ok(title)
}
```

Конвейер и функции высшего порядка:

```ozvyx
let result = users
    |> filter(fn(u) => u.age >= 18)
    |> map(fn(u) => u.name)
    |> sort()
    |> join(", ")
```

Pattern matching по enum:

```ozvyx
enum Shape {
    Circle(Float),
    Rect(Float, Float),
    Point,
}

fn area(s: Shape) -> Float {
    match s {
        when Shape::Circle(r)  => 3.14159 * r * r,
        when Shape::Rect(w, h) => w * h,
        when Shape::Point      => 0.0,
    }
}
```

Трейты и дженерики:

```ozvyx
trait Drawable {
    fn draw(self)
}

fn render_all[T: Drawable](items: List[T]) {
    for item in items {
        item.draw()
    }
}
```

### Компилятор — OzvyxKompilator

Компилятор поставляется одним бинарником `ozvk`. Он включает всё необходимое для сборки, запуска, тестирования, документирования и публикации проектов на OzvyxScript.

Команды:

```
$ ozvk new my_project      создать новый проект
$ ozvk build               собрать проект
$ ozvk run                 собрать и запустить
$ ozvk repl                интерактивная сессия
$ ozvk test                запустить тесты
$ ozvk add http            добавить зависимость
$ ozvk remove http         удалить зависимость
$ ozvk update              обновить зависимости
$ ozvk lsp                 запустить языковой сервер
$ ozvk fmt                 отформатировать исходники
$ ozvk doc                 сгенерировать документацию
$ ozvk check               проверить типы без сборки
$ ozvk publish             опубликовать пакет
$ ozvk --version           вывести версию
$ ozvk --help              вывести справку
```

Конвейер компиляции:

```
.yx источник
   │
   ▼
Lexer → Parser → AST → HIR → Type Checker → MIR → Borrow Checker → Optimizer → Codegen
                                                                                    │
                                                                                    ▼
                                                                    LLVM IR | C | WASM | ASM
```

Каждый этап — чистая функция, возвращающая либо результат, либо список диагностик. Один и тот же конвейер используется и командной строкой, и REPL, и LSP-сервером — поэтому диагностики выглядят одинаково во всех трёх контекстах.

Инкрементальная компиляция: компилятор использует query-based архитектуру с мемоизацией. Когда файл меняется, инвалидируются только зависящие от него запросы. Если пересчитанный результат совпадает с предыдущим, зависимые запросы не инвалидируются вовсе.

Живая диагностика: LSP-сервер внутри `ozvk` использует тот же query-слой, что и командный компилятор. Диагностики публикуются в редактор по мере набора, с теми же кодами ошибок, позициями и подсказками, что появились бы в терминале.

Целевые бэкенды:

| Бэкенд | Назначение |
|--------|-----------|
| LLVM IR | Основной бэкенд для продакшн-сборок |
| C | Bootstrap-бэкенд; портируемость на платформы без LLVM |
| WASM | Целевая платформа WebAssembly |
| ASM | Ассемблерный вывод для инспекции |

Пакетный менеджер: встроен в `ozvk`. Проект описывается манифестом (`ozvk.toml`) и lock-файлом (`ozvk.lock`). Зависимости скачиваются из общественного реестра. Публикация — одна команда: `ozvk publish`.

### Стандартная библиотека

Стандартная библиотека написана на самом OzvyxScript и покрывает:

- `io` — потоки ввода-вывода
- `fs` — доступ к файловой системе
- `math` — числовые операции
- `string` — работа со строками и Unicode
- `collections` — `List`, `Map`, `Set` и связанные типы
- `net` — TCP, UDP, HTTP
- `time` — часы, длительности, даты
- `sync` — потоки, каналы, мьютексы
- `result` — утилиты для `Result` и `Option`
- `json` — парсинг и сериализация JSON

### Поддержка редакторов

- Visual Studio Code — расширение в репозитории
- Neovim — через встроенный LSP-сервер
- JetBrains IDE — плагин запланирован
- Поддержка подсветки — TextMate grammar и Tree-sitter grammar

### Документация

- `docs/spec/philosophy.md` — философия дизайна подробно
- `docs/spec/language.md` — полная спецификация языка
- `docs/spec/grammar.ebnf` — формальная EBNF-грамматика
- `docs/OZVYXKOMPILATOR.md` — документация компилятора
- `docs/roadmap.md` — дорожная карта
- `docs/faq.md` — часто задаваемые вопросы
- `docs/tutorials/` — пошаговые руководства

### Статус проекта

Версия 1.0.0 — стадия дизайна.

Фундамент языка в процессе спецификации. Компилятор пока не реализован. Репозиторий содержит спецификацию, архитектуру и проектную документацию.

### Участие в проекте

Проект открыт для вклада в любом из направлений:

- Разработка компилятора — Rust, теория компиляторов, LLVM
- Стандартная библиотека — написание `std` на OzvyxScript
- Документация — спецификации, туториалы, руководства
- Интеграции с редакторами — VS Code, Neovim, JetBrains, Emacs
- Тестовые наборы — спецификационные кейсы, runtime-тесты, UI-тесты
- Дизайн — логотип, сайт, брендинг

Перед началом работы рекомендуется прочитать философию и спецификацию языка. Существенные изменения обсуждаются в issues или discussions. Все вклады обязаны следовать гарантиям стабильности проекта — не предлагайте изменения, ломающие существующий синтаксис или семантику.

### Лицензия

Лицензия MIT. Подробности — в файле [LICENSE](LICENSE).

### Поддержать проект

- Поставьте звезду репозиторию — это помогает проекту быть заметным.
- Изучите документацию и поделитесь замечаниями.
- Сообщите о проблемах или неясностях в спецификации.
- Участвуйте в обсуждениях — предложения по языку приветствуются.
- Внесите вклад — код, документация, тесты, дизайн.

Репозиторий: [github.com/mixailneAI/OzvyxScript](https://github.com/mixailneAI/OzvyxScript)

С любовью от mixailneAI.
