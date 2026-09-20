# logger

[![CI](https://github.com/alya-lang/logger/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/logger/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/logger?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Flogger%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Flogger%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Structured logging, leveled output, JSON format, and size-based file rotation for Alya applications.

---

## 🌟 Features

- ⚡ **High Performance**: Micro-benchmarked at ~1 µs per text log and 500 ns on filtered-out log rejections.
- 🎯 **Six Leveled Channels**: Full hierarchy including `TRACE (0)`, `DEBUG (1)`, `INFO (2)`, `WARN (3)`, `ERROR (4)`, `FATAL (5)`, and `OFF (6)`.
- 🗂️ **Structured Context Fields**: Attach key-value metadata to logger instances or individual call-sites.
- 📊 **Dual Formatting**: Human-friendly ANSI color badges in text mode, or compact escaped JSON objects for log aggregators (ELK, Datadog, CloudWatch).
- 🔄 **Size-Based File Rotation**: Built-in rolling file appender automatically archives logs once a size limit is reached, maintaining a configurable number of backup files (`.1`, `.2`, ...).
- 🔀 **Multi-Appender Architecture**: Fan-out log entries simultaneously across console output, static log files, and rolling log archives.

---

## 📁 Project Architecture

```
logger/
├── alya.toml                  # Package manifest (v0.1.0)
├── src/
│   ├── lib.alya               # Public API facade & factory helpers
│   ├── types.alya             # Structs, constants, level getters
│   └── core/
│       ├── utils.alya         # ANSI stripping, JSON escaping, file helpers
│       ├── levels.alya        # Level conversions and ANSI badges
│       ├── formatters.alya    # Text and JSON formatters
│       ├── appenders.alya     # Console, File, RollingFile appenders
│       └── logger.alya        # Core logger dispatching & methods
├── examples/
│   └── demo.alya              # Full feature demonstration
├── tests/
│   ├── test_basic.alya        # Core logger lifecycle & setters
│   ├── test_levels.alya       # Level parsing, badges, and filters
│   ├── test_json.alya         # JSON serialization & string escaping
│   ├── test_appenders.alya    # Console and File appenders
│   └── test_rotation.alya     # Size threshold & file rotation limits
└── benches/
    └── bench_basic.alya       # Micro-benchmark suite
```

---

## 📦 Installation

Add `logger` to your project's `alya.toml`:

```toml
[dependencies]
logger = { git = "https://github.com/alya-lang/logger", branch = "main" }
```

Or install it directly via the Alya CLI:

```bash
alya add logger --git https://github.com/alya-lang/logger --branch main
alya install
```

---

## 🚀 Quick Start

### 1. Quick Global Logging

```alya
import "logger" as log

function main()
    log::log_info("Application service started")
    log::log_warn("Database connection pool is 80% full")
    log::log_error("Failed to query cache cluster")
end

main()
```

### 2. Structured Component Logger with Timestamps

```alya
import "logger" as log

function main()
    let app_log = log::create("ApiGateway", log::LogLevel.Info)
    app_log.set_timestamps(1)

    app_log.info("Server listening on 0.0.0.0:8080")
    app_log.debug("This debug trace is skipped at INFO level")
end

main()
```

### 3. Structured Key-Value Fields

```alya
import "logger" as log

function main()
    let auth_log = log::create("AuthService", log::LogLevel.Info)
    auth_log.set_timestamps(1)
    auth_log.add_field("env", "production")

    let call_fields = [
        log::field("user_id", "4092"),
        log::field("ip", "10.0.4.15")
    ]
    auth_log.info_fields("User session authenticated", call_fields)
end

main()
```

*Output:*
```text
[21:40:12] [AuthService] [INFO]  User session authenticated env=production user_id=4092 ip=10.0.4.15
```

### 4. JSON Formatted Logs

```alya
import "logger" as log

function main()
    let json_log = log::create("Telemetry", log::LogLevel.Info)
    json_log.set_format(log::LogFormat.Json)

    let event_fields = [
        log::field("latency_ms", "28"),
        log::field("status", "200")
    ]
    json_log.info_fields("HTTP transaction completed", event_fields)
end

main()
```

*Output:*
```json
{"level":"INFO","time":1789157665,"target":"Telemetry","msg":"HTTP transaction completed","fields":{"latency_ms":"28","status":"200"}}
```

### 5. Appenders & Log Rotation

A single logger can write to multiple output streams simultaneously.

#### Multi-Appender Setup

```alya
import "logger" as log

function main()
    let multi_log = log::create("App", log::LogLevel.Info)
    multi_log.add_console()
    multi_log.add_file("logs/app.log")

    multi_log.info("Written to both console and static log file")
end

main()
```

#### Rolling File Appender

Automatically archives old logs once the file size reaches a specified byte threshold:

```alya
import "logger" as log

function main()
    let rot_log = log::create("App", log::LogLevel.Info)
    
    # Rotate at 10 MB (10485760 bytes), keep 5 backups (app.log.1 ... app.log.5)
    rot_log.add_rolling_file("logs/app.log", 10485760, 5)

    rot_log.info("Service transaction record")
end

main()
```

---

## 📖 API Reference

### Enums & Types

| Symbol | Type | Description |
|---|---|---|
| `LogLevel` | `enum` | Leveled logging hierarchy (`Trace`, `Debug`, `Info`, `Warn`, `Error`, `Fatal`, `Off`). |
| `LogFormat` | `enum` | Formatting modes (`Text = 1`, `Json = 2`). |
| `AppenderType` | `enum` | Destination types (`Console = 1`, `File = 2`, `Rolling = 3`). |
| `AppLogger` | `struct` | Main logger container with fluent methods (`info`, `debug`, `add_file`, etc.). |
| `LogField` | `struct` | Key-value contextual field pair (`fld_key`, `fld_value`). |
| `LogRecord` | `struct` | Individual log event metadata container. |
| `Appender` | `struct` | Output destination configuration. |

### Logger Construction & Settings

| Function | Arguments | Description |
|---|---|---|
| `create(name, level)` | `name = "", level = 2` | Creates a new Logger instance |
| `logger_new(name, level)` | `name = "", level = 2` | Constructor matching `std/log` convention |
| `logger_set_level(l, level)` | `l: Logger, level: int` | Sets the minimum threshold level |
| `logger_set_colored(l, colored)` | `l: Logger, colored: 0 \| 1` | Enables/disables ANSI colors |
| `logger_set_timestamps(l, enabled)` | `l: Logger, enabled: 0 \| 1` | Enables/disables `[HH:MM:SS]` timestamps |
| `logger_set_format(l, format)` | `l: Logger, format: 1 \| 2` | Sets text (`1`) or JSON (`2`) formatting |
| `logger_set_file(l, path)` | `l: Logger, path: string` | Sets destination file path |
| `logger_add_console(l)` | `l: Logger` | Registers stdout console appender |
| `logger_add_file(l, path)` | `l: Logger, path: string` | Registers static file appender |
| `logger_add_rolling_file(l, path, max_bytes, max_files)` | `l: Logger, ...` | Registers size-based rolling appender |
| `logger_add_field(l, key, value)` | `l: Logger, key, val` | Adds persistent context field |
| `logger_clear_appenders(l)` | `l: Logger` | Clears all registered appenders |
| `logger_clear_fields(l)` | `l: Logger` | Clears persistent context fields |

### Logging Methods

| Function | Arguments | Description |
|---|---|---|
| `logger_log(l, level, msg)` | `l: Logger, level: int, msg: string` | Logs message at specific level |
| `logger_trace(l, msg)` | `l: Logger, msg: string` | Logs at TRACE level (`0`) |
| `logger_debug(l, msg)` | `l: Logger, msg: string` | Logs at DEBUG level (`1`) |
| `logger_info(l, msg)` | `l: Logger, msg: string` | Logs at INFO level (`2`) |
| `logger_warn(l, msg)` | `l: Logger, msg: string` | Logs at WARN level (`3`) |
| `logger_error(l, msg)` | `l: Logger, msg: string` | Logs at ERROR level (`4`) |
| `logger_fatal(l, msg)` | `l: Logger, msg: string` | Logs at FATAL level (`5`) |
| `logger_info_fields(l, msg, fields)` | `l: Logger, msg: string, fields: array` | Logs with call-site structured fields |

### Global Helper Functions

| Function | Arguments | Description |
|---|---|---|
| `log_trace(msg)` | `msg: string` | Prints quick colored `[TRACE]` line |
| `log_debug(msg)` | `msg: string` | Prints quick colored `[DEBUG]` line |
| `log_info(msg)` | `msg: string` | Prints quick colored `[INFO ]` line |
| `log_warn(msg)` | `msg: string` | Prints quick colored `[WARN ]` line |
| `log_error(msg)` | `msg: string` | Prints quick colored `[ERROR]` line |
| `log_fatal(msg)` | `msg: string` | Prints quick colored `[FATAL]` line |
| `level_to_str(level)` | `level: int` | Converts numeric level to name string |
| `level_from_str(str)` | `str: string` | Parses level name (case-insensitive) |

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.