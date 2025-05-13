# CentralDemo Project Architecture

Based on my analysis of the project, I've created an architecture diagram and explanation for the CentralDemo project, which uses Bluetooth to communicate with Android AR glasses, with CentralLib as a submodule.

## Project Overview

```mermaid
graph TD
    A[CentralDemo App] --> B[CentralLib]
    A --> C[App Features]
    B --> D[Bluetooth Communication]
    B --> E[WiFi Direct Communication]
    D --> F[AR Glasses]
    E --> F
    C --> G[UI Components]
    C --> H[Services]
    H --> D
```

## System Architecture

The CentralDemo project is an Android application designed to communicate with AR glasses using both Bluetooth and WiFi Direct technologies. The architecture follows a modular approach with a clear separation between the main application and the communication library.

### Core Components

```mermaid
graph TD
    subgraph CentralDemo
        A[Main App] --> B[BleClientService]
        A --> C[Activities]
        C --> D[MainActivity]
        C --> E[SelectionActivity]
        C --> F[QChatSampleActivity]
        C --> G[ViewportShowcaseActivity]
        C --> H[DeepLinkActivity]
        B --> I[Operator Modules]
        I --> J[AI Manager]
        I --> K[Command Handler]
        I --> L[Speech Processing]
        I --> M[Action Manager]
    end
    
    subgraph CentralLib
        N[BleClient] --> O[Bluetooth Connectors]
        N --> P[WiFi Direct Manager]
        O --> Q[BLE Connector]
        O --> R[Socket Connector]
    end
    
    B --> N
```

## Communication Flow

```mermaid
sequenceDiagram
    participant App as CentralDemo App
    participant Service as BleClientService
    participant BleClient as CentralLib BleClient
    participant Glasses as AR Glasses
    
    App->>Service: Initialize service
    Service->>BleClient: Set up communication
    BleClient->>Glasses: Scan/Connect
    Glasses->>BleClient: Connection established
    BleClient->>Service: onDeviceConnected
    Service->>BleClient: startWiFiDirect
    BleClient->>Glasses: WiFi Direct connection
    Glasses->>BleClient: WiFi status update
    BleClient->>Service: onWiFiStatusChanged
    App->>Service: Send command
    Service->>BleClient: sendShortMessage/sendMessage
    BleClient->>Glasses: Command data
    Glasses->>BleClient: Response/Notification
    BleClient->>Service: onNotifyDataReceived/onLongNotifyReceived
    Service->>App: Event notification
```

## Key Components

### 1. Main Application (app module)

The main application handles user interactions, UI components, and orchestrates the communication with AR glasses through the CentralLib module.

#### Key Classes:
- **BleClientService**: A foreground service that manages the Bluetooth connection lifecycle and handles communication with AR glasses.
- **Activities**: Various activities for different functionalities (Main, Selection, QChat, Viewport, DeepLink).
- **Operator Package**: Contains components for handling specific operations:
  - **AI Manager**: Processes AI-related commands and responses
  - **Command Handler**: Manages different types of commands (defined in CommandType enum)
  - **Action Manager**: Handles QChat actions and triggers

### 2. CentralLib Module

This is a submodule that encapsulates all the communication logic with AR glasses, abstracting the complexities of Bluetooth and WiFi Direct communications.

#### Key Classes:
- **BleClient**: The main class that handles all communication with AR glasses, supporting both Bluetooth and WiFi Direct.
- **Bluetooth Connectors**: 
  - **BleConnector**: Handles BLE (Bluetooth Low Energy) communication
  - **SocketConnector**: Manages Bluetooth Socket communication
- **WiFi Direct Manager**: Manages WiFi Direct connections for higher bandwidth data transfer

### 3. Communication Protocols

The system uses a dual-communication approach:
- **Bluetooth**: Used for initial connection, control commands, and smaller data transfers
- **WiFi Direct**: Used for higher bandwidth data transfers like images and videos

#### Message Types:
- **Short Messages**: Simple commands sent via Bluetooth (represented as byte arrays)
- **Long Messages**: JSON-formatted messages for complex data
- **Notifications**: Phone notifications forwarded to glasses
- **Images**: Image data transferred primarily via WiFi Direct when available

### 4. Application Types

The system supports various application types (defined in AppType enum):
- QR Scanner
- Settings
- Demo App
- Camera
- ChatGPT
- Picture Viewer
- Media Pipe
- Navigation Demo
- Video
- Translate
- SMS
- Menu Demo
- Voice Assistant
- AI Camera

### 5. Command System

Commands are sent to the glasses to trigger specific actions (defined in CommandType enum):
- Volume controls
- Translation
- QR Code scanning
- Video recording/playback
- Gesture recognition
- Navigation
- Picture viewing
- AI features (Ingredients analysis, Nutrition info, etc.)

## Data Flow

1. **Connection Establishment**:
   - App scans for available AR glasses devices
   - User selects a device to connect
   - BleClientService establishes Bluetooth connection
   - Once connected, WiFi Direct connection is attempted for higher bandwidth

2. **Command Execution**:
   - User triggers an action in the app
   - Command is sent to BleClientService
   - Service forwards command to CentralLib's BleClient
   - BleClient sends command to glasses via appropriate channel (Bluetooth/WiFi)

3. **Data Reception**:
   - Glasses send data/notifications back to the app
   - BleClient receives and processes the data
   - Callbacks notify BleClientService of received data
   - Service processes data and updates UI or triggers appropriate actions

4. **Media Handling**:
   - Images captured by glasses are sent back to the app
   - Images are processed by AI Manager if needed
   - Processed results are sent back to glasses for display

## Security and Permissions

The application requires numerous permissions for its functionality:
- Bluetooth permissions (BLUETOOTH, BLUETOOTH_ADMIN, BLUETOOTH_SCAN, BLUETOOTH_CONNECT)
- WiFi permissions (ACCESS_WIFI_STATE, CHANGE_WIFI_STATE)
- Location permissions (required for Bluetooth scanning)
- Notification access permissions
- SMS and call handling permissions
- Storage permissions for media handling

## Conclusion

The CentralDemo project demonstrates a sophisticated architecture for communicating with AR glasses using a dual-channel approach (Bluetooth and WiFi Direct). The modular design with CentralLib as a separate module provides good separation of concerns, making the codebase more maintainable and extensible.

The system supports a wide range of applications and commands, allowing for versatile interactions with AR glasses, from simple notifications to complex AI-powered features like translation, object recognition, and virtual try-on experiences.
