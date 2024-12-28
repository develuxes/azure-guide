# `Azure`
Azure provides a service called Azure Web PubSub, which is designed for real-time communication using WebSocket protocols. This is a great alternative to AWS API Gateway’s WebSocket Server, allowing you to implement a chat system in Azure without relying on libraries like Socket.io.


Here’s a step-by-step guide and code samples to achieve this:




## 1. Set Up Azure Web PubSub
### 1) 	Create an Azure Web PubSub Service:
	•	Go to the Azure portal
https://login.microsoftonline.com/organizations/oauth2/v2.0/authorize?redirect_uri=https%3A%2F%2Fportal.azure.com%2Fsignin%2Findex%2F&response_type=code%20id_token&scope=https%3A%2F%2Fmanagement.core.windows.net%2F%2Fuser_impersonation%20openid%20email%20profile&state=OpenIdConnect.AuthenticationProperties%3DmQXIESW6w6l7HD4NB7DHcw1hLXv_nA_ugRPTGcPlunpzvxZiURh3o0-ReIcctyBYfPBNFrznsOufioRa1O4so9e2NowGyEVp6bQOwBHB5Eq5MQFAPeBV4k4PhLdr3n8CeAX8TWU7kKn72oC0INZB50TdKr4dj5MVzcamSEwWTp61Zea65Lwub_9Xy8BQkZW5GYmTxVy-cO8X17SLOLZuP3U6AOZM956JTH7NCE5U7zveB5MPvePyFVz-kiEvomGkpOdsoQwPKIrphbrhOC8tL_AlHiiXpmJj5f7FeiVOPS4xYZaj__o3zDFZ1JnY9zHLNMnfAKZVa7a-97gk1Vcr3F6RYP6r3xrqqHAL5lOclPZVXRfkqLlPZ6w0ArazzDT8mwNjo1NGLeieZsT7q17T3IS5M0O_JcCdmk9o1_BsJSXM8V35MSyBA7l8CBzgzYttoFwrdYl_j0Ts19G4-qgwcpAEB-c-62t7I64vQSbB8BA&response_mode=form_post&nonce=638709993464855633.MjE2NWMyYjgtNWZiNy00ZWQwLTllNjEtNmE1ZjliNGVmZWI1ZWU1M2E1YTAtNTM1Mi00M2ZiLTk3ZDUtY2NlNDNmYWEyN2U1&client_id=c44b4083-3bb0-49c1-b47d-974e53cbdf3c&site_id=501430&client-request-id=002d2569-754c-4086-8fd7-e9b06a9c22a5&x-client-SKU=ID_NET472&x-client-ver=7.5.0.0

	•	Search for “Web PubSub” and create a new service.
	•	Choose a subscription, resource group, and name for your service.
	•	Select a pricing tier (Free Tier is available for testing).


### 2) 	Enable Public WebSocket Client Connections:
  	•	Navigate to Settings > Features in your Web PubSub resource.
	•	Enable “Client connections.”

### 3) 	Enable Public WebSocket Client Connections:
    •	Go to Keys under Settings and copy the connection string.

## 2. Backend Code
You need a backend server to issue Web PubSub access tokens. Azure provides SDKs for this purpose.

``Install Azure Web PubSub SDK``

    npm install @azure/web-pubsub
  
``Node.js Backend Code``


```js
const { WebPubSubServiceClient } = require('@azure/web-pubsub');

const connectionString = "<Your Azure Web PubSub Connection String>";
const hubName = "chat"; // Name your hub (e.g., "chat")

const serviceClient = new WebPubSubServiceClient(connectionString, hubName);

// Generate a client access URL
function generateClientUrl(userId) {
    return serviceClient.getClientAccessUrl({ userId });
}

// Example endpoint for your React frontend to get the WebSocket URL
const express = require('express');
const app = express();

app.get('/getSocketUrl', (req, res) => {
    const userId = req.query.userId || "anonymous";
    const url = generateClientUrl(userId);
    res.json({ url });
});

const PORT = 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

## 3. Fronted Code

Your React application connects to the WebSocket URL provided by the backend.

``React Frontend Code``

```js
import React, { useState, useEffect } from 'react';

const ChatApp = () => {
    const [messages, setMessages] = useState([]);
    const [newMessage, setNewMessage] = useState('');
    const [ws, setWs] = useState(null);

    useEffect(() => {
        // Fetch the WebSocket URL from the backend
        fetch('/getSocketUrl?userId=123')
            .then((res) => res.json())
            .then(({ url }) => {
                const socket = new WebSocket(url);

                // Handle incoming messages
                socket.onmessage = (event) => {
                    const data = JSON.parse(event.data);
                    setMessages((prev) => [...prev, data]);
                };

                setWs(socket);
            });

        return () => {
            if (ws) ws.close();
        };
    }, []);

    const sendMessage = () => {
        if (ws && newMessage.trim()) {
            ws.send(JSON.stringify({ type: 'message', content: newMessage }));
            setNewMessage('');
        }
    };

    return (
        <div>
            <h1>Chat App</h1>
            <div>
                {messages.map((msg, index) => (
                    <p key={index}>{msg.content}</p>
                ))}
            </div>
            <input
                type="text"
                value={newMessage}
                onChange={(e) => setNewMessage(e.target.value)}
            />
            <button onClick={sendMessage}>Send</button>
        </div>
    );
};

export default ChatApp;
```


## 4. Features and Tutorials
	•	Broadcast Messages: Azure Web PubSub supports broadcasting messages to all connected clients or specific groups.
	•	Groups and Permissions: You can use groups to manage users and control message visibility.


``Broadcast Example``

```js
async function broadcastMessage(message) {
    await serviceClient.sendToAll({ type: 'message', content: message });
}
```

## 5. Resources

https://learn.microsoft.com/en-us/azure/azure-web-pubsub/overview


https://learn.microsoft.com/en-us/azure/azure-web-pubsub/quickstart-javascript

https://github.com/Azure/azure-webpubsub