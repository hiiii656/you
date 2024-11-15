<html><head><base href="." />
<title>Voice Chat Hangout</title>
<script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
<style>
:root {
  --bg: #1a1a2e;
  --text: #e6e6e6;
  --accent: #4CAF50;
}

body {
  margin: 0;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: var(--bg);
  color: var(--text);
}

.container {
  max-width: 800px;
  margin: 0 auto;
}

.user-list {
  background: rgba(255,255,255,0.1);
  border-radius: 8px;
  padding: 20px;
  margin: 20px 0;
}

.user {
  display: flex;
  align-items: center;
  margin: 10px 0;
  padding: 10px;
  border-radius: 4px;
  background: rgba(255,255,255,0.05);
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--accent);
  color: white;
  font-weight: bold;
}

.controls {
  display: flex;
  gap: 10px;
  margin: 20px 0;
  flex-wrap: wrap;
}

button {
  background: var(--accent);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  transition: opacity 0.2s;
}

button:hover {
  opacity: 0.9;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.status {
  padding: 10px;
  border-radius: 4px;
  background: rgba(255,255,255,0.05);
  margin: 10px 0;
}

.audio-indicator {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #666;
  margin-left: auto;
}

.audio-indicator.speaking {
  background: var(--accent);
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

/* New chat styles */
.chat-container {
  background: rgba(255,255,255,0.1);
  border-radius: 8px;
  padding: 20px;
  margin: 20px 0;
  height: 300px;
  display: flex;
  flex-direction: column;
}

.chat-messages {
  flex-grow: 1;
  overflow-y: auto;
  margin-bottom: 10px;
  padding: 10px;
  background: rgba(255,255,255,0.05);
  border-radius: 4px;
}

.message {
  margin: 5px 0;
  padding: 5px 10px;
  border-radius: 4px;
  word-wrap: break-word;
}

.message.mine {
  background: var(--accent);
  margin-left: 20%;
}

.message.others {
  background: rgba(255,255,255,0.1);
  margin-right: 20%;
}

.chat-input-container {
  display: flex;
  gap: 10px;
  position: relative; /* Add this */
}

.chat-input {
  flex-grow: 1;
  padding: 10px;
  border-radius: 4px;
  border: none;
  background: rgba(255,255,255,0.1);
  color: var(--text);
}

.chat-input:focus {
  outline: none;
  background: rgba(255,255,255,0.15);
}

.send-btn {
  padding: 10px 20px;
}

/* Add new enable voice chat button styles */
#enableVoiceBtn {
  background: #e67e22;  /* Different color to distinguish it */
  margin-bottom: 20px;
}

#enableVoiceBtn.enabled {
  background: var(--accent);
}

/* Add new room list styles */
.room-list {
  background: rgba(255,255,255,0.1);
  border-radius: 8px;
  padding: 20px;
  margin: 20px 0;
  display: none; /* Hidden by default, shown when leaving a room */
}

.room {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 10px 0;
  padding: 15px;
  border-radius: 4px;
  background: rgba(255,255,255,0.05);
  transition: transform 0.2s;
}

.room:hover {
  transform: translateX(5px);
  background: rgba(255,255,255,0.1);
}

.room-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.room-name {
  font-weight: bold;
  color: var(--accent);
}

.room-users {
  font-size: 0.9em;
  color: rgba(255,255,255,0.7);
}

