# AirPost

A lightweight, high-performance file transfer utility designed for efficient peer-to-peer transfers over local networks or the internet.

## Features

- **Server mode**: Launch a file transmitting server with configurable control port, storage directory, and bandwidth limits.
- **Upload mode**: Upload files to any running AirPost server with multi-threaded support.
- **Authentication system**: Optional identity-based access control using device UIDs.
- **Flexible configuration**: Customize ports, thread counts, speed limits, and more via command-line options.
- **Cross-platform**: Written in Rust, single binary deployment.

## Installation

AirPost is a standalone executable. You don't need to install extra dependencies.

## Usage

```
airpost server [..options]
airpost upload <file_path> <server_addr:control_port> [..options]
airpost auth <command>
```

### Global Options

| Option                 | Description                          |
|------------------------|--------------------------------------|
| `help`, `--help`, `-h` | Show this help message               |
| `-V`                   | Get prettier output (verbose)        |
| `-v`                   | Show version                         |

---

## Commands

### `server` – Start an AirPost server

Launch a server that listens for incoming file uploads.

#### Server Options

| Option           | Default             | Description                                            |
|------------------|---------------------|--------------------------------------------------------|
| `-p <port>`      | Auto-allocated      | Control port (range: 1024-65535)                       |
| `-s <directory>` | `./airpost_storage` | Directory where received files are stored              |
| `-l <bytes/s>`   | No limit            | Maximum transfer speed in bytes per second             |
| `-a`             | Off                 | Enable identity authentication (requires `auth` setup) |

#### Examples

```bash
# Start with auto-selected port, no speed limit, no authentication
airpost server

# Start on port 9000 with authentication enabled
airpost server -p 9000 -a

# Start with custom storage dir and speed limit of 1 MiB/s
airpost server -p 9000 -s /tmp/airpost_storage -l 1048576
```

---

### `upload` – Send a file to a server

Transfer a file to a remote AirPost server.

#### Upload Options

| Option       | Default | Description                         |
|--------------|---------|-------------------------------------|
| `-t <count>` | 8       | Number of concurrent upload threads |

#### Arguments

| Argument                     | Description                                   |
|------------------------------|-----------------------------------------------|
| `<file_path>`                | Path to the file to upload                    |
| `<server_addr:control_port>` | Address and control port of the target server |

#### Examples

```bash
# Upload with default 8 threads
airpost upload myfile.zip 127.0.0.1:9000

# Upload with 4 threads
airpost upload myfile.zip 127.0.0.1:9000 -t 4
```

---

### `auth` – Manage authentication

Identity-based access control (`-a` parameter) requires each client to be registered before uploading.

#### Auth Subcommands

| Command                            | Description                                                                                                                                                                                                                                                                                 |
|------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `uid`                              | Display this device's unique identifier (UID)                                                                                                                                                                                                                                               |
| `register <uid> [validity_period]` | Register a device UID on the server’s whitelist. Generates a **client authentication binary** (`auth.bin`) that must be placed alongside the `airpost` executable (or at the path specified by `AIRPOST_AUTH_BIN` environment variable). Also updates the server’s authentication database. |
| `revoke <uid> [validity_period]`   | Revoke a previously registered device’s token. Updates the server’s authentication database.                                                                                                                                                                                                |

#### Auth Options

| Parameter         | Default             | Description                                                |
|-------------------|---------------------|------------------------------------------------------------|
| `uid`             | –                   | Device UID (hex string, e.g., `1a2b3c4d5e6f`)              |
| `validity_period` | `31536000` (1 year) | Token validity in seconds; use `-1` for permanent validity |

#### Environment Variable

| Variable           | Description                                                                    |
|--------------------|--------------------------------------------------------------------------------|
| `AIRPOST_AUTH_BIN` | Override the default path (`./auth.bin`) for the client authentication binary. |

#### Examples

```bash
# Get current device UID
airpost auth uid

# Register device 1a2b3c4d5e6f with token valid for 1 day (86400 seconds)
airpost auth register 1a2b3c4d5e6f 86400
# Output example:
# Client auth binary successfully saved to ./auth.bin
# Remember to copy it to the same directory of airpost or the file pointed to by AIRPOST_AUTH_BIN
# Server auth binary successfully updated

# Revoke device 1a2b3c4d5e6f (default permanent revocation)
airpost auth revoke 1a2b3c4d5e6f
```

---

### Authentication Workflow

1. On the server, start with `-a` flag to enable authentication.
2. Each client runs `airpost auth uid` to obtain its own UID.
3. The server administrator registers the client’s UID using `airpost auth register <uid> [duration]`. This command:
    - Creates a file `auth.bin` containing the client’s authentication token.
    - Updates the server’s internal whitelist.
4. The client must **copy** the generated `auth.bin` file to the same directory as the `airpost` executable, or place it
   as the file defined by the `AIRPOST_AUTH_BIN`(e.g. /root/auth_server.bin) environment variable.
5. The client can now upload files to that authenticated server.
6. To remove access, run `airpost auth revoke <uid>` on the server side.

> **Important:**
> - The `auth.bin` file is **unique per client registration**. If a client loses this file or replaces its motherboard,
    it must be re-registered.
> - The `auth` commands (`register`, `revoke`) must be executed on the **server machine**. The `uid` command is run on
    the **client machine**.
> - To allow identity authentication, ensure the server is running with authentication enabled (`-a`).

## License

See LICENSE for details.
