
# Nexus Chat Room

Nexus Chat is a simple and real-time chat application built using Flask and Socket.IO that allows users to join or create chat rooms. Each chat room has its own unique code, and users can exchange messages with others in the same room.

## Features
- **Create a Room**: Generate a unique room code and start chatting instantly.
- **Join a Room**: Enter an existing room using a room code to begin chatting with others.
- **Real-time Messaging**: Send and receive messages in real-time using WebSockets.
- **Session Management**: Keep track of users' sessions and ensure a seamless experience.
- **User Notifications**: Notify users when someone enters or leaves the room.

## Screenshots
- **Home Page**
- **Chat Room**
- **Message Sending**

## Requirements
- Python 3.x
- Flask
- Flask-SocketIO

You can install the necessary dependencies by running the following command:

```bash
pip install flask flask-socketio

```

## How to Run

1.  Clone the repository:
    
    ```bash
    git clone <repository-url>
    cd <project-folder>
    
    ```
    
2.  Run the application:
    
    ```bash
    python main.py
    
    ```
    
    The application will be available at [http://127.0.0.1:5000/](http://127.0.0.1:5000/) in your browser.
    

## How It Works

### Home Page

On the home page, users can:

-   Enter their name: This helps identify them in the chat room.
-   Join an existing room: Enter the room code of an existing chat room to join.
-   Create a new room: Users can click on "Create a Room" to generate a unique room code.

### Chat Room

Once users have joined or created a room, they can send and receive messages in real-time. Each message includes:

-   The sender's name
-   The message text
-   A timestamp of when the message was sent

Messages appear in the chat room interface and are updated in real-time for all members of the room.

### WebSocket Integration (Socket.IO)

The chat application uses Socket.IO to establish WebSocket connections between the server and the client. Here’s how the WebSocket interaction works:

#### 1. Establishing a Connection

When a user joins or creates a room, the client establishes a WebSocket connection with the server through `socketio.connect()`. This allows real-time communication between the server and the client.

#### 2. User Join Event

When a user joins a room (either by creating or joining a room), the `connect` event is triggered on the server. The server then:

-   Adds the user to the specified room using `join_room()`.
-   Sends a notification to all other users in the room, indicating that the new user has entered using the `send()` method.

```python
@socketio.on("connect")
def connect(auth):
    room = session.get("room")
    name = session.get("name")
    if not room or not name:
        return
    if room not in rooms:
        leave_room(room)
        return
    
    join_room(room)
    send({"name": name, "message": "has entered the room"}, to=room)
    rooms[room]["members"] += 1

```

#### 3. Real-Time Messaging

Once users are connected to a room, they can send messages. When a user sends a message, the client emits a message event using `socketio.emit("message", data)` which is received on the server:

-   The server processes the message and broadcasts it to all users in the room using the `send()` method.
-   The message is stored in the room's history for future reference.

```python
@socketio.on("message")
def message(data):
    room = session.get("room")
    if room not in rooms:
        return

    content = {
        "name": session.get("name"),
        "message": data["data"]
    }
    send(content, to=room)
    rooms[room]["messages"].append(content)

```

#### 4. User Disconnect Event

When a user disconnects from the room, the `disconnect` event is triggered on the server. The server:

-   Removes the user from the room using `leave_room()`.
-   Sends a notification to all other users in the room, indicating that the user has left.
-   If the room has no more members, it is deleted from the server.

```python
@socketio.on("disconnect")
def disconnect():
    room = session.get("room")
    name = session.get("name")
    leave_room(room)

    if room in rooms:
        rooms[room]["members"] -= 1
        if rooms[room]["members"] <= 0:
            del rooms[room]

    send({"name": name, "message": "has left the room"}, to=room)

```

## Code Structure

-   `main.py`: The main Flask application file where routes, socket events, and logic are defined.
-   `templates/`: Contains the HTML templates for the home page and the chat room.
    -   `home.html`: The page where users can create or join rooms.
    -   `room.html`: The page where users interact with the chat room.
-   `static/`: This directory holds the CSS files.
    -   `style.css`: Contains the CSS styling for the chatroom's layout and appearance.