.join-btn {
  background: var(--accent);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.create-room-container {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.create-room-input {
  flex-grow: 1;
  padding: 10px;
  border-radius: 4px;
  border: none;
  background: rgba(255,255,255,0.1);
  color: var(--text);
}

.create-room-input:focus {
  outline: none;
  background: rgba(255,255,255,0.15);
}

/* Add these to the existing CSS for emoji support */
.emoji-picker {
  position: absolute;
  bottom: 100%;
  left: 0;
  background: var(--bg);
  border: 1px solid rgba(255,255,255,0.2);
  border-radius: 8px;
  padding: 10px;
  display: none;
  grid-template-columns: repeat(8, 1fr);
  gap: 5px;
  margin-bottom: 10px;
  max-width: 300px;
}

.emoji-picker.show {
  display: grid;
}

.emoji-btn {
  background: none;
  border: none;
  font-size: 1.2em;
  padding: 5px;
  cursor: pointer;
  border-radius: 4px;
}

.emoji-btn:hover {
  background: rgba(255,255,255,0.1);
}

.emoji-trigger {
  background: none;
  border: none;
  padding: 10px;
  cursor: pointer;
  font-size: 1.2em;
}
</style>
</head>
<body>
<div class="container">
  <h1>Voice Chat Hangout</h1>
  
  <div class="status" id="status">Please enable voice chat to begin</div>
  
  <!-- Room List -->
  <div class="room-list" id="roomList">
    <h2>Available Rooms</h2>
    <div class="create-room-container">
      <input type="text" class="create-room-input" id="roomNameInput" placeholder="Enter room name...">
      <button class="join-btn" id="createRoomBtn">Create Room</button>
    </div>
    <div id="roomsContainer">
      <!-- Rooms will be added here dynamically -->
    </div>
  </div>
  
  <!-- Add enable voice chat button before controls -->
  <button id="enableVoiceBtn">Enable Voice Chat</button>
  
  <div class="controls">
    <button id="muteBtn" disabled>Mute</button>
    <button id="leaveBtn" disabled>Leave Room</button>
  </div>
  
  <div class="user-list" id="userList">
    <!-- Users will be added here dynamically -->
  </div>

  <!-- Chat container -->
  <div class="chat-container">
    <div class="chat-messages" id="chatMessages"></div>
    <div class="chat-input-container">
      <div class="emoji-picker" id="emojiPicker">
        <!-- Emojis will be added here by JavaScript -->
      </div>
      <button class="emoji-trigger" id="emojiTrigger">😊</button>
      <input type="text" class="chat-input" id="chatInput" placeholder="Type a message...">
      <button class="send-btn" id="sendBtn">Send</button>
    </div>
  </div>
</div>

<script>
const status = document.getElementById('status');
const userList = document.getElementById('userList');
const muteBtn = document.getElementById('muteBtn');
const leaveBtn = document.getElementById('leaveBtn');
const enableVoiceBtn = document.getElementById('enableVoiceBtn');

// Add these new variables for room functionality
const roomList = document.getElementById('roomList');
const roomsContainer = document.getElementById('roomsContainer');
const roomNameInput = document.getElementById('roomNameInput');
const createRoomBtn = document.getElementById('createRoomBtn');

// Add these new variables for chat functionality
const chatInput = document.getElementById('chatInput');
const sendBtn = document.getElementById('sendBtn');
const chatMessages = document.getElementById('chatMessages');

// Add these constants for emoji functionality
const emojiTrigger = document.getElementById('emojiTrigger');
const emojiPicker = document.getElementById('emojiPicker');

// Add an array of common emojis
const emojis = [
  '😊', '😂', '🥰', '😎', '😍', '🤔', '😅', '😆',
  '👋', '👍', '🎉', '✨', '💖', '🎵', '🎮', '😴',
  '🤣', '😇', '🥺', '😋', '😉', '🙂', '😄', '😃'
];

let peer;
let myStream;
let connections = {};
let muted = false;
let voiceEnabled = false;

// Mock rooms data (in a real app, this would come from a server)
let rooms = [
  { id: 'room1', name: 'Casual Chat', users: 3 },
  { id: 'room2', name: 'Music Lovers', users: 5 },
  { id: 'room3', name: 'Gaming Zone', users: 2 }
];

async function init() {
  try {
    const userId = Math.random().toString(36).substr(2, 9);
    
    peer = new Peer(userId, {
      host: 'peerjs-server.herokuapp.com',
      secure: true,
      port: 443
    });

    peer.on('open', (id) => {
      status.textContent = 'Connected! Your ID: ' + id + '. Enable voice chat to begin.';
      addUser(id, true);
    });

    peer.on('call', (call) => {
      if (myStream) {
        call.answer(myStream);
        handleCall(call);
      }
    });

    peer.on('connection', handleDataConnection);

    peer.on('error', (err) => {
      console.error(err);
      status.textContent = 'Error: ' + err.message;
    });

  } catch (err) {
    console.error(err);
    status.textContent = 'Error: ' + err.message;
  }
}

// Initialize emoji picker
function initEmojiPicker() {
  emojis.forEach(emoji => {
    const button = document.createElement('button');
    button.className = 'emoji-btn';
    button.textContent = emoji;
    button.onclick = () => {
      chatInput.value += emoji;
      emojiPicker.classList.remove('show');
      chatInput.focus();
    };
    emojiPicker.appendChild(button);
  });
}

// Toggle emoji picker
emojiTrigger.addEventListener('click', () => {
  emojiPicker.classList.toggle('show');
});

// Close emoji picker when clicking outside
document.addEventListener('click', (e) => {
  if (!emojiPicker.contains(e.target) && !emojiTrigger.contains(e.target)) {
    emojiPicker.classList.remove('show');
  }
});

// Add enable voice chat functionality
enableVoiceBtn.addEventListener('click', async () => {
  try {
    if (!voiceEnabled) {
      myStream = await navigator.mediaDevices.getUserMedia({ audio: true });
      voiceEnabled = true;
      enableVoiceBtn.textContent = 'Voice Chat Enabled';
      enableVoiceBtn.classList.add('enabled');
      status.textContent = 'Voice chat enabled! You can now communicate with others.';
      muteBtn.disabled = false;
      leaveBtn.disabled = false;
    }
  } catch (err) {
    console.error(err);
    status.textContent = 'Error getting microphone access: ' + err.message;
  }
});

let dataConnections = {};

function handleDataConnection(conn) {
  conn.on('data', (data) => {
    if (data.type === 'chat') {
      addMessage(data.message, data.userId);
    }
  });
}

function handleCall(call) {
  call.on('stream', (remoteStream) => {
    addUser(call.peer);
    const audio = new Audio();
    audio.srcObject = remoteStream;
    audio.play();
    
    connections[call.peer] = {
      call,
      audio
    };
  });

  call.on('close', () => {
    removeUser(call.peer);
    delete connections[call.peer];
  });
}

function addUser(userId, isMe = false) {
  const userDiv = document.createElement('div');
  userDiv.className = 'user';
  userDiv.id = `user-${userId}`;
  
  const avatar = document.createElement('div');
  avatar.className = 'avatar';
  avatar.textContent = userId.substr(0, 2).toUpperCase();
  
  const name = document.createElement('span');
  name.textContent = isMe ? 'You' : `User ${userId.substr(0, 4)}`;
  
  const indicator = document.createElement('div');
  indicator.className = 'audio-indicator';
  
  userDiv.appendChild(avatar);
  userDiv.appendChild(name);
  userDiv.appendChild(indicator);
  userList.appendChild(userDiv);
}

function removeUser(userId) {
  const userDiv = document.getElementById(`user-${userId}`);
  if (userDiv) {
    userDiv.remove();
  }
}

muteBtn.addEventListener('click', () => {
  muted = !muted;
  myStream.getAudioTracks().forEach(track => {
    track.enabled = !muted;
  });
  muteBtn.textContent = muted ? 'Unmute' : 'Mute';
  
  if (muted) {
    enableVoiceBtn.textContent = 'Voice Chat Disabled';
    enableVoiceBtn.classList.remove('enabled');
    voiceEnabled = false;
  } else {
    enableVoiceBtn.textContent = 'Voice Chat Enabled';
    enableVoiceBtn.classList.add('enabled');
    voiceEnabled = true;
  }
});

leaveBtn.addEventListener('click', () => {
  Object.values(connections).forEach(({ call }) => call.close());
  Object.values(dataConnections).forEach(conn => conn.close());
  dataConnections = {};
  peer.destroy();
  
  if (myStream) {
    myStream.getTracks().forEach(track => track.stop());
  }
  
  status.textContent = 'Left the room';
  userList.innerHTML = '';
  chatMessages.innerHTML = '';
  muteBtn.disabled = true;
  leaveBtn.disabled = true;
  sendBtn.disabled = true;
  chatInput.disabled = true;
  
  voiceEnabled = false;
  enableVoiceBtn.textContent = 'Enable Voice Chat';
  enableVoiceBtn.classList.remove('enabled');
  
  // Show room list after leaving
  roomList.style.display = 'block';
  displayRooms();
});

// Connect to a peer using their ID
window.connectToPeer = function(peerId) {
  if (peer && voiceEnabled && myStream) {
    const call = peer.call(peerId, myStream);
    handleCall(call);

    const conn = peer.connect(peerId);
    conn.on('open', () => {
      dataConnections[peerId] = conn;
    });
    handleDataConnection(conn);
  } else if (!voiceEnabled) {
    status.textContent = 'Please enable voice chat first';
  }
};

function addMessage(message, userId) {
  const messageDiv = document.createElement('div');
  messageDiv.className = `message ${userId === peer.id ? 'mine' : 'others'}`;
  messageDiv.textContent = `${userId === peer.id ? 'You' : 'User ' + userId.substr(0, 4)}: ${message}`;
  chatMessages.appendChild(messageDiv);
  chatMessages.scrollTop = chatMessages.scrollHeight;
}

function sendMessage(message) {
  if (message.trim()) {
    addMessage(message, peer.id);
    Object.values(dataConnections).forEach(conn => {
      conn.send({
        type: 'chat',
        message: message,
        userId: peer.id
      });
    });
  }
}

// Chat input handlers
sendBtn.addEventListener('click', () => {
  sendMessage(chatInput.value);
  chatInput.value = '';
});

chatInput.addEventListener('keypress', (e) => {
  if (e.key === 'Enter') {
    sendMessage(chatInput.value);
    chatInput.value = '';
  }
});

// Room functionality
function displayRooms() {
  roomsContainer.innerHTML = '';
  rooms.forEach(room => {
    const roomDiv = document.createElement('div');
    roomDiv.className = 'room';
    roomDiv.innerHTML = `
      <div class="room-info">
        <div class="room-name">${room.name}</div>
        <div class="room-users">${room.users} users online</div>
      </div>
      <button class="join-btn" onclick="joinRoom('${room.id}')">Join</button>
    `;
    roomsContainer.appendChild(roomDiv);
  });
}

function joinRoom(roomId) {
  const room = rooms.find(r => r.id === roomId);
  
  if (room) {
    roomList.style.display = 'none';
    status.textContent = `Joined room: ${room.name}`;
    init(); // Reinitialize peer connection for the new room
  }
}

createRoomBtn.addEventListener('click', () => {
  const roomName = roomNameInput.value.trim();
  if (roomName) {
    const newRoom = {
      id: 'room' + (rooms.length + 1),
      name: roomName,
      users: 1
    };
    rooms.push(newRoom);
    roomNameInput.value = '';
    displayRooms();
    joinRoom(newRoom.id);
  }
});

// Initialize room list
displayRooms();

// Initialize when the page loads
init();
initEmojiPicker();
</script>
</body></html>
