# GhostY — Private Peer-to-Peer Communication

**GhostY** is an Android communication application built around WebRTC, enabling two devices to establish a peer-to-peer connection for messaging and real-time communication. A lightweight WebSocket signaling server helps the devices establish their connection, while WebRTC handles the communication channel.

The project explores decentralized communication, QR-based device pairing, real-time connectivity, and privacy-conscious application design.

> **Project status:** Development project. Features, network compatibility, and security properties should be verified against the current implementation before production use.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [How It Works](#how-it-works)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Building the APK](#building-the-apk)
- [Network Requirements](#network-requirements)
- [Security and Privacy](#security-and-privacy)
- [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [Related Repository](#related-repository)
- [Contributing](#contributing)
- [License](#license)

## Overview

Traditional messaging applications generally rely on centralized infrastructure to coordinate communication and deliver messages. GhostY experiments with a different approach: establish a connection between two Android devices using WebRTC, with a signaling service used during connection setup.

The signaling server exchanges the information required to negotiate a WebRTC session. After the connection is established, supported communication can travel directly between the devices or through a TURN relay when direct connectivity is unavailable.

GhostY is designed as a learning and development project for understanding:

- WebRTC peer connections and ICE negotiation.
- QR-based device pairing.
- WebSocket signaling.
- Real-time data channels.
- Android application lifecycle and connection management.
- Network traversal using STUN and TURN.

## Features

### Device pairing

- QR-code-based connection initiation.
- QR scanning to begin the pairing process.
- Explicit connection approval on the receiving device.
- Support for exchanging pairing information through an image, where implemented by the current build.

**Important:** A pairing QR code is connection information, not proof of identity by itself. Treat it as sensitive and share it only with the intended recipient.

### Real-time messaging

- WebRTC DataChannel-based messaging.
- Sending messages after a peer connection is established.
- Receiving messages through the data channel.
- Connection status handling for the messaging interface.

### Audio and video communication

- WebRTC-based media communication.
- Camera and microphone access subject to Android permissions.
- ICE negotiation for establishing the underlying peer connection.

Media functionality depends on the current implementation, device permissions, network conditions, and successful connection negotiation.

### Connection management

- WebSocket signaling for exchanging connection setup messages.
- SDP offer/answer negotiation.
- ICE candidate exchange.
- STUN/TURN support for network traversal.
- Connection-state monitoring.

### Application access and navigation

- Application lock and biometric-related access flow where supported by the device and current build.
- Navigation between the connection and chat interfaces.

## Architecture

```text
             Android Device A
          ┌─────────────────────┐
          │ GhostY Android App  │
          │ QR pairing          │
          │ WebRTC client       │
          └──────────┬──────────┘
                     │
                     │ WebSocket signaling
                     ▼
          ┌─────────────────────┐
          │ Signaling Server    │
          │ Node.js + ws        │
          │ Session negotiation │
          └──────────┬──────────┘
                     │
                     │ WebSocket signaling
                     ▼
             Android Device B
          ┌─────────────────────┐
          │ GhostY Android App  │
          │ QR pairing          │
          │ WebRTC client       │
          └─────────────────────┘

        WebRTC communication path:

        Device A ◄────────────► Device B
                  or
        Device A ◄── TURN ────► Device B
```

The signaling server coordinates connection establishment. It is not intended to act as the normal transport for messages sent over an established WebRTC DataChannel.

A TURN relay may carry WebRTC traffic when a direct connection cannot be established.

## How It Works

1. **Initiate pairing:** One device generates or displays the pairing information.
2. **Share the QR code:** The other device scans the code or imports its image if that workflow is supported by the installed build.
3. **Approve the connection:** The receiving user reviews and accepts the connection request.
4. **Establish signaling:** Both devices communicate with the signaling server over WebSocket.
5. **Exchange SDP:** The peers exchange session descriptions containing the information required to negotiate a WebRTC session.
6. **Exchange ICE candidates:** The peers discover and exchange possible network routes.
7. **Establish connectivity:** WebRTC attempts a suitable connection, potentially using a TURN relay.
8. **Communicate:** Once the required connection or DataChannel is ready, supported messages and media can flow.

The exact sequence can vary depending on when ICE candidates are gathered and exchanged.

## Technology Stack

| Technology | Purpose |
|---|---|
| Android SDK | Android application platform |
| Kotlin / Android components | Application logic and interface |
| XML layouts | Android UI |
| WebRTC | Peer connections, media, and DataChannels |
| WebSocket | Signaling transport |
| Node.js | Signaling server runtime |
| `ws` | WebSocket server library |
| STUN | Discovery of public-facing network addresses |
| TURN | Relaying traffic when direct connectivity fails |
| Render | Hosting the signaling service |

## Prerequisites

Install the following before building the project:

- Android Studio.
- A compatible JDK for the project's Gradle version.
- Android SDK and the required SDK platform/build tools.
- Git.
- An Android device or emulator for testing.
- Network access to the configured signaling service.
- Valid STUN/TURN configuration if required by your network environment.

Use the JDK and Android SDK versions specified by the project configuration rather than assuming the latest versions are compatible.

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/TheVivekYadav/GhostY.git
cd GhostY
```

### 2. Open the project

Open the cloned directory in Android Studio and allow Gradle synchronization to finish.

### 3. Configure connection services

Check the application's WebRTC and signaling configuration.

The currently used signaling service is:

`wss://ghostlink-signaling-dwaq.onrender.com`

If the application is configured to use another server, update the endpoint in the appropriate configuration location.

Configure STUN/TURN servers according to your deployment. Do not commit private TURN credentials, API keys, or other secrets to a public repository.

### 4. Run the application

Connect an Android device with USB debugging enabled, or start a compatible emulator.

Select the application configuration in Android Studio and run it.

For meaningful peer-to-peer testing, use two devices. Test them on the same network and on different networks to evaluate connectivity behavior.

## Building the APK

Build a debug APK using the Gradle wrapper:

```bash
./gradlew assembleDebug
```

On Linux, if the wrapper is not executable:

```bash
chmod +x gradlew
./gradlew assembleDebug
```

The APK is normally generated at:

`app/build/outputs/apk/debug/app-debug.apk`

Install it on a connected device using:

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

For distribution, generate a signed release APK or Android App Bundle through Android Studio. Protect the signing key and its passwords, and never commit them to Git.

## Network Requirements

WebRTC connectivity depends on the network environment.

- **Signaling:** Both devices must be able to reach the configured WebSocket server during connection establishment.
- **STUN:** Helps discover candidate network addresses.
- **TURN:** May be required for restrictive NATs, firewalls, or networks that prevent direct connectivity.
- **Permissions:** Camera and microphone access are needed for corresponding media features.
- **Background behavior:** Android may suspend network activity or restrict resources when the application is backgrounded.
- **Connection recovery:** Reopening the application does not automatically guarantee that an old peer connection is still valid.

A successful signaling connection does not necessarily mean that the WebRTC connection has succeeded.

## Security and Privacy

GhostY explores peer-to-peer communication, but privacy claims must match the actual implementation.

### What the architecture provides

- WebRTC uses encryption for its standard media and data transport.
- WebSocket Secure (`wss://`) encrypts the connection between a device and the signaling server in transit.
- An established DataChannel can carry messages without routing them through the signaling server as ordinary application messages.
- A TURN relay forwards traffic; it does not inherently provide access to application plaintext.

### What still requires verification

- **Peer identity:** A QR code alone does not prove that the other endpoint belongs to the intended person.
- **End-to-end security claims:** WebRTC transport encryption is not, by itself, proof of an independently verified identity system or a complete application-level end-to-end encryption design.
- **Message persistence:** Do not assume messages are stored locally, delivered while offline, or automatically synchronized unless implemented.
- **Metadata:** Signaling services can potentially observe connection metadata, such as connection timing, IP-related information, and session identifiers.
- **Pairing QR codes:** Protect them from unintended recipients and implement expiration or one-time use where appropriate.
- **Credentials:** Keep TURN credentials and deployment secrets out of source control.
- **Biometric access:** An application lock protects access to the local interface; it does not automatically secure network signaling or verify remote peers.

Do not use GhostY for sensitive production communication until the authentication, key management, session lifecycle, and security model have been reviewed and tested.

## Troubleshooting

### The peers do not connect

- Verify the configured signaling URL.
- Check whether the signaling service is running and reachable.
- Confirm that both devices have joined the expected session or room.
- Inspect SDP offer/answer exchange and ICE candidate logs.
- Check whether TURN configuration is required.

### ICE reaches `FAILED`

A signaling connection can work even when the peers cannot establish a media or data path.

- Verify STUN/TURN configuration.
- Confirm that TURN credentials are valid and not expired.
- Inspect the selected ICE candidate pair.
- Test using different networks.

### The DataChannel never opens

- Check whether both peers completed session negotiation.
- Inspect ICE and peer-connection state transitions.
- Confirm that the offer/answer and data-channel negotiation logic is consistent.

### Video is black or unavailable

- Verify camera permission and Android runtime permission handling.
- Confirm that the correct video track and capturer are initialized.
- Check local and remote video renderer setup.
- Inspect media negotiation and track lifecycle events.

### Messaging stops after navigating away and returning

- Inspect the Activity lifecycle and ownership of the WebRTC client.
- Check whether the connection was closed or retained during navigation.
- Ensure the UI observes the current connection state instead of assuming the previous session is still active.

## Project Structure

The main Android project includes:

```text
GhostY/
├── app/
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew
├── .gitignore
└── README.md
```

The `app/` directory contains the Android application source, resources, and module configuration.

## Related Repository

**Signaling server:** [github.com/TheVivekYadav/signallingServer](https://github.com/TheVivekYadav/signallingServer)

The server handles signaling messages required to negotiate WebRTC connections between peers.

## Contributing

Contributions and bug reports are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make and test your changes.
4. Open a pull request describing the change and its testing results.

When reporting connection issues, include relevant logs and network conditions. Remove credentials, tokens, and other sensitive information before sharing logs.

## License

No license is specified here. Add a `LICENSE` file and update this section before distributing the project publicly. Until a license is added, do not assume that others have permission to reuse or redistribute the source code.
