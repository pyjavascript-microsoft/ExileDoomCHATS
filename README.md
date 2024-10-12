# ExileDoomCHATS
ask questions, Chat, have fun
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ExileDoom</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f0f0f0;
            color: #333;
        }
        h1 {
            text-align: center;
            color: #007bff;
            margin-bottom: 30px;
        }
        h2 {
            margin-top: 30px;
            border-bottom: 2px solid #007bff;
            padding-bottom: 10px;
        }
        .container {
            display: grid;
            grid-template-columns: 1fr;
            gap: 20px;
        }
        .question, .dm-message, .group-message {
            margin: 20px 0;
            padding: 15px;
            border: 1px solid #ccc;
            border-radius: 5px;
            background: white;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }
        button {
            background-color: #007bff;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #0056b3;
        }
        .hidden {
            display: none;
        }
        .notification {
            color: green;
            font-weight: bold;
        }
    </style>
</head>
<body>

<h1>ExileDoom</h1>
<div id="notifications"></div>

<!-- User Registration -->
<h2>Register</h2>
<input type="email" id="register-email" placeholder="Email" />
<input type="password" id="register-password" placeholder="Password" />
<button onclick="register()">Register</button>
<div id="register-message" class="notification"></div>

<!-- User Login -->
<h2>Login</h2>
<input type="email" id="login-email" placeholder="Email" />
<input type="password" id="login-password" placeholder="Password" />
<button onclick="login()">Login</button>
<div id="login-message" class="notification"></div>

<button id="logout-button" class="hidden" onclick="logout()">Logout</button>

<!-- Direct Messaging -->
<h2>Direct Messages</h2>
<input type="text" id="dm-recipient" placeholder="Recipient Username" />
<textarea id="dm-input" placeholder="Your Message"></textarea>
<button onclick="sendDM()">Send DM</button>
<div id="dm-messages" class="container"></div>

<!-- Group Functionality -->
<h2>Create Group</h2>
<input type="text" id="group-name" placeholder="Group Name" />
<input type="text" id="group-members" placeholder="Members (comma-separated)" />
<button onclick="createGroup()">Create Group</button>
<div id="group-message" class="notification"></div>

<h2>Groups</h2>
<div id="groups-container" class="container"></div>

<!-- Question Form -->
<h2>Ask a Question</h2>
<input type="text" id="question" placeholder="Question Title" />
<textarea id="questionBody" placeholder="Your Question"></textarea>
<input type="text" id="tags" placeholder="Tags (comma separated)" />
<button onclick="submitQuestion()">Submit Question</button>
<div id="question-error" class="notification"></div>

<!-- Questions List -->
<h2>Questions</h2>
<div id="posts-container" class="container"></div>

