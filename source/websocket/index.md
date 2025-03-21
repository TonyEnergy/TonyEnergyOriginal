---
title: WebSocket Test
date: 2025-03-20 12:00:00
---

<h2>WebSocket Client</h2>
<button onclick="sendMessage()">Send Message</button>
<p id="response">Waiting for response...</p>

<script>
    const socket = new WebSocket("wss://ocpp-java-app.onrender.com/ws");

    socket.onopen = function() {
        console.log("Connected to WebSocket!");
    };

    socket.onmessage = function(event) {
        document.getElementById("response").innerText = "Server says: " + event.data;
    };

    socket.onclose = function() {
        console.log("Disconnected from WebSocket");
    };

    function sendMessage() {
        if (socket.readyState === WebSocket.OPEN) {
            socket.send("Hello Server!");
        } else {
            alert("WebSocket is not connected!");
        }
    }
</script>
