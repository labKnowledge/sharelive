# DropShare - Peer-to-Peer File Transfer

A secure, decentralized file sharing application that enables direct device-to-device file transfers without requiring a central server. Built with modern web technologies to create a true peer-to-peer mesh network.

**Author**: [Remy Gakwaya](https://github.com/labKnowledge)

I built this tool to help me share files across my devices. Now I see that it can help even my friends to exchange files too. It's open sourced and hosted on GitHub for everyone to test alone or with friends.

## 🔗 Links

- **Live Demo**: [https://labknowledge.github.io/sharelive/](https://labknowledge.github.io/sharelive/)
- **GitHub Repository**: [https://github.com/labKnowledge/sharelive](https://github.com/labKnowledge/sharelive)

## 🚀 Features

- **True P2P Communication**: Direct device-to-device connections using WebRTC
- **Many-to-Many File Sharing**: Multiple devices can connect and share files simultaneously
- **Mesh Network Architecture**: Files are automatically broadcasted to all connected devices
- **Device Identification**: Unique device fingerprints for tracking file sources
- **No Server Required**: All communication happens directly between devices
- **Native Sharing**: Uses Web Share API for seamless link sharing on mobile devices
- **Persistent File List**: Files remain visible until the sending device disconnects

## 🏗️ Architecture Overview

### Core Technologies

#### 1. **PeerJS & WebRTC**
- **PeerJS** (`peerjs@1.5.2`): A wrapper library that simplifies WebRTC peer connections
- **WebRTC (Web Real-Time Communication)**: Browser API for direct peer-to-peer data transfer
- **How it works**:
  - One device acts as the "host" and generates a unique peer ID
  - Other devices connect using the host's peer ID via a shareable link
  - PeerJS handles the complex WebRTC signaling through its free signaling server
  - Once connected, data flows directly between devices (peer-to-peer)

#### 2. **Mesh Network Broadcasting**
The application implements a mesh network topology where:
- Each device maintains connections to all other devices
- When a file is uploaded, it's sent to all directly connected peers
- Each receiving device automatically forwards the file to all its other connections
- This creates a broadcast mechanism ensuring all devices receive all files

**Key Implementation**:
```javascript
function forwardDataToOthers(data, senderConnId, fileId) {
    // Forward data to all other connections except the sender
    connections.forEach((connInfo, connId) => {
        if (connId !== senderConnId && connInfo.connection.open) {
            connInfo.connection.send(data);
        }
    });
}
```

#### 3. **Chunked File Transfer**
Files are transferred in 16KB chunks to:
- Handle large files efficiently
- Provide real-time progress updates
- Prevent memory issues with large files
- Maintain responsive UI during transfers

**Configuration**:
```javascript
const CHUNK_SIZE = 16 * 1024; // 16KB chunks
```

#### 4. **Device Fingerprinting**
Unique device identification using multiple browser characteristics:

**Components**:
- **Canvas Fingerprinting**: Renders text to canvas and extracts rendering characteristics
- **Screen Information**: Resolution and color depth
- **Timezone**: User's timezone setting
- **Language**: Browser language preference
- **User Agent**: Browser and OS information

**Implementation**:
```javascript
function generateDeviceFingerprint() {
    // Combines multiple device characteristics
    // Stores in localStorage for persistence
    // Returns a unique 16-character identifier
}
```

**Purpose**:
- Identify the source device for each file
- Display device information in the file list
- Maintain device identity across sessions (via localStorage)

#### 5. **Web Share API**
Native sharing functionality for modern browsers and mobile devices:

**Features**:
- Opens native share dialog on mobile devices
- Allows sharing via SMS, email, social apps, etc.
- Falls back to clipboard copy on unsupported browsers
- Provides better UX than manual link copying

**Implementation**:
```javascript
await navigator.share({
    title: 'DropShare - Secure P2P File Transfer',
    text: 'Join me for a secure peer-to-peer file transfer',
    url: shareUrl
});
```

#### 6. **File Processing Pipeline**

**Upload Flow**:
1. User selects file → FileReader reads file in chunks
2. Metadata sent first (filename, size, type, source device)
3. Binary chunks sent sequentially to all connected peers
4. Progress tracked and displayed in real-time

**Download Flow**:
1. Receive metadata → Create file entry in UI
2. Receive binary chunks → Accumulate in buffer
3. Track progress → Update progress bar
4. Complete transfer → Create Blob and enable download

**Duplicate Prevention**:
- Each file gets a unique ID (`fileId`)
- `processedFileIds` Set tracks received files
- Prevents processing the same file multiple times

## 📡 Network Topology

```
Device A (Host)
    ├─── Device B (Connected via link)
    │       └─── Receives files from A
    │       └─── Forwards files to C
    │
    └─── Device C (Connected via same link)
            └─── Receives files from A
            └─── Receives files from B (forwarded)
            └─── Forwards files to B
```

**Connection Model**:
- **Host-Initiated**: First device creates a peer ID and shares link
- **Peer Connections**: Other devices connect using the shared link
- **Bidirectional**: All connections are bidirectional (full-duplex)
- **Automatic Forwarding**: Files are automatically broadcasted to all peers

## 🔐 Security & Privacy

### What's Secure:
- **Direct P2P**: Files never pass through a central server
- **WebRTC Encryption**: All data is encrypted in transit (DTLS/SRTP)
- **No Storage**: Files are only held in memory during transfer
- **Local Processing**: All file handling happens client-side

### Privacy Considerations:
- **Device Fingerprinting**: Used only for identification, not tracking
- **No Analytics**: No external tracking or analytics
- **Ephemeral**: Files are removed when devices disconnect
- **LocalStorage**: Only stores device fingerprint (not files)

## 🛠️ Technical Stack

### Frontend Technologies:
- **HTML5**: Semantic markup and structure
- **CSS3**: Modern styling with CSS variables and flexbox/grid
- **Vanilla JavaScript**: No frameworks, pure ES6+ JavaScript
- **FileReader API**: Reading files from user's device
- **Blob API**: Creating downloadable file objects
- **URL.createObjectURL**: Generating download links

### Browser APIs Used:
- **WebRTC**: Peer-to-peer connections (via PeerJS)
- **Web Share API**: Native sharing functionality
- **Clipboard API**: Fallback for link copying
- **LocalStorage API**: Device fingerprint persistence
- **Canvas API**: Device fingerprinting
- **File API**: File handling and reading

### External Dependencies:
- **PeerJS** (CDN): WebRTC abstraction library
- **Google Fonts**: Inter font family

## 📊 Data Flow

### File Transfer Protocol:

1. **Connection Establishment**:
   ```
   Device A → PeerJS Server → Device B
   (Signaling for WebRTC handshake)
   ```

2. **Device Identification**:
   ```
   Device A → Device B: {type: 'device-info', fingerprint: '...'}
   Device B → Device A: {type: 'device-info', fingerprint: '...'}
   ```

3. **File Transfer**:
   ```
   Device A → Device B: {type: 'metadata', fileId: '...', name: '...', size: '...'}
   Device B → Device C: {type: 'metadata', ...} (forwarded)
   Device A → Device B: [Binary Chunk 1]
   Device B → Device C: [Binary Chunk 1] (forwarded)
   Device A → Device B: [Binary Chunk 2]
   Device B → Device C: [Binary Chunk 2] (forwarded)
   ...
   ```

4. **Completion**:
   ```
   Device B: Receives all chunks → Creates Blob → Enables download
   Device C: Receives all chunks → Creates Blob → Enables download
   ```

## 🎯 Key Design Decisions

### Why PeerJS?
- Simplifies complex WebRTC signaling
- Handles NAT traversal automatically
- Provides reliable data channels
- Free signaling server included

### Why Chunked Transfer?
- Prevents memory overflow with large files
- Enables real-time progress tracking
- Maintains UI responsiveness
- Allows for pause/resume (future enhancement)

### Why Device Fingerprinting?
- Provides user-friendly device identification
- Better than showing raw peer IDs
- Persistent across sessions
- Privacy-preserving (local only)

### Why Mesh Broadcasting?
- Ensures all devices see all files
- Works regardless of connection topology
- No single point of failure
- True many-to-many communication

## 🚧 Limitations & Considerations

### Current Limitations:
- **Signaling Server Dependency**: Uses PeerJS's free signaling server (can be self-hosted)
- **Browser Support**: Requires modern browsers with WebRTC support
- **NAT/Firewall**: May have issues behind strict firewalls (WebRTC handles most cases)
- **File Size**: Limited by browser memory (practical limit ~100-500MB depending on device)
- **No Encryption at Rest**: Files are only encrypted in transit

### Browser Compatibility:
- ✅ Chrome/Edge (Chromium): Full support
- ✅ Firefox: Full support
- ✅ Safari: Full support (iOS 11+)
- ⚠️ Older browsers: Limited or no support

## 🔮 Future Enhancements

Potential improvements:
- **End-to-End Encryption**: Encrypt files before transfer
- **Resume Transfer**: Pause and resume file transfers
- **File Compression**: Compress files before transfer
- **Self-Hosted Signaling**: Use custom signaling server
- **Connection Quality Indicators**: Show connection strength
- **File Preview**: Preview images/videos before download
- **Transfer History**: Log of transferred files

## 📝 Usage

1. **Host Device**:
   - Open the application
   - Click "Share Link" (uses Web Share API or copies to clipboard)
   - Share the link with other devices

2. **Peer Devices**:
   - Open the shared link
   - Automatically connects to the host
   - Can now send and receive files

3. **File Transfer**:
   - Any device can upload files
   - All connected devices receive the file
   - Files show the source device fingerprint
   - Download when ready

## 🧪 Testing

You can test the application using the [live demo](https://labknowledge.github.io/sharelive/) or by running it locally.

**Using the Live Demo**:
1. Open [https://labknowledge.github.io/sharelive/](https://labknowledge.github.io/sharelive/) in multiple browser tabs/windows or on different devices
2. One tab/device acts as host (no `?id=` parameter)
3. Click "Share Link" and share it with other tabs/devices
4. Open the shared link in other tabs/devices (adds `?id=` parameter)
5. Upload files from any device
6. Verify all devices receive the files

**Local Testing**:
1. Clone the repository: `git clone https://github.com/labKnowledge/sharelive.git`
2. Open `index.html` in a web browser
3. Follow the same steps as above

## 📄 License

This project is open source and available for use and modification.

## 🙏 Acknowledgments

- **PeerJS**: For simplifying WebRTC peer connections
- **WebRTC**: For enabling direct browser-to-browser communication
- **Web Share API**: For native sharing capabilities

---

**Built with modern web technologies for a truly decentralized file sharing experience.**

**Created by Remy Gakwaya** - A tool to share files across devices, now open for everyone to use.

