# Reliable File Transfer Using UDP

## Overview

This project involves implementing a reliable sender (`sender.py`) over an unreliable UDP network using techniques such as stop-and-wait reliability and cumulative-ACK-based selective repeat. The provided `receiver.py` handles receiving and acknowledges file transmissions over a lossy channel. The goal is to ensure the receiver’s version of the file matches the sender’s, even in the presence of packet or ACK loss.

## Objectives

1. Implement stop-and-wait reliability in `sender.py`.
2. Extend the sender to support pipelined reliability with cumulative acknowledgments.
3. Test the reliability of the implemented sender under various loss scenarios.

---

## Implementation Steps

### Step 1: Initial Testing
1. Test file transfer without packet or ACK loss:
   - Run `receiver.py` with `--pktloss noloss`.
   - Run `sender.py` to upload `test-input.txt`.
   - Verify file integrity using `diff test-output.txt test-input.txt`.

2. Test with default packet loss:
   - Run `receiver.py` (default settings drop every 3rd packet).
   - Run `sender.py`.
   - Check the downloaded file for missing data and observe sequence space holes in the receiver logs.

### Step 2: Stop-and-Wait Reliability
1. Modify `send_reliable()` in `sender.py`:
   - Wait for an ACK before sending the next packet.
   - Retransmit packets on timeout using `select()` for socket readiness.
   - Advance the window (`win_left_edge`) on receiving valid ACKs.

2. Test stop-and-wait implementation:
   - Simulate different packet and ACK loss scenarios using `--pktloss` and `--ackloss` flags.
   - Validate file integrity with `diff`.

3. Freeze stop-and-wait implementation by saving it as `stopandwait.py`.

### Step 3: Pipelined Reliability
1. Implement cumulative acknowledgments:
   - Transmit multiple packets within a window using `transmit_entire_window_from()`.
   - Track the highest cumulative ACK (`last_acked`) and slide the window forward.

2. Handle out-of-order data:
   - Enable out-of-order buffering in `receiver.py` using `--ooo_enabled`.
   - Retransmit only unacknowledged packets after a timeout.

3. Test pipelined reliability:
   - Use various window sizes (`--winsize` flag) and loss scenarios.
   - Compare performance with stop-and-wait reliability.

---

## Testing Scenarios

1. No packet or ACK loss.
2. Packet loss:
   - Every nth packet (`--pktloss everyn`).
   - Alternate packet loss (`--pktloss alteveryn`).
   - Random packet loss (`--pktloss iid`).
3. ACK loss:
   - Every nth ACK (`--ackloss everyn`).
   - Alternate ACK loss (`--ackloss alteveryn`).
   - Random ACK loss (`--ackloss iid`).
4. Combination of packet and ACK loss.
5. Delayed receiver start.
