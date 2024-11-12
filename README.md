# fincode.github.io

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

// Initialize app
const app = express();
app.use(cors());
app.use(express.json());  // For parsing application/json
app.use(express.static('public'));  // Serve static files (frontend)

// Connect to MongoDB
mongoose.connect('mongodb://localhost/chatboard', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.log('MongoDB connection error:', err));

// Define message schema
const messageSchema = new mongoose.Schema({
  username: { type: String, required: true },
  content: { type: String, required: true },
  timestamp: { type: Date, default: Date.now }
});

const Message = mongoose.model('Message', messageSchema);

// Route to get all messages
app.get('/messages', async (req, res) => {
  try {
    const messages = await Message.find().sort({ timestamp: -1 });
    res.json(messages);
  } catch (error) {
    res.status(500).send('Error retrieving messages');
  }
});

// Route to post a new message
app.post('/messages', async (req, res) => {
  const { username, content } = req.body;
  if (!username || !content) {
    return res.status(400).send('Username and message content are required');
  }

  try {
    const newMessage = new Message({ username, content });
    await newMessage.save();
    res.status(201).send('Message saved');
  } catch (error) {
    res.status(500).send('Error saving message');
  }
});

// Start the server
const port = 3000;
app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
