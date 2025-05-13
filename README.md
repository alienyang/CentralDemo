# QChatSampleActivity 架構解析

## 架構圖

```mermaid
graph TD
    A[QChatSampleActivity] --> B[QChatActionManager]
    B --> C[BleClientService]
    C --> D[AiManager]
    D --> E[AR 眼鏡]
    
    F[EventBus] --> A
    D --> F
    
    A --> G[UI 元件]
    G --> H[開始錄音按鈕]
    G --> I[停止錄音按鈕]
    G --> J[發送命令按鈕]
    G --> K[關閉按鈕]
    G --> L[文本顯示區域]
```

## 流程圖

### 開始錄音流程

```mermaid
sequenceDiagram
    participant User as 用戶
    participant QCA as QChatSampleActivity
    participant QAM as QChatActionManager
    participant BCS as BleClientService
    participant AIM as AiManager
    participant ARG as AR眼鏡
    
    User->>QCA: 點擊開始錄音按鈕
    QCA->>QAM: startRecordingAsync(QChatType)
    QAM-->>QAM: 發出 startRecordingTriggered 事件
    QAM->>BCS: 通知開始錄音
    BCS->>AIM: startRecording(QChatType)
    AIM->>AIM: 開始錄音並處理
    AIM->>ARG: 發送處理結果
    AIM->>EventBus: 發送 OnSentenceDetected 事件
    EventBus->>QCA: 接收 OnSentenceDetected 事件
    QCA->>QCA: 更新 UI 顯示文本
```

### 停止錄音流程

```mermaid
sequenceDiagram
    participant User as 用戶
    participant QCA as QChatSampleActivity
    participant QAM as QChatActionManager
    participant BCS as BleClientService
    participant AIM as AiManager
    
    User->>QCA: 點擊停止錄音按鈕
    QCA->>QAM: stopRecordingAsync()
    QAM-->>QAM: 發出 stopRecordingTriggered 事件
    QAM->>BCS: 通知停止錄音
    BCS->>AIM: stopRecording()
    AIM->>AIM: 停止錄音並完成處理
```

### 發送命令流程

```mermaid
sequenceDiagram
    participant User as 用戶
    participant QCA as QChatSampleActivity
    participant QAM as QChatActionManager
    participant BCS as BleClientService
    participant ARG as AR眼鏡
    
    User->>QCA: 點擊發送命令按鈕
    QCA->>QAM: sendCommandAsync(byteArray)
    QAM-->>QAM: 發出 commandToBeSent 事件
    QAM->>BCS: 通知發送命令
    BCS->>ARG: 通過藍牙發送命令
```

### 關閉活動流程

```mermaid
sequenceDiagram
    participant User as 用戶
    participant QCA as QChatSampleActivity
    participant QAM as QChatActionManager
    participant BCS as BleClientService
    participant ARG as AR眼鏡
    
    User->>QCA: 點擊關閉按鈕
    QCA->>QAM: stopRecordingAsync()
    QAM->>BCS: 通知停止錄音
    QCA->>QAM: sendCommandAsync(closeApp)
    QAM->>BCS: 通知發送關閉命令
    BCS->>ARG: 發送關閉應用命令
    QCA->>QCA: finish() 關閉活動
```

## 模組圖

