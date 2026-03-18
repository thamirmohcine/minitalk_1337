# minitalk

![Language](https://img.shields.io/badge/Language-C-00599C?logo=c&logoColor=white)
![Build](https://img.shields.io/badge/Build-Make-6D00CC?logo=gnu)
![IPC](https://img.shields.io/badge/IPC-Signals%20(SIGUSR1%2FSIGUSR2)-2E8B57)
![School](https://img.shields.io/badge/42-Project-000000?logo=42&logoColor=white)

A lightweight UNIX IPC project that demonstrates how to transmit text between two processes using only POSIX signals.

## Key Features

- Client/server message transfer over `SIGUSR1` and `SIGUSR2`
- Bitwise character encoding (8 signals per character)
- PID-based targeting for directed message delivery
- Mandatory version with stable signal decoding loop
- Bonus version with server acknowledgment (`SIGUSR2`) after full message receipt
- Separate build targets for mandatory and bonus executables

## Tech Stack

  - None (CLI-only interaction)
  - C (procedural architecture)
  - POSIX process signaling APIs (`kill`, `sigaction`, `signal`, `pause`)
  - UNIX process model (`pid_t`, `getpid`)

- **Embedded / Low-level Tools**
  - GNU Make (`Makefile` automation)
  - `cc` compiler with `-Wall -Wextra -Werror`
  - Linux/UNIX system headers (`signal.h`, `unistd.h`, `sys/types.h`)

## Architecture

### Core Files (Analyzed)

- `server.c`: mandatory signal receiver/decoder and stdout renderer
- `client.c`: mandatory bitwise sender for user message and terminators
- `server_bonus.c`: bonus receiver with client acknowledgment behavior
- `client_bonus.c`: bonus sender with delivery confirmation handler
- `Makefile`: build orchestration for mandatory and bonus binaries

### Communication Flow

1. Server starts and prints its PID.
2. Client receives target PID + message from CLI.
3. Each character is sent bit-by-bit using:
   - `SIGUSR1` for bit `0`
   - `SIGUSR2` for bit `1`
4. Server accumulates 8 bits, reconstructs one character, and prints it.
5. Bonus path: on message completion, server sends `SIGUSR2` back to confirm delivery.

## Getting Started

### Prerequisites

- Linux or macOS
- `make`
- `cc` (or compatible C compiler)

### Build

```bash
make
```

Builds mandatory binaries:
- `server`
- `client`

### Build Bonus

```bash
make bonus
```

Builds bonus binaries:
- `server_b`
- `client_b`

### Environment Variables

No environment variables are required.

## Usage

### Mandatory

1. Start server:

```bash
./server
```

2. In another terminal, send a message:

```bash
./client <SERVER_PID> "Hello from minitalk"
```

### Bonus

1. Start bonus server:

```bash
./server_b
```

2. Send message with bonus client:

```bash
./client_b <SERVER_PID> "Bonus protocol test"
```

The bonus client prints a confirmation message after receiving server acknowledgment.

## Build Maintenance

```bash
make clean    # remove object files
make fclean   # remove objects + binaries
make re       # full rebuild
```

## Troubleshooting

- **Invalid PID**
  - Ensure you copy the PID printed by the running server process.
  - Verify the server is still alive: `ps -p <SERVER_PID>`.

- **No output on server**
  - Confirm you started `./server` (or `./server_b`) before sending from client.
  - Rebuild and retry to avoid stale binaries: `make re`.

- **Message appears corrupted or incomplete**
  - Very fast consecutive runs can cause timing issues with signals.
  - Retry once or run with shorter test messages first.

- **Bonus acknowledgment not shown**
  - Use matching bonus binaries together: `./server_b` with `./client_b`.
  - The acknowledgment is sent when message termination is detected.

- **Permission error when signaling**
  - Client and server should run under the same user session.
  - On restricted environments, avoid cross-user PID signaling.
