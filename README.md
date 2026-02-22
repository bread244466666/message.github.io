<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Firebase Chat App</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-database-compat.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Ubuntu+Mono:ital,wght@0,400;0,700;1,400;1,700&display=swap');

        * {
            font-family: "Ubuntu Mono", serif;
            box-sizing: border-box; /* Better sizing control */
        }

        /* 1. Ensure the viewport takes up the full screen height */
        html, body {
            margin: 0;
            padding: 0;
            height: 100vh; 
            width: 100vw;
            overflow: hidden; /* Prevents accidental scrollbars */
        }

        /* 2. Flexbox centering logic */
        body {
            display: flex;
            align-items: center;      /* Vertical center */
            justify-content: center;   /* Horizontal center */
            background-image: linear-gradient(to top, #48c6ef 0%, #6f86d6 100%);
        }

        #main-wrapper {
            width: 100%;
            max-width: 400px; /* Slightly wider for better readability */
            padding: 20px;
        }

        #app-title {
            color: #fff;
            text-shadow: 0px 3px 5px rgba(0,0,0,0.2);
            text-align: center;
            margin-bottom: 25px;
            font-size: 1.5rem;
        }

        #msg-txt-field {
            resize: none;
        }

        #send-btn-container {
            width: 70px;
        }

        #send-btn {
            width: 100%;
            height: 100%;
        }

        #message-items-container {
            height: 350px; /* Increased height */
            overflow-y: auto;
            display: flex;
            flex-flow: column;
            background: #fff;
        }

        #message-items-container:empty {
            align-items: center;
            justify-content: center;
        }

        #message-items-container:empty::before {
            content: "conversation box is currently empty...";
            font-size: 11px;
            font-style: italic;
            color: #858585;
        }

        .message-item {
            max-width: 85%;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 15px;
            word-wrap: break-word; /* Prevents long text from breaking layout */
        }

        .message-item.received {
            background-image: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            align-self: flex-start;
        }

        .message-item.sent {
            background-image: linear-gradient(120deg, #89f7fe 0%, #66a6ff 100%);
            align-self: flex-end;
            text-align: right;
        }

        .message-item .sender-name {
            font-style: italic;
            font-size: 11px;
            display: block;
            margin-bottom: 2px;
        }

        .message-item .message-content {
            font-size: 13px;
        }
    </style>
</head>
<body>
    <div id="main-wrapper">
        <h2 id="app-title"><strong> Chat</strong></h2>
        <div id="conversation-box" class="card shadow-lg border-0">
            <div class="card-header bg-white">
                <h5 class="card-title mb-0"><strong>Conversation</strong></h5>
            </div>
            <div class="card-body p-2">
                <div class="container-fluid" id="message-items-container"></div>
            </div>
            <div class="card-footer bg-light border-0">
                <div class="mb-2">
                    <small class="text-muted">User: <span id="userName" class="fw-bold"></span></small>
                </div>
                <div class="d-flex w-100 gap-2">
                    <div class="flex-grow-1">
                        <textarea id="msg-txt-field" rows="2" class="form-control form-control-sm" placeholder="Type a message..."></textarea>
                    </div>
                    <div id="send-btn-container">
                        <button class="btn btn-primary btn-sm h-100 w-100" id="send-btn">Send</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Your Firebase config
        const firebaseConfig = {
            apiKey: "AIzaSyC4oYXDJb5Lbb4B3nAqX-Hmxf18s13whyA",
            authDomain: "message-5ce86.firebaseapp.com",
            databaseURL: "https://message-5ce86-default-rtdb.firebaseio.com",
            projectId: "message-5ce86",
            storageBucket: "message-5ce86.firebasestorage.app",
            messagingSenderId: "532852021993",
            appId: "1:532852021993:web:4bfce7c8796bd6cdb9ae78"
        };

        const app = firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
        const messagesRef = db.ref("messages");
        let user_name = "";

        const msgBubbleTemplate = `
            <div class="message-item">
                <span class="text-muted sender-name"></span>
                <div class="message-content"></div>
            </div>`;

        function setupChat() {
            let enter_user = prompt("Enter Your Name");
            
            if (enter_user && enter_user.trim() !== "") {
                user_name = enter_user.trim();
            } else {
                alert("Username is required!");
                location.reload();
                return;
            }
            
            $('#userName').text(user_name);

            messagesRef.orderByChild("timestamp").limitToLast(50).on("child_added", (snapshot) => {
                const msgData = snapshot.val();
                if (!msgData) return;
                
                const $bubble = $(msgBubbleTemplate);
                $bubble.addClass(msgData.user === user_name ? "sent" : "received");
                
                if (msgData.user !== user_name) {
                    $bubble.find(".sender-name").text(msgData.user);
                }
                
                $bubble.find(".message-content")
                      .text(msgData.msg)
                      .html(function(i, html) { 
                          return html.replace(/\n/g, "<br>"); 
                      });
                      
                $('#message-items-container').append($bubble);
                
                const container = $('#message-items-container')[0];
                container.scrollTop = container.scrollHeight;
            });

            $('#send-btn').on("click", triggerSend);

            $('#msg-txt-field').on("keypress", function (e) {
                if (e.key === "Enter" && !e.shiftKey) {
                    e.preventDefault();
                    triggerSend();
                }
            });
        }

        function triggerSend() {
            const msg = $("#msg-txt-field").val().trim();
            if (msg === "") return;

            messagesRef.push({
                user: user_name,
                msg: msg,
                timestamp: firebase.database.ServerValue.TIMESTAMP
            });

            $("#msg-txt-field").val("").focus();
        }

        $(document).ready(setupChat);
    </script>
</body>
</html>
