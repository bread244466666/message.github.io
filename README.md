<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Firebase Chat App</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <!-- Firebase SDKs -->
    <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Ubuntu+Mono:ital,wght@0,400;0,700;1,400;1,700&display=swap');

        * {
            font-family: "Ubuntu Mono", serif;
            font-weight: 400;
            font-style: normal;
        }

        strong, b {
            font-weight: 700;
        }

        html, body {
            margin: unset;
            padding: unset;
            height: 100%;
            max-height: 100%;
            width: 100%;
            max-width: 100%;
            overflow: auto;
        }

        body {
            display: flex;
            flex-flow: column;
            align-items: center;
            justify-content: center;
            background-image: linear-gradient(to top, #48c6ef 0%, #6f86d6 100%);
        }

        #main-wrapper {
            width: 350px;
        }

        #app-title {
            color: #fff;
            text-shadow: 0px 3px 5px #9a9a9a;
            text-align: center;
            margin-bottom: 50px;
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
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #message-items-container {
            height: 300px;
            overflow: auto;
            display: flex;
            flex-flow: column-reverse;
        }

        #message-items-container:empty {
            align-items: center;
            justify-content: center;
        }

        #message-items-container:empty::before {
            content: "conversation box is currently empty...";
            font-size: 11px;
            font-style: italic;
            align-self: center;
            color: #858585;
        }

        .message-item {
            max-width: 85%;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 15px;
        }

        .message-item.received {
            background-image: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            align-self: start;
        }

        .message-item.sent {
            background-image: linear-gradient(120deg, #89f7fe 0%, #66a6ff 100%);
            align-self: end;
        }

        .message-item .sender-name {
            font-style: italic;
            font-size: 11px;
        }

        .message-item .message-content {
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div id="main-wrapper">
        <h2 id="app-title"><strong>Chat App using HTML, CSS, JS, and Firebase</strong></h2>
        <div id="conversation-box" class="card rounded-0">
            <div class="card-header rounded-0">
                <h4 class="card-title"><strong>Conversation Box</strong></h4>
            </div>
            <div class="card-body rounded-0">
                <div class="container-fluid" id="message-items-container"></div>
            </div>
            <div class="card-footer rounded-0">
                <div>
                    <span><small><i class="text-muted">You are logged in as: <span id="userName"></span></i></small></span>
                </div>
                <div class="d-flex w-100">
                    <div class="flex-grow-1" id="txt-field-container">
                        <textarea id="msg-txt-field" rows="2" class="form-control form-control-sm rounded-0" placeholder="Write your message here..."></textarea>
                    </div>
                    <div id="send-btn-container">
                        <button class="btn btn-primary rounded-0" id="send-btn"><strong>Send</strong></button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script>
        // Replace with your Firebase config
      import { initializeApp } from "firebase/app";
        const firebaseConfig = {
  apiKey: "AIzaSyC4oYXDJb5Lbb4B3nAqX-Hmxf18s13whyA",
  authDomain: "message-5ce86.firebaseapp.com",
  databaseURL: "https://message-5ce86-default-rtdb.firebaseio.com",
  projectId: "message-5ce86",
  storageBucket: "message-5ce86.firebasestorage.app",
  messagingSenderId: "532852021993",
  appId: "1:532852021993:web:4bfce7c8796bd6cdb9ae78"
};

        // Initialize Firebase
        const app = firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // Message bubble template
        let msgBubble = `<div class="message-item">
            <div><span class="text-muted sender-name"></span></div>
            <div class="message-content"></div>
        </div>`;

        let user_name = localStorage.getItem("userName") || "";
        let currentUser = null;

        // Listen for auth state changes
        auth.onAuthStateChanged((user) => {
            if (user) {
                currentUser = user;
                setupChat();
            } else {
                // Sign in anonymously
                auth.signInAnonymously().catch((error) => {
                    console.error("Anonymous auth failed:", error);
                });
            }
        });

        function setupChat() {
            if (user_name === "") {
                var enter_user = prompt("Enter Your Name");
                if (enter_user !== "") {
                    user_name = enter_user;
                    localStorage.setItem("userName", user_name);
                } else {
                    alert("User Name must be provided!");
                    location.reload();
                    return;
                }
            }
            $('#userName').text(user_name);

            // Listen for real-time messages (limit to last 50 for performance)
            db.collection("messages")
                .orderBy("timestamp", "desc")
                .limit(50)
                .onSnapshot((snapshot) => {
                    $('#message-items-container').empty(); // Clear and rebuild to avoid duplicates
                    snapshot.forEach((doc) => {
                        const msgData = doc.data();
                        var bubble = $(msgBubble).clone(true);
                        bubble.addClass(msgData.user === user_name ? "sent" : "received");
                        if (msgData.user !== user_name) {
                            bubble.find(".sender-name").text(`${msgData.user}`);
                        }
                        var content = (msgData.msg).replace(/\n/gi, "<br>");
                        bubble.find(".message-content").html(content);
                        $('#message-items-container').prepend(bubble); // Prepend to maintain order
                    });
                });

            // Send button click event
            $('#send-btn').on("click", triggerSend);

            // Enter key press event in text field
            $('#msg-txt-field').on("keypress", function (e) {
                if (e.key === "Enter" && !e.shiftKey) {
                    e.preventDefault();
                    triggerSend();
                }
            });
        }

        // Function to handle sending messages
        function triggerSend() {
            var msg = $("#msg-txt-field").val().trim();
            if (msg === "") return;

            // Send to Firestore
            db.collection("messages").add({
                user: user_name,
                msg: msg,
                timestamp: firebase.firestore.FieldValue.serverTimestamp()
            }).catch((error) => {
                console.error("Error sending message:", error);
            });

            $("#msg-txt-field").val("");
            $("#msg-txt-field").focus();
        }

        $(document).ready(function () {
            // Auth will trigger setupChat
        });
    </script>
</body>
</html>
