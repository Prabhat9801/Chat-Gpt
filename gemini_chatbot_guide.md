# Complete Gemini API Chatbot Guide

This guide will help you create a fully functional chatbot using Google's Gemini API with a modern web interface.

## Directory Structure

```
gemini-chatbot/
├── backend/
│   ├── server.js
│   ├── package.json
│   └── .env
├── frontend/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── package.json
├── README.md
└── .gitignore
```

## Prerequisites

1. Node.js (version 14 or higher)
2. npm or yarn
3. Google AI Studio account for Gemini API key

## Getting Your Gemini API Key

1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click "Create API Key"
4. Copy the generated API key (you'll need this later)

## Installation Process

### Step 1: Create Project Directory

```bash
mkdir gemini-chatbot
cd gemini-chatbot
```

### Step 2: Set Up Backend

```bash
mkdir backend
cd backend
npm init -y
npm install express cors dotenv @google/generative-ai
cd ..
```

### Step 3: Set Up Frontend

```bash
mkdir frontend
cd frontend
npm init -y
npm install --save-dev live-server
cd ..
```

## Code Files

### Backend Files

#### `backend/package.json`

```json
{
  "name": "gemini-chatbot-backend",
  "version": "1.0.0",
  "description": "Backend for Gemini AI Chatbot",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "@google/generative-ai": "^0.2.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

#### `backend/.env`

```env
GEMINI_API_KEY=your_gemini_api_key_here
PORT=3000
```

#### `backend/server.js`

```javascript
const express = require('express');
const cors = require('cors');
const { GoogleGenerativeAI } = require('@google/generative-ai');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Initialize Gemini AI
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-pro" });

// Middleware
app.use(cors());
app.use(express.json());

// Store conversation history
let conversationHistory = [];

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'Server is running!' });
});

app.post('/chat', async (req, res) => {
  try {
    const { message } = req.body;
    
    if (!message) {
      return res.status(400).json({ error: 'Message is required' });
    }

    // Add user message to history
    conversationHistory.push({ role: 'user', content: message });

    // Create context from conversation history
    const context = conversationHistory
      .map(msg => `${msg.role}: ${msg.content}`)
      .join('\n');

    // Generate response
    const result = await model.generateContent(context);
    const response = await result.response;
    const botReply = response.text();

    // Add bot response to history
    conversationHistory.push({ role: 'assistant', content: botReply });

    // Keep only last 10 exchanges to manage context length
    if (conversationHistory.length > 20) {
      conversationHistory = conversationHistory.slice(-20);
    }

    res.json({ 
      reply: botReply,
      timestamp: new Date().toISOString()
    });

  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ 
      error: 'Failed to generate response',
      details: error.message 
    });
  }
});

app.post('/clear', (req, res) => {
  conversationHistory = [];
  res.json({ message: 'Conversation history cleared' });
});

