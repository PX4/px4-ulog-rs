# px4-ulog

[![crates.io](https://img.shields.io/crates/v/px4-ulog.svg)](https://crates.io/crates/px4-ulog)

A Rust parser for the [PX4 ULog](https://docs.px4.io/main/en/dev_log/ulog_file_format.html)
file format, with a small memory footprint. Reading is streaming where possible,
so large logs can be processed without loading them entirely into memory.

## Install

```shell
cargo add px4-ulog
```

## Usage

The crate offers two parsers.

### Full parser

Reads the whole log and returns every dataset, grouped by topic. Simplest to
use when you can hold the parsed result in memory.

```rust
use px4_ulog::full_parser::read_file;

let parsed = read_file("flight.ulg")?;
for (topic, instances) in &parsed.messages {
    println!("{topic}: {} instance(s)", instances.len());
}
# Ok::<(), std::io::Error>(())
```

### Streaming parser

Drives a callback for each message as the file is read, so memory use stays
flat regardless of log size. Return `SimpleCallbackResult::Stop` to halt early.

```rust
use px4_ulog::stream_parser::{read_file_with_simple_callback, Message, SimpleCallbackResult};

read_file_with_simple_callback("flight.ulg", &mut |msg| {
    if let Message::Data(data) = msg {
        println!("{}", data.flattened_format.message_name());
    }
    SimpleCallbackResult::KeepReading
})?;
# Ok::<(), std::io::Error>(())
```

For finer control, `stream_parser::LogParser` lets you register a callback per
message type and feed bytes in via `consume_bytes`. See the [`examples`](examples/)
directory for more.

## Command line

The crate also ships a `px4-ulog` binary for quick inspection:

```shell
# List the topics in a log
px4-ulog flight.ulg

# Dump a topic, optionally filtered to specific fields
px4-ulog flight.ulg vehicle_gps_position lat lon alt
```

## Features

All ULog format features are supported. Writing ULog files is not supported.

Recovering from file corruption by scanning for the synchronization message is
planned.

## Design goals

  * Stream the file and read it in a single pass
  * Keep memory use minimal
  * No `unsafe`
  * Don't panic on malformed input

## Development

Install the pre-commit hooks before contributing:

```shell
pre-commit install
```

## License

MIT
