Task Board Backend
A RESTful API backend for the Task Board application, built with Node.js, Express, and MongoDB.
Project Structure
TASKBOARDBACKEND/
├── App/
│   ├── controllers/         # Request handlers for each endpoint
│   ├── middlewares/         # Custom middleware functions
│   ├── models/
│   │   └── Task.js          # Mongoose model for tasks
│   ├── routes/
│   │   ├── taskRoutes.js    # Routes for task operations
│   │   └── index.js         # Main router that combines all routes
│   └── node_modules/        # npm packages
├── .env                     # Environment variables
├── .gitignore               # Git ignore file
├── package-lock.json        # Dependency lock file
└── package.json             # Project metadata and dependencies
Features

RESTful API: Provides endpoints for CRUD operations on tasks
MongoDB Integration: Stores task data in a MongoDB database
Structured Architecture: Follows MVC pattern with separate controllers, models, and routes
Middleware Support: Custom middleware for error handling, validation, etc.
Environment Configuration: Uses dotenv for environment variables management

Tech Stack

Node.js: JavaScript runtime for the server
Express: Web framework for building the API
MongoDB: NoSQL database for data storage
Mongoose: MongoDB object modeling for Node.js
dotenv: Environment variable management

Prerequisites

Node.js (v14 or higher)
MongoDB (local installation or MongoDB Atlas account)
npm or yarn package manager

Installation

Clone the repository:
bashgit clone https://github.com/Aayushgupta2005/TaskBoardBackend.git
cd taskboardbackend

Install dependencies:
bashnpm install

Create a .env file in the root directory with the following variables:
PORT=5000
MONGODB_URI=""
# For production, use your MongoDB Atlas URI
# MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/taskboard

Start the server:
bashnpm start
For development with auto-restart:
bashnpm run dev


API Endpoints
Tasks
MethodEndpointDescriptionRequest BodyResponseGET/api/tasksGet all tasks-Array of task objectsPOST/api/tasksCreate a new task{title, description, status}Newly created task objectPUT/api/tasks/:idUpdate a task{title, description, status}Updated task objectDELETE/api/tasks/:idDelete a task-Success message
Request & Response Examples
Get all tasks
GET /api/tasks
Response:
json{
  "success": true,
  "data": [
    {
      "_id": "60d21b4667d0d8992e610c85",
      "title": "Implement drag and drop",
      "description": "Add drag and drop functionality to the task board",
      "status": "inprogress",
      "createdAt": "2023-05-15T10:30:40.000Z"
    },
    {
      "_id": "60d21b4967d0d8992e610c86",
      "title": "Create API documentation",
      "description": "Document all API endpoints",
      "status": "todo",
      "createdAt": "2023-05-15T10:35:20.000Z"
    }
  ]
}
Create a new task
POST /api/tasks
Request body:
json{
  "title": "Deploy application",
  "description": "Deploy the application to production",
  "status": "todo"
}
Response:
json{
  "success": true,
  "data": {
    "_id": "60d21b5367d0d8992e610c87",
    "title": "Deploy application",
    "description": "Deploy the application to production",
    "status": "todo",
    "createdAt": "2023-05-15T10:40:15.000Z"
  }
}
Update a task
PUT /api/tasks/60d21b5367d0d8992e610c87
Request body:
json{
  "status": "inprogress"
}
Response:
json{
  "success": true,
  "data": {
    "_id": "60d21b5367d0d8992e610c87",
    "title": "Deploy application",
    "description": "Deploy the application to production",
    "status": "inprogress",
    "createdAt": "2023-05-15T10:40:15.000Z"
  }
}
Delete a task
DELETE /api/tasks/60d21b5367d0d8992e610c87
Response:
json{
  "success": true,
  "data": {}
}
Implementation Details
Models (App/models/Task.js)
The Task model defines the schema for task documents in the MongoDB database:
javascriptconst mongoose = require('mongoose');

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    trim: true
  },
  description: {
    type: String,
    trim: true
  },
  status: {
    type: String,
    enum: ['todo', 'inprogress', 'done'],
    default: 'todo'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Task', taskSchema);
Routes (App/routes/taskRoutes.js)
The task routes define the API endpoints for task operations:
javascriptconst express = require('express');
const router = express.Router();
const taskController = require('../controllers/taskController');

router.get('/', taskController.getAllTasks);
router.post('/', taskController.createTask);
router.put('/:id', taskController.updateTask);
router.delete('/:id', taskController.deleteTask);

module.exports = router;
Controllers (App/controllers/taskController.js)
The task controller contains the logic for handling task operations:
javascriptconst Task = require('../models/Task');

// Get all tasks
exports.getAllTasks = async (req, res) => {
  try {
    const tasks = await Task.find().sort({ createdAt: -1 });
    res.json({ success: true, data: tasks });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

// Create a new task
exports.createTask = async (req, res) => {
  try {
    const task = await Task.create(req.body);
    res.status(201).json({ success: true, data: task });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

// Update a task
exports.updateTask = async (req, res) => {
  try {
    const task = await Task.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
      runValidators: true
    });
    
    if (!task) {
      return res.status(404).json({ success: false, error: 'Task not found' });
    }
    
    res.json({ success: true, data: task });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

// Delete a task
exports.deleteTask = async (req, res) => {
  try {
    const task = await Task.findByIdAndDelete(req.params.id);
    
    if (!task) {
      return res.status(404).json({ success: false, error: 'Task not found' });
    }
    
    res.json({ success: true, data: {} });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};
Middleware (App/middlewares/)
This directory contains custom middleware functions for:

Request validation
Error handling
Authentication (if implemented)
Logging

Connecting with the Frontend
To connect this API with the Task Board frontend:

Ensure the frontend's API service points to your backend URL:
javascript// In the frontend's api.js file
const API_URL = 'http://localhost:5000/api';

The frontend should call these endpoints for task operations:

GET /api/tasks to fetch all tasks
POST /api/tasks to create a new task
PUT /api/tasks/:id to update a task (including drag-and-drop status changes)
DELETE /api/tasks/:id to delete a task



Environment Variables
The following environment variables should be defined in your .env file:

PORT: The port number for the server (default: 5000)
MONGODB_URI: MongoDB connection string
NODE_ENV: Environment mode (development, production, test)

Scripts
Add these scripts to your package.json:
json"scripts": {
  "start": "node App/index.js",
  "dev": "nodemon App/index.js"
}