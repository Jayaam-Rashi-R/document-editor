<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Real-Time Collaborative Document Editor</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 2rem; max-width: 800px; margin: auto; }
    textarea { width: 100%; height: 400px; padding: 1rem; border: 1px solid #ccc; border-radius: 8px; font-size: 1rem; }
    h1 { margin-bottom: 1rem; }
  </style>
</head>
<body>
  <h1>Real-Time Collaborative Document Editor</h1>
  <textarea id="editor"></textarea>

  <script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
  <script>
    const socket = io('http://localhost:5000');
    const editor = document.getElementById('editor');

    socket.on('receive-changes', (data) => {
      editor.value = data;
    });

    editor.addEventListener('input', () => {
      socket.emit('send-changes', editor.value);
    });
  </script>
</body>
</html>



const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const path = require('path');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: {
    origin: '*',
    methods: ['GET', 'POST']
  }
});

app.use(express.static(path.join(__dirname, '../public')));

io.on('connection', (socket) => {
  console.log('User connected:', socket.id);

  socket.on('send-changes', (data) => {
    socket.broadcast.emit('receive-changes', data);
  });

  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

server.listen(5000, () => {
  console.log('Server running on http://localhost:5000');
});