app.get('/history', (req, res) => {
  res.json({ history: conversationHistory });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Frontend Files

#### `frontend/package.json`

```json
{
  "name": "gemini-chatbot-frontend",
  "version": "1.0.0",
  "description": "Frontend for Gemini AI Chatbot",
  "main": "index.html",
  "scripts": {
    "start": "live-server --port=8080 --open=/index.html"
  },
  "devDependencies": {
    "live-server": "^1.2.2"
  }
}
```

#### `frontend/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemini AI Chatbot</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🤖 Gemini AI Chatbot</h1>
            <button id="clearBtn" class="clear-btn">Clear Chat</button>
        </header>
        
        <div class="chat-container">
            <div id="chatMessages" class="chat-messages">
                <div class="message bot-message">
                    <div class="message-content">
                        <strong>Gemini:</strong> Hello! I'm your AI assistant powered by Google's Gemini. How can I help you today?
                    </div>
                    <div class="message-time" data-time=""></div>
                </div>
            </div>
        </div>
        
        <div class="input-container">
            <div class="input-wrapper">
                <input type="text" id="messageInput" placeholder="Type your message here..." autocomplete="off">
                <button id="sendBtn">Send</button>
            </div>
            <div id="typing" class="typing-indicator" style="display: none;">
                <span></span>
                <span></span>
                <span></span>
            </div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
```

#### `frontend/style.css`

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

.container {
    width: 90%;
    max-width: 800px;
    height: 90vh;
    background: white;
    border-radius: 20px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

header h1 {
    font-size: 1.5rem;
    font-weight: 600;
}

.clear-btn {
    background: rgba(255, 255, 255, 0.2);
    color: white;
    border: 1px solid rgba(255, 255, 255, 0.3);
    padding: 8px 16px;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.3s ease;
}

.clear-btn:hover {
    background: rgba(255, 255, 255, 0.3);
}

.chat-container {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
}

.chat-messages {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.message {
    max-width: 80%;
    animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
}

.user-message {
    align-self: flex-end;
}

.user-message .message-content {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 12px 18px;
    border-radius: 20px 20px 5px 20px;
    word-wrap: break-word;
}

.bot-message {
    align-self: flex-start;
}

.bot-message .message-content {
    background: #f8f9fa;
    color: #333;
    padding: 12px 18px;
    border-radius: 20px 20px 20px 5px;
    border: 1px solid #e9ecef;
    word-wrap: break-word;
}

.message-time {
    font-size: 0.75rem;
    color: #888;
    margin-top: 5px;
    text-align: right;
}

.bot-message .message-time {
    text-align: left;
}

.input-container {
    padding: 20px;
    background: #f8f9fa;
    border-top: 1px solid #e9ecef;
}

.input-wrapper {
    display: flex;
    gap: 10px;
    align-items: center;
}

#messageInput {
    flex: 1;
    padding: 12px 18px;
    border: 2px solid #e9ecef;
    border-radius: 25px;
    font-size: 16px;
    outline: none;
    transition: border-color 0.3s ease;
}

#messageInput:focus {
    border-color: #667eea;
}

#sendBtn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 25px;
    cursor: pointer;
    font-weight: 600;
    transition: transform 0.2s ease;
}

#sendBtn:hover {
    transform: translateY(-2px);
}

#sendBtn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
}

.typing-indicator {
    display: flex;
    align-items: center;
    gap: 4px;
    margin-top: 10px;
    color: #888;
}

.typing-indicator span {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #888;
    animation: typing 1.4s infinite ease-in-out;
}

.typing-indicator span:nth-child(1) { animation-delay: -0.32s; }
.typing-indicator span:nth-child(2) { animation-delay: -0.16s; }

@keyframes typing {
    0%, 80%, 100% { transform: scale(0); }
    40% { transform: scale(1); }
}

/* Scrollbar styling */
.chat-container::-webkit-scrollbar {
    width: 6px;
}

.chat-container::-webkit-scrollbar-track {
    background: #f1f1f1;
}

.chat-container::-webkit-scrollbar-thumb {
    background: #c1c1c1;
    border-radius: 3px;
}

.chat-container::-webkit-scrollbar-thumb:hover {
    background: #a8a8a8;
}

/* Responsive design */
@media (max-width: 768px) {
    .container {
        width: 95%;
        height: 95vh;
        border-radius: 15px;
    }
    
    header {
        padding: 15px;
    }
    
    header h1 {
        font-size: 1.2rem;
    }
    
    .message {
        max-width: 90%;
    }
    
    .input-container {
        padding: 15px;
    }
}
```

#### `frontend/script.js`

```javascript
class GeminiChatbot {
    constructor() {
        this.apiUrl = 'http://localhost:3000';
        this.chatMessages = document.getElementById('chatMessages');
        this.messageInput = document.getElementById('messageInput');
        this.sendBtn = document.getElementById('sendBtn');
        this.clearBtn = document.getElementById('clearBtn');
        this.typingIndicator = document.getElementById('typing');
        
        this.initializeEventListeners();
        this.setInitialTime();
    }

    initializeEventListeners() {
        this.sendBtn.addEventListener('click', () => this.sendMessage());
        this.messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                this.sendMessage();
            }
        });
        this.clearBtn.addEventListener('click', () => this.clearChat());
    }

    setInitialTime() {
        const timeElement = document.querySelector('.message-time');
        if (timeElement) {
            timeElement.textContent = this.formatTime(new Date());
        }
    }

    async sendMessage() {
        const message = this.messageInput.value.trim();
        if (!message) return;

        // Disable input while processing
        this.toggleInput(false);
        
        // Add user message to chat
        this.addMessage(message, 'user');
        this.messageInput.value = '';

        // Show typing indicator
        this.showTyping(true);

        try {
            const response = await fetch(`${this.apiUrl}/chat`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ message })
            });

            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }

            const data = await response.json();
            
            // Hide typing indicator
            this.showTyping(false);
            
            // Add bot response to chat
            this.addMessage(data.reply, 'bot', data.timestamp);
            
        } catch (error) {
            console.error('Error:', error);
            this.showTyping(false);
            this.addMessage('Sorry, I encountered an error. Please try again.', 'bot');
        } finally {
            this.toggleInput(true);
            this.messageInput.focus();
        }
    }

    addMessage(content, sender, timestamp = null) {
        const messageDiv = document.createElement('div');
        messageDiv.className = `message ${sender}-message`;
        
        const messageContent = document.createElement('div');
        messageContent.className = 'message-content';
        
        if (sender === 'user') {
            messageContent.innerHTML = `<strong>You:</strong> ${this.escapeHtml(content)}`;
        } else {
            messageContent.innerHTML = `<strong>Gemini:</strong> ${this.formatBotResponse(content)}`;
        }
        
        const messageTime = document.createElement('div');
        messageTime.className = 'message-time';
        messageTime.textContent = this.formatTime(timestamp ? new Date(timestamp) : new Date());
        
        messageDiv.appendChild(messageContent);
        messageDiv.appendChild(messageTime);
        this.chatMessages.appendChild(messageDiv);
        
        // Scroll to bottom
        this.chatMessages.scrollTop = this.chatMessages.scrollHeight;
    }

    formatBotResponse(content) {
        // Basic markdown-like formatting
        return this.escapeHtml(content)
            .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
            .replace(/\*(.*?)\*/g, '<em>$1</em>')
            .replace(/`(.*?)`/g, '<code>$1</code>')
            .replace(/\n/g, '<br>');
    }

    escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }

    formatTime(date) {
        return date.toLocaleTimeString([], { 
            hour: '2-digit', 
            minute: '2-digit' 
        });
    }

    toggleInput(enabled) {
        this.messageInput.disabled = !enabled;
        this.sendBtn.disabled = !enabled;
    }

    showTyping(show) {
        this.typingIndicator.style.display = show ? 'flex' : 'none';
    }

    async clearChat() {
        try {
            await fetch(`${this.apiUrl}/clear`, { method: 'POST' });
            
            // Clear chat messages except the initial greeting
            this.chatMessages.innerHTML = `
                <div class="message bot-message">
                    <div class="message-content">
                        <strong>Gemini:</strong> Hello! I'm your AI assistant powered by Google's Gemini. How can I help you today?
                    </div>
                    <div class="message-time">${this.formatTime(new Date())}</div>
                </div>
            `;
        } catch (error) {
            console.error('Error clearing chat:', error);
        }
    }
}

// Initialize chatbot when page loads
document.addEventListener('DOMContentLoaded', () => {
    new GeminiChatbot();
});
```

### Root Files

#### `.gitignore`

```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Logs
logs
*.log

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/

# nyc test coverage
.nyc_output

# Dependency directories
node_modules/
jspm_packages/

# Optional npm cache directory
.npm

# Optional eslint cache
.eslintcache

# Output of 'npm pack'
*.tgz

# Yarn Integrity file
.yarn-integrity

# dotenv environment variables file
.env

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
```

## Running the Application

### Step 1: Configure Environment Variables

1. Navigate to the `backend` directory
2. Open the `.env` file
3. Replace `your_gemini_api_key_here` with your actual Gemini API key

### Step 2: Start the Backend Server

```bash
cd backend
npm install
npm start
```

The backend server will start on `http://localhost:3000`

### Step 3: Start the Frontend (In a new terminal)

```bash
cd frontend
npm install
npm start
```

The frontend will open automatically in your browser at `http://localhost:8080`

## Development Mode

For development with auto-restart:

```bash
# Install nodemon globally (optional)
npm install -g nodemon

# Start backend in development mode
cd backend
npm install nodemon --save-dev
npx nodemon server.js
```

## Features

- **Real-time Chat**: Instant messaging with Gemini AI
- **Conversation Memory**: Maintains context throughout the conversation
- **Modern UI**: Responsive design with smooth animations
- **Message History**: Persistent conversation history
- **Clear Chat**: Reset conversation anytime
- **Error Handling**: Graceful error management
- **Typing Indicators**: Visual feedback during AI response generation
- **Mobile Responsive**: Works on all device sizes

## API Endpoints

- `GET /health` - Check server status
- `POST /chat` - Send message to Gemini AI
- `POST /clear` - Clear conversation history
- `GET /history` - Get conversation history

## Troubleshooting

### Common Issues

1. **API Key Error**: Make sure your Gemini API key is correctly set in the `.env` file
2. **CORS Error**: Ensure the backend server is running on port 3000
3. **Network Error**: Check if both frontend and backend servers are running
4. **Rate Limiting**: Gemini API has rate limits; wait a moment before retrying

### Port Conflicts

If ports 3000 or 8080 are already in use:

```bash
# Change backend port
PORT=3001 npm start

# Change frontend port
npx live-server --port=8081
```

Remember to update the `apiUrl` in `script.js` if you change the backend port.

## Next Steps

- Add user authentication
- Implement message persistence with a database
- Add file upload capabilities
- Create mobile app version
- Add voice input/output
- Deploy to cloud platforms

## Support

If you encounter any issues:
1. Check the console for error messages
2. Verify your API key is valid
3. Ensure all dependencies are installed
4. Check that both servers are running

Happy coding! 🚀