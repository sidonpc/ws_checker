# ws_check.py
It connects to a target WebSocket endpoint, varies one control at a time, and records exactly what the server does so you can judge whether an endpoint has real security impact.

## Purpose
This tool exists to answer the one question that sets severity.

*Does the server distinguish an authenticated client from a merely connected one, and what if anything is reachable before you prove who you are?*

It does that by isolating each access control (origin, subprotocol, credentials) and observing the handshake result, the WebSocket close code and reason, response timing, and any frames the server returns.

It is a diagnostic, not an exploit. The application-layer probing uses benign message shapes and a message you supply yourself (ideally a real client's captured first frame). It does not carry attack payloads, does not brute force, and does not fuzz aggressively.

## What it tests
1. baseline    Normal connection: origin + subprotocol + any supplied auth
2. no origin   Whether the Origin header is required at all
3. Wrong origin    Cross-Site WebSocket Hijacking (CSWSH exposure) Answers if a cross-site origin still connects.
4. No subprotocol    Whether the 'Sec-WebSocket-Protocol value is a real gate or cosmetic
5. Stripped Creds    Whether removing the cookie/auth header changes the outcome
6. idle socket     Whether the server waits for a client auth message or closes immediately + how fast
7. pre-auth probes     Pre-authentication parser surface. Sends empty, text, JSON-shaped, and binary frames and compares the reaction
8. replay valid message     Replays a captured legitimate first app message to test whether ther server acts on it without authentication. Main broken access check

## Requirements
```pip install "websockets>=12"```


## Usage
Basic Usage

```python3 ws_severity_check.py \
    --url wss://target.com/api/ws \
    --subprotocol ISYSUB \
    --origin https://target.com
```
Usage with valid first frame

```python3 ws_severity_check.py \
    --url wss://target.com/api/ws \
    --subprotocol ISYSUB \
    --origin https://target.com \
    --cookie "SESSION=...; other=..." \
    --header "Authorization: Bearer ..." \
    --valid-msg-file first_frame.bin
```

## Flags
--url   Target Websocket URL 'wss://' or 'ws://' required
--subprotocol   Value sent in 'Sec-WebSocket-Protocol (exa. ISYSUB)
--origin    ORIGIN header to send
--cookie    Cookie header string for cookie-based auth
--header    Extra header "Name: value", repeatable
--valid-msg-file    Path to captured legitimate first app message to replay (.bin files only. Edit the send type in-script if the protocol is text-based)

## Authorization
Use this only against systems you are explicitly authorized to test. It opens connections to and sends frames at the target endpoint; run it solely within the scope of a sanctioned engagement.