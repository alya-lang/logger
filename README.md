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
- 🤝 **100% Backward Compatible**: Drop-in replacement for the `std/log` standard library module.

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
│       ├── logger.alya        # Core logger dispatching & methods
│       └── compat.alya        # 100% std/log compatibility bridge
├── examples/
│   └── demo.alya              # Full feature demonstration
├── tests/
│   ├── test_basic.alya        # Core logger lifecycle & setters
│   ├── test_levels.alya       # Level parsing, badges, and filters
│   ├── test_json.alya         # JSON serialization & string escaping
│   ├── test_appenders.alya    # Console and File appenders
│   ├── test_rotation.alya     # Size threshold & file rotation limits
│   └── test_compat.alya       # std/log backward compatibility tests
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
alyac add logger --git https://github.com/alya-lang/logger --branch main
alyac install
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
    let app_log = log::create("ApiGateway", log::LOG_INFO())
    log::logger_set_timestamps(app_log, 1)

    log::logger_info(app_log, "Server listening on 0.0.0.0:8080")
    log::logger_debug(app_log, "This debug trace is skipped at INFO level")
end

main()
```

### 3. Structured Key-Value Fields

```alya
import "logger" as log

function main()
    let auth_log = log::create("AuthService", log::LOG_INFO())
    log::logger_set_timestamps(auth_log, 1)
    log::logger_add_field(auth_log, "env", "production")

    let call_fields = [
        log::field("user_id", "4092"),
        log::field("ip", "10.0.4.15")
    ]
    log::logger_info_fields(auth_log, "User session authenticated", call_fields)
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
    let json_log = log::create("Telemetry", log::LOG_INFO())
    log::logger_set_format(json_log, log::FORMAT_JSON())

    let event_fields = [
        log::field("latency_ms", "28"),
        log::field("status", "200")
    ]
    log::logger_info_fields(json_log, "HTTP transaction completed", event_fields)
end

main()
```

*Output:*
```json
{"level":"INFO","time":1789157665,"target":"Telemetry","msg":"HTTP transaction completed","fields":{"latency_ms":"28","status":"200"}}
```

---

## 🔄 Appenders & Log Rotation

A single logger can write to multiple output streams simultaneously.

### Multi-Appender Setup

```alya
import "logger" as log

function main()
    let multi_log = log::create("App", log::LOG_INFO())
    log::logger_add_console(multi_log)
    log::logger_add_file(multi_log, "logs/app.log")

    log::logger_info(multi_log, "Written to both console and static log file")
end

main()
```

### Rolling File Appender

Automatically archives old logs once the file size reaches a specified byte threshold:

```alya
import "logger" as log

function main()
    let rot_log = log::create("App", log::LOG_INFO())
    
    # Rotate at 10 MB (10485760 bytes), keep 5 backups (app.log.1 ... app.log.5)
    log::logger_add_rolling_file(rot_log, "logs/app.log", 10485760, 5)

    log::logger_info(rot_log, "Service transaction record")
end

main()
```

---

## 📖 API Reference

### Logger Construction & Settings

| Function | Arguments | Description |
|---|---|---|
| `create(name, level)` | `name = "", level = 2` | Creates a new Logger instance |
| `logger_new(name, level)` | `name = "", level = 2` | Constructor matching `std/log` convention |
| `logger_set_level(l, level)` | `l: Logger, level: int` | Sets the minimum threshold level |
| `logger_set_colored(l, colored)` | `l: Logger, colored: 0 \| 1` | Enables/disables ANSI colors |
| `logger_set_timestamps(l, enabled)` | `l: Logger, enabled: 0 \| 1` | Enables/disables `[HH:MM:SS]` timestamps |
| `logger_set_format(l, format)` | `l: Logger, format: 1 \| 2` | Sets text (`1`) or JSON (`2`) formatting |
| `logger_set_file(l, path)` | `l: Logger, path: string` | Sets legacy fallback destination file |
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

Run the complete test suite:

```bash
alyac test
```

Run micro-benchmarks:

```bash
alyac run benches/bench_basic.alya
```

Run feature demonstration:

```bash
alyac run examples/demo.alya
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.