<script>
    let currentUser = null;
    const users = JSON.parse(localStorage.getItem('users')) || [];
    const questions = JSON.parse(localStorage.getItem('questions')) || [];
    const directMessages = JSON.parse(localStorage.getItem('directMessages')) || {};
    const groups = JSON.parse(localStorage.getItem('groups')) || {};

    function register() {
        const email = document.getElementById('register-email').value;
        const password = document.getElementById('register-password').value;

        const existingUser = users.find(user => user.email === email);
        if (existingUser) {
            document.getElementById('register-message').innerText = 'Email already registered.';
            return;
        }

        users.push({ email, password });
        localStorage.setItem('users', JSON.stringify(users));
        document.getElementById('register-message').innerText = 'User registered successfully!';
    }

    function login() {
        const email = document.getElementById('login-email').value;
        const password = document.getElementById('login-password').value;

        const user = users.find(user => user.email === email && user.password === password);
        if (user) {
            currentUser = user;
            document.getElementById('login-message').innerText = 'User logged in successfully!';
            document.getElementById('logout-button').classList.remove('hidden');
            displayQuestions(); // Refresh questions after login
            displayDMs(); // Refresh direct messages after login
            displayGroups(); // Refresh groups after login
        } else {
            document.getElementById('login-message').innerText = 'Invalid email or password.';
        }
    }

    function logout() {
        currentUser = null;
        document.getElementById('logout-button').classList.add('hidden');
        document.getElementById('login-message').innerText = 'User logged out.';
    }

    function sendDM() {
        const recipient = document.getElementById('dm-recipient').value;
        const message = document.getElementById('dm-input').value;

        if (!currentUser) {
            alert('You must be logged in to send a message.');
            return;
        }

        if (!directMessages[recipient]) {
            directMessages[recipient] = [];
        }
        directMessages[recipient].push({ from: currentUser.email, message });
        localStorage.setItem('directMessages', JSON.stringify(directMessages));
        document.getElementById('dm-recipient').value = '';
        document.getElementById('dm-input').value = '';
        displayDMs();
    }

    function displayDMs() {
        const dmMessagesContainer = document.getElementById('dm-messages');
        dmMessagesContainer.innerHTML = '';

        if (!currentUser) return;

        Object.keys(directMessages).forEach((recipient) => {
            const messages = directMessages[recipient];
            const recipientDiv = document.createElement('div');
            recipientDiv.classList.add('dm-message');
            recipientDiv.innerHTML = `<h4>Messages with ${recipient}</h4>`;
            messages.forEach(msg => {
                recipientDiv.innerHTML += `<p><strong>${msg.from}:</strong> ${msg.message}</p>`;
            });
            dmMessagesContainer.appendChild(recipientDiv);
        });
    }

    function createGroup() {
        const groupName = document.getElementById('group-name').value;
        const groupMembers = document.getElementById('group-members').value.split(',').map(member => member.trim());

        if (!currentUser) {
            document.getElementById('group-message').innerText = 'You must be logged in to create a group.';
            return;
        }

        if (!groupName) {
            document.getElementById('group-message').innerText = 'Please enter a group name.';
            return;
        }

        groups[groupName] = { members: groupMembers, messages: [] };
        localStorage.setItem('groups', JSON.stringify(groups));
        document.getElementById('group-message').innerText = 'Group created successfully!';
        displayGroups();
    }

    function displayGroups() {
        const groupsContainer = document.getElementById('groups-container');
        groupsContainer.innerHTML = '';

        Object.keys(groups).forEach(groupName => {
            const groupDiv = document.createElement('div');
            groupDiv.classList.add('group-message');
            groupDiv.innerHTML = `<h4>${groupName}</h4>`;
            const messages = groups[groupName].messages;

            messages.forEach(msg => {
                groupDiv.innerHTML += `<p><strong>${msg.from}:</strong> ${msg.message}</p>`;
            });

            const messageInput = document.createElement('textarea');
            messageInput.placeholder = 'Send a message to this group...';
            const sendButton = document.createElement('button');
            sendButton.innerText = 'Send';
            sendButton.onclick = () => sendGroupMessage(groupName, messageInput.value);

            groupDiv.appendChild(messageInput);
            groupDiv.appendChild(sendButton);
            groupsContainer.appendChild(groupDiv);
        });
    }

    function sendGroupMessage(groupName, message) {
        if (!currentUser) {
            alert('You must be logged in to send a group message.');
            return;
        }

        if (message) {
            groups[groupName].messages.push({ from: currentUser.email, message });
            localStorage.setItem('groups', JSON.stringify(groups));
            displayGroups(); // Refresh group messages
        }
    }

    function submitQuestion() {
        const title = document.getElementById('question').value;
        const body = document.getElementById('questionBody').value;
        const tags = document.getElementById('tags').value.split(',').map(tag => tag.trim());

        if (!currentUser) {
            document.getElementById('question-error').innerText = 'You must be logged in to ask a question.';
            return;
        }

        if (title && body) {
            const question = { title, body, tags, votes: 0, comments: [] };
            questions.push(question);
            localStorage.setItem('questions', JSON.stringify(questions));
            displayQuestions();
            document.getElementById('question').value = '';
            document.getElementById('questionBody').value = '';
            document.getElementById('tags').value = '';
        } else {
            document.getElementById('question-error').innerText = 'Please enter a title and body.';
        }
    }

    function displayQuestions() {
        const postsContainer = document.getElementById('posts-container');
        postsContainer.innerHTML = '';

        questions.forEach((q, index) => {
            const questionDiv = document.createElement('div');
            questionDiv.classList.add('question');
            questionDiv.innerHTML = `
                <h3>${q.title}</h3>
                <p>${q.body}</p>
                <p>Tags: ${q.tags.map(tag => `<span class="tag">${tag}</span>`).join(' ')}</p>
                <p>
                    <span class="upvote" onclick="voteQuestion(${index}, 1)">↑</span>
                    <span>${q.votes}</span>
                    <span class="downvote" onclick="voteQuestion(${index}, -1)">↓</span>
                </p>
                <div>
                    <h4>Comments</h4>
                    <textarea id="commentBody-${index}" placeholder="Add a comment..."></textarea>
                    <button onclick="submitComment(${index})">Comment</button>
                    <div id="comments-${index}"></div>
                </div>
            `;
            postsContainer.appendChild(questionDiv);
        });
    }

    function voteQuestion(index, value) {
        questions[index].votes += value;
        localStorage.setItem('questions', JSON.stringify(questions));
        displayQuestions();
    }

    function submitComment(questionIndex) {
        const commentBody = document.getElementById(`commentBody-${questionIndex}`).value;
        if (commentBody) {
            questions[questionIndex].comments.push({ body: commentBody, user: currentUser.email });
            localStorage.setItem('questions', JSON.stringify(questions));
            document.getElementById(`commentBody-${questionIndex}`).value = '';
            displayQuestions();
        }
    }

    // Initial load
    displayQuestions();
    displayDMs();
    displayGroups();
</script>

</body>
</html>