```mermaid
classDiagram
    class QChatSampleActivity {
        -ActivityQchatSampeBinding binding
        -QChatActionManager qChatActions
        -QChatType type
        -byte[] closeApp
        +onCreate(Bundle)
        +onDestroy()
        +onStartRecordingClicked(View)
        +onStopRecordingClicked(View)
        +onSendCommandClicked(View)
        +onCloseClicked(View)
        -startRecording()
        -stopRecording()
        -sendCommand(byte[])
        +onSentenceDetected(OnSentenceDetected)
    }
    
    class QChatActionManager {
        -MutableSharedFlow<QChatType> _startRecordingTriggered
        -MutableSharedFlow<Any> _stopRecordingTriggered
        -MutableSharedFlow<ByteArray> _commandToBeSent
        +SharedFlow<QChatType> startRecordingTriggered
        +SharedFlow<Any> stopRecordingTriggered
        +SharedFlow<ByteArray> commandToBeSent
        -startRecording(QChatType)
        -stopRecording()
        -sendCommand(ByteArray)
        +startRecordingAsync(QChatType): CompletableFuture
        +stopRecordingAsync(): CompletableFuture
        +sendCommandAsync(ByteArray): CompletableFuture
    }
    
    class BleClientService {
        -BleClient bleClient
        -AiManager aiManager
        -QChatActionManager qChatAction
        +init()
        +startVoiceRecorder()
        +stopVoiceRecorder()
        +sendShortMessage(ByteArray)
    }
    
    class AiManager {
        -IAiManager listener
        +startRecording(QChatType)
        +stopRecording()
        +analyzeImage(ByteArray)
    }
    
    class OnSentenceDetected {
        -String _sentence
        +String getSentence()
    }
    
    class QChatType {
        <<enumeration>>
        Translation
        Menu
        VoiceAssistant
        AICamera
        None
    }
    
    QChatSampleActivity --> QChatActionManager: 使用
    QChatSampleActivity --> QChatType: 使用
    QChatSampleActivity ..> OnSentenceDetected: 接收
    QChatActionManager --> BleClientService: 通知
    BleClientService --> AiManager: 使用
    AiManager ..> OnSentenceDetected: 發送
```

## QChatSampleActivity 功能模組

```mermaid
graph TB
    subgraph QChatSampleActivity
        A[初始化] --> B[UI 交互]
        B --> C[錄音控制]
        B --> D[命令發送]
        B --> E[活動關閉]
        F[事件處理] --> G[更新 UI]
    end
    
    subgraph QChatActionManager
        H[事件流管理] --> I[異步橋接]
        I --> J[Java 兼容性]
    end
    
    subgraph BleClientService
        K[藍牙通信] --> L[語音處理]
        L --> M[AI 分析]
    end
    
    subgraph AR眼鏡
        N[接收命令] --> O[執行操作]
        O --> P[返回結果]
    end
    
    C --> H
    D --> H
    E --> H
    H --> K
    P --> F
```

## QChatType 與功能對應

```mermaid
graph LR
    subgraph QChatType
        A[Translation] --> B[翻譯功能]
        C[Menu] --> D[菜單控制]
        E[VoiceAssistant] --> F[語音助手]
        G[AICamera] --> H[AI 相機功能]
        I[None] --> J[無特定功能]
    end
    
    B --> K[語音翻譯處理]
    D --> L[菜單導航控制]
    F --> M[語音指令處理]
    H --> N[圖像分析處理]
```

## 數據流向圖

```mermaid
graph TD
    A[用戶語音輸入] --> B[QChatSampleActivity]
    B --> C[QChatActionManager]
    C --> D[BleClientService]
    D --> E[AiManager]
    E --> F[語音處理]
    F --> G[文本轉換]
    G --> H[命令識別]
    H --> I[EventBus]
    I --> J[OnSentenceDetected]
    J --> K[UI 更新]
    H --> L[命令發送]
    L --> M[AR眼鏡]
    M --> N[執行操作]
    N --> O[返回結果]
    O --> P[藍牙接收]
    P --> Q[事件通知]
    Q --> R[UI 反饋]
```

## 組件交互關係

```mermaid
graph TD
    subgraph 用戶界面層
        A[QChatSampleActivity]
        B[activity_qchat_sampe.xml]
    end
    
    subgraph 事件管理層
        C[QChatActionManager]
        D[EventBus]
    end
    
    subgraph 服務層
        E[BleClientService]
    end
    
    subgraph 處理層
        F[AiManager]
    end
    
    subgraph 通信層
        G[BleClient]
    end
    
    subgraph 設備層
        H[AR眼鏡]
    end
    
    A <--> B
    A --> C
    C --> E
    E --> F
    E --> G
    G --> H
    F --> D
    D --> A
```

以上圖表全面展示了 QChatSampleActivity 的架構、流程和模組關係，清晰地說明了該活動如何與其他組件交互以實現與 AR 眼鏡的語音通信功能。




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
