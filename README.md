# To-Do-List
to-do-list/
 /todo-app
/public
/css
styles.css
/js
app.js
/views
index.html
/routes
tasks.js
package.json
server.js
README.md
# Node.js
mkdir to-do-list
cd to-do-list
npm init -y
npm install express body-parser
# server.js(Java Script)

const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3000;

let tasks = [];

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

app.get('/api/tasks', (req, res) => {
    res.json(tasks);
});

app.post('/api/tasks', (req, res) => {
    const newTask = req.body;
    tasks.push({ id: Date.now(), ...newTask, completed: false });
    res.status(201).json(newTask);
});

app.put('/api/tasks/:id', (req, res) => {
    const taskId = parseInt(req.params.id);
    const taskIndex = tasks.findIndex(task => task.id === taskId);
    if (taskIndex !== -1) {
        tasks[taskIndex] = { ...tasks[taskIndex], ...req.body };
        res.json(tasks[taskIndex]);
    } else {
        res.status(404).send('Task not found');
    }
});

app.delete('/api/tasks/:id', (req, res) => {
    const taskId = parseInt(req.params.id);
    tasks = tasks.filter(task => task.id !== taskId);
    res.status(204).send();
});

app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
# html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css">
    <title>To-Do List</title>
</head>
<body>
    <div class="container">
        <h1>To-Do List</h1>
        <div class="task-input">
            <input type="text" id="new-task" placeholder="Add a new task...">
            <button id="add-task">Add Task</button>
        </div>
        <ul id="task-list"></ul>
    </div>

    <script src="script.js"></script>
</body>
</html>

# CSS
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.container {
    background: #fff;
    padding: 20px;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    width: 300px;
}

h1 {
    text-align: center;
    margin-bottom: 20px;
}

.task-input {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
}

input[type="text"] {
    flex: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-right: 10px;
}

button {
    padding: 10px;
    background-color: #28a745;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background-color: #218838;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    border-bottom: 1px solid #ddd;
}

.completed {
    text-decoration: line-through;
    color: #888;
}

.edit-button, .delete-button, .complete-button {
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-left: 5px;
}

.delete-button {
    background-color: #dc3545;
}

.complete-button {
    background-color: #28a745;
}

.edit-button:hover {
    background-color: #0056b3;
}

.delete-button:hover {
    background-color: #c82333;
}

.complete-button:hover {
    background-color: #218838;
}

# Java Script

document.addEventListener('DOMContentLoaded', () => {
    const taskInput = document.getElementById('new-task');
    const addTaskButton = document.getElementById('add-task');
    const taskList = document.getElementById('task-list');

    // Load tasks from server
    fetchTasks();

    addTaskButton.addEventListener('click', addTask);

    async function fetchTasks() {
        const response = await fetch('/api/tasks');
        const tasks = await response.json();
        taskList.innerHTML = '';
        tasks.forEach(task => addTaskToDOM(task));
    }

    async function addTask() {
        const taskText = taskInput.value.trim();
        if (taskText === '') return;

        const newTask = { text: taskText };

        const response = await fetch('/api/tasks', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(newTask)
        });

        const addedTask = await response.json();
        addTaskToDOM(addedTask);
        taskInput.value = '';
    }

    function addTaskToDOM(task) {
        const li = document.createElement('li');
        li.setAttribute('data-id', task.id);
        li.innerHTML = `
            <span class="${task.completed ? 'completed' : ''}">${task.text}</span>
            <div>
                <button class="complete-button">Complete</button>
                <button class="edit-button">Edit</button>
                <button class="delete-button">Delete</button>
            </div>
        `;
        taskList.appendChild(li);

        const completeButton = li.querySelector('.complete-button');
        const editButton = li.querySelector('.edit-button');
        const deleteButton = li.querySelector('.delete-button');

        completeButton.addEventListener('click', () => toggleComplete(task.id));
        editButton.addEventListener('click', () => editTask(task.id));
        deleteButton.addEventListener('click', () => deleteTask(task.id));
    }

    async function toggleComplete(taskId) {
        const taskElement = document.querySelector(`[data-id='${taskId}'] span`);
        const isCompleted = !taskElement.classList.contains('completed');

        const response = await fetch(`/api/tasks/${taskId}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ completed: isCompleted })
        });

        if (response.ok) {
            taskElement.classList.toggle('completed');
        }
    }

    function editTask(taskId) {
        const taskElement = document.querySelector(`[data-id='${taskId}'] span`);
        const newText = prompt('Edit your task:', taskElement.textContent);
        if (newText && newText.trim() !== '') {
            updateTask(taskId, newText.trim());
        }
    }

    async function updateTask(taskId, newText) {
        const response = await fetch(`/api/tasks/${taskId}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ text: newText })
        });

        if (response.ok) {
            const taskElement = document.querySelector(`[data-id='${taskId}'] span`);
            taskElement.textContent = newText;
        }
    }

    async function deleteTask(taskId) {
        const response = await fetch(`/api/tasks/${taskId}`, {
            method: 'DELETE'
        });

        if (response.ok) {
            const taskElement = document.querySelector(`[data-id='${taskId}']`);
            taskElement.remove();
        }
    }
});

