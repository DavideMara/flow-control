# Flow-Control

This repository tries implements a reliable data transfer protocol over a simulated network link using a **Stop-and-Wait** mechanism.

## Project Overview

The goal of this project is to provide reliable communication between two endpoints over an unreliable link that can drop or corrupt packets. The system simulates a network environment where packets are sent over UDP.

### Key Features
- **Reliable Data Transfer:** Ensures data is delivered correctly and in order.
- **Error Detection:** Uses a 16-bit checksum to detect corrupted packets.
- **Flow Control:** Implements a Stop-and-Wait protocol to manage data flow between sender and receiver.
- **Retransmission:** Uses timers to retransmit packets that are lost or corrupted.
- **Synthetic Traffic Generation:** Supports both manual console input and synthetic traffic for testing.
- **Error Simulation:** Can simulate packet corruption to test the protocol's robustness.

## Protocol Description: Stop-and-Wait

The implemented protocol is a classic Stop-and-Wait ARQ (Automatic Repeat Request) using the **Alternating Bit** algorithm:

1.  **Sender Side:**
    -   Wait for data from the application layer.
    -   Send a DATA packet with a sequence number (0 or 1).
    -   Start a retransmission timer.
    -   Wait for an ACK with the matching sequence number.
    -   If an ACK is received:
        -   Stop the timer.
        -   Toggle the sequence number.
        -   Resume sending data.
    -   If the timer expires:
        -   Retransmit the last DATA packet.
        -   Restart the timer.

2.  **Receiver Side:**
    -   Wait for a DATA packet.
    -   Validate the checksum.
    -   If the packet is valid and has the expected sequence number:
        -   Deliver the data to the application layer.
        -   Send an ACK with the received sequence number.
        -   Toggle the expected sequence number.
    -   If the packet is a duplicate (previous sequence number):
        -   Resend the ACK for that sequence number.
    -   If the packet is corrupted:
        -   Discard it and wait for retransmission.

## Implementation Details

The core logic is implemented in `eventCallbacks.c`, which interacts with a provided library (`library.c`, `library.h`, `global.h`).

### State Variables
- `sender_seq_next`: The sequence number for the next packet to be sent (0 or 1).
- `receiver_seq_expected`: The sequence number the receiver is currently waiting for.
- `waiting_ack`: A flag indicating the sender is awaiting an acknowledgment.
- `lst_packet_buffer`: Stores the last sent DATA packet for potential retransmission.
- `timeout_ns`: The duration of the retransmission timer in nanoseconds.

### API Functions Used
- `READ_DATA_FROM_APP_LAYER`: Retrieves data to be sent from the application.
- `SEND_DATA_PACKET`: Transmits a DATA packet.
- `SEND_ACK_PACKET`: Transmits an ACK packet.
- `ACCEPT_DATA`: Delivers received data to the application layer.
- `SET_TIMER` / `CLEAR_TIMER`: Manages the retransmission timer.
- `VALIDATE_CHECKSUM`: Verifies the integrity of received packets.
- `PAUSE_TRANSMISSION` / `RESUME_TRANSMISSION`: Controls the flow of data from the application layer.

## Project Structure

- `eventCallbacks.c`: Contains the student's implementation of the protocol callbacks.
- `global.h`: Defines packet structures and API function prototypes.
- `library.c` / `library.h`: The simulation environment and network utilities (UDP handling, checksums, etc.).
- `Makefile`: Build instructions for the project.
- `flowcontrol`: The compiled executable.

## Compilation and Usage

### Prerequisites
- GCC compiler

### Building the Project
Run the following command in the project directory:
```bash
make
```

### Running the Simulation
To start the program:
```bash
./flowcontrol [options]
```

### Common Options
- `-w <size>`: Set the window size (default is 1 for Stop-and-Wait).
- `-t <timeout>`: Set the retransmission timeout in nanoseconds.
- `-e <prob>`: Set the probability of packet corruption (e.g., `0.1` for 10%).
- `-s`: Enable synthetic traffic generation.
- `-d <level>`: Enable debugging output (levels 1-3).

## Author
Developed by Davide Maracaglia for the **G661 - Introducción a las Redes de Computadores** course at **UniCan** (Universidad de Cantabria), Santander.
