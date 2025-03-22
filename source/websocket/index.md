---
title: OCPP EV Charger WebSocket Client
date: 2025-03-20 12:00:00
---

# OCPP WebSocket Client

<div id="serverStatus" style="width: 100%; margin: 10px 0; padding: 15px; text-align: center; background-color: #ffcc80; color: #6d4c41; font-weight: bold; border-radius: 5px;">
⚠️ If this is your first time visiting this page, the backend service might be asleep and could take about 1 minute to start. Please wait until this box turns green...
</div>

<div class="container" style="width: 100%; max-width: none; margin: 20px; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); text-align: left;">

<!-- First row: Create Charger Instance -->
<div class="row" style="margin-bottom: 20px;">
    <label for="chargerId">Create a Charger</label>
    <input type="text" id="chargerId" placeholder="Enter Charger ID" style="padding: 5px; width: 40%;">
    <button id="createButton" class="send-btn">Send</button>
</div>

<!-- Second row: Get Configuration -->
<div class="row" style="margin-bottom: 20px;">
    <label for="getConfigInput">Get Configuration</label>
    <input type="text" id="getConfigInput" placeholder="Enter config key" style="padding: 5px; width: 40%; margin-left: 10px;">
    <button id="getConfigButton" class="send-btn">Send</button>
</div>

<!-- Third row: Change Configuration -->
<div class="row" style="margin-bottom: 20px;">
    <label for="changeConfigInput">Change Configuration</label>
    <input type="text" id="changeConfigInput" placeholder="Enter config key" style="padding: 5px; width: 40%; margin-left: 10px;">
    <button id="changeConfigButton" class="send-btn">Send</button>
</div>

<p id="response" style="margin-top: 15px; padding: 10px; background: #d4edda; border-radius: 5px; color: black;">Waiting for response...</p>

<!-- Log Section -->
<button id="toggleLog" style="margin-top: 20px; padding: 5px 10px; background-color: #6c757d; color: white; border: none; border-radius: 5px; cursor: pointer;">Show Logs</button>
<button id="clearLog" style="margin-top: 20px; padding: 5px 10px; background-color: #dc3545; color: white; border: none; border-radius: 5px; cursor: pointer;">Clear Logs</button>

<div class="container" style="width: 100%; max-width: none; margin: 20px; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); text-align: left;">
    <table border="1" style="border-collapse: collapse; width: 100%; table-layout: fixed; text-align: left; word-wrap: break-word;">
        <thead>
            <tr>
                <th>Number</th>
                <th>Time</th>
                <th>Charger SN</th>
                <th>Command</th>
                <th>Parameters</th>
                <th>Response Body</th>
            </tr>
        </thead>
        <tbody id="logTable"></tbody>
    </table>
</div>


</div>

<script>
    let socket;
    let pingInterval;
    const serverStatus = document.getElementById("serverStatus");
    const responseElement = document.getElementById("response");
    let logData = JSON.parse(localStorage.getItem("logData")) || [];

    function connectWebSocket() {
    const wsUrl = "wss://ocpp-java-app.onrender.com/ws";
    socket = new WebSocket(wsUrl);

    socket.onopen = function () {
        responseElement.style.background = "#d4edda";
        responseElement.innerText = "Connected to WebSocket!";
        serverStatus.style.background = "#c8e6c9";
        serverStatus.style.color = "#2e7d32";
        serverStatus.innerText = "✅ Server is up and WebSocket is connected!";

        pingInterval = setInterval(() => {
            if (socket.readyState === WebSocket.OPEN) {
                const heartbeat = { currentTime: new Date().toISOString() };
                socket.send(JSON.stringify({ action: "Heartbeat", parameters: {} }));
                addLogEntry("SYSTEM", "Heartbeat", "{}", JSON.stringify(heartbeat));  // 添加日志记录
            }
        }, 60000);
    };

    socket.onmessage = function (event) {
        responseElement.style.background = "#d4edda";
        responseElement.innerText = "Server says: " + event.data;
    };

    socket.onclose = function () {
        responseElement.style.background = "#f8d7da";
        responseElement.innerText = "Disconnected from WebSocket!";
        serverStatus.style.background = "#ffcdd2";
        serverStatus.innerText = "❌ Server is not responding.";
        clearInterval(pingInterval);
    };
}


    function sendMessage(action, key, value = null) {
        if (!socket || socket.readyState !== WebSocket.OPEN) {
            alert("WebSocket is not connected!");
            return;
        }

        let message = { action, key };
        if (value !== null) message.value = value;
        
        socket.send(JSON.stringify(message));
        addLogEntry("Client", action, JSON.stringify(message), "Sent");
    }

    function addLogEntry(sn, command, parameters, responseBody) {
        const now = new Date().toISOString().replace("T", " ").substring(0, 19);
        logData.push({ id: logData.length + 1, time: now, sn, command, parameters, responseBody });
        localStorage.setItem("logData", JSON.stringify(logData));
        updateLogTable();
    }

    function updateLogTable() {
        document.getElementById("logTable").innerHTML = logData.map(log => `
            <tr>
                <td>${log.id}</td>
                <td>${log.time}</td>
                <td>${log.sn}</td>
                <td>${log.command}</td>
                <td>${log.parameters}</td>
                <td>${log.responseBody}</td>
            </tr>
        `).join("");
    }

    document.getElementById("createButton").addEventListener("click", function () {
        const chargerId = document.getElementById("chargerId").value.trim();
        if (!chargerId) return alert("Please enter a Charger ID.");
        sendMessage("createCharger", chargerId);
    });

    document.getElementById("getConfigButton").addEventListener("click", function () {
        const configKey = document.getElementById("getConfigInput").value.trim();
        if (!configKey) return alert("Please enter a config key.");
        sendMessage("getConfiguration", configKey);
    });

    document.getElementById("changeConfigButton").addEventListener("click", function () {
        const configKey = document.getElementById("changeConfigInput").value.trim();
        if (!configKey) return alert("Please enter a config key.");
        sendMessage("changeConfiguration", configKey, "newValue");
    });

    document.getElementById("toggleLog").addEventListener("click", function () {
        const logContainer = document.getElementById("logContainer");
        logContainer.style.display = logContainer.style.display === "none" ? "block" : "none";
        this.innerText = logContainer.style.display === "none" ? "Show Logs" : "Hide Logs";
    });

    document.getElementById("clearLog").addEventListener("click", function () {
        localStorage.removeItem("logData");
        logData = [];
        updateLogTable();
    });

    connectWebSocket();
    updateLogTable();
</script>
