# MERN-Freelancer-Marketplace
Creating a beautiful Freelancer Marketplace app with the MERN stack (MongoDB, Express.js, React.js, Node.js) involves several key steps. The app will include features like user registration, job posting, bidding, messaging, and a payment gateway (e.g., PayPal or Stripe). I'll guide you through creating a basic version of this app and provide code examples to get you started.
Steps for Building the Freelancer Marketplace App
1. Set up the MERN stack environment

You'll need to set up the backend and frontend in separate directories and use MongoDB for data storage. Follow the steps below.

    Install Node.js & MongoDB:
        Make sure you have Node.js and MongoDB installed on your local system.

    Create the Backend (Node.js + Express + MongoDB):

# Backend Setup (API server)
mkdir freelancer-backend
cd freelancer-backend
npm init -y
npm install express mongoose dotenv bcryptjs jsonwebtoken cors

Create a file structure like:

freelancer-backend/
  ├── models/
  ├── routes/
  ├── controllers/
  ├── .env
  └── server.js

server.js (Backend entry point)

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');
dotenv.config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB Connection
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.log(err));

// Routes
const userRoutes = require('./routes/userRoutes');
const jobRoutes = require('./routes/jobRoutes');
app.use('/api/users', userRoutes);
app.use('/api/jobs', jobRoutes);

// Starting server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

.env (Environment variables for MongoDB connection)

MONGO_URI=your_mongo_connection_string
JWT_SECRET=your_jwt_secret

models/User.js (User Schema)

const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  name: String,
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['freelancer', 'employer'], default: 'freelancer' },
}, { timestamps: true });

module.exports = mongoose.model('User', UserSchema);

routes/userRoutes.js (User Routes)

const express = require('express');
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const router = express.Router();

// Register
router.post('/register', async (req, res) => {
  try {
    const { name, email, password, role } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const newUser = new User({ name, email, password: hashedPassword, role });
    await newUser.save();
    const token = jwt.sign({ userId: newUser._id }, process.env.JWT_SECRET);
    res.json({ token });
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(400).json({ message: 'User not found' });
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ message: 'Invalid credentials' });
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET);
    res.json({ token });
  } catch (err) {
    res.status(500).send(err.message);
  }
});

module.exports = router;

models/Job.js (Job Schema)

const mongoose = require('mongoose');

const JobSchema = new mongoose.Schema({
  title: { type: String, required: true },
  description: { type: String, required: true },
  employer: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  bids: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
}, { timestamps: true });

module.exports = mongoose.model('Job', JobSchema);

routes/jobRoutes.js (Job Routes)

const express = require('express');
const Job = require('../models/Job');
const router = express.Router();

// Create Job
router.post('/', async (req, res) => {
  try {
    const { title, description, employerId } = req.body;
    const newJob = new Job({ title, description, employer: employerId });
    await newJob.save();
    res.json(newJob);
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Get Jobs
router.get('/', async (req, res) => {
  try {
    const jobs = await Job.find().populate('employer', 'name');
    res.json(jobs);
  } catch (err) {
    res.status(500).send(err.message);
  }
});

module.exports = router;

2. Set up the Frontend (React)

Now, we’ll set up the React application for the frontend.

# Frontend Setup (React)
npx create-react-app freelancer-frontend
cd freelancer-frontend
npm install axios react-router-dom

Create a file structure for the React app like:

freelancer-frontend/
  ├── components/
  ├── pages/
  ├── App.js
  ├── axios.js
  └── index.js

src/axios.js (Axios instance for API requests)

import axios from 'axios';

const axiosInstance = axios.create({
  baseURL: 'http://localhost:5000/api', // API server URL
});

export default axiosInstance;

src/App.js (React App Structure)

import React, { useState, useEffect } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import axios from './axios';
import JobList from './components/JobList';
import JobPost from './components/JobPost';
import UserLogin from './components/UserLogin';
import UserRegister from './components/UserRegister';

const App = () => {
  const [jobs, setJobs] = useState([]);

  useEffect(() => {
    axios.get('/jobs')
      .then(response => setJobs(response.data))
      .catch(err => console.error(err));
  }, []);

  return (
    <Router>
      <div>
        <Switch>
          <Route path="/" exact>
            <JobList jobs={jobs} />
          </Route>
          <Route path="/post-job" exact>
            <JobPost />
          </Route>
          <Route path="/login" exact>
            <UserLogin />
          </Route>
          <Route path="/register" exact>
            <UserRegister />
          </Route>
        </Switch>
      </div>
    </Router>
  );
};

export default App;

components/JobList.js (Display Jobs)

import React from 'react';

const JobList = ({ jobs }) => {
  return (
    <div>
      <h1>Available Jobs</h1>
      <ul>
        {jobs.map(job => (
          <li key={job._id}>
            <h3>{job.title}</h3>
            <p>{job.description}</p>
            <p>Posted by: {job.employer.name}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default JobList;

components/JobPost.js (Post Job Form)

import React, { useState } from 'react';
import axios from '../axios';

const JobPost = () => {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await axios.post('/jobs', { title, description });
      alert('Job posted successfully!');
    } catch (err) {
      alert('Error posting job');
    }
  };

  return (
    <div>
      <h1>Post a Job</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Job Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
        />
        <textarea
          placeholder="Job Description"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
        ></textarea>
        <button type="submit">Post Job</button>
      </form>
    </div>
  );
};

export default JobPost;

3. Run the App

    Start the backend server:

cd freelancer-backend
node server.js

    Start the React frontend:

cd freelancer-frontend
npm start

Now, open your browser and go to http://localhost:3000 to view the application. You should be able to post jobs, view jobs, and implement further features like bidding, messaging, and payments.
Next Steps:

    Authentication & Authorization: Implement JWT-based authentication for user registration, login, and authorization of job posting/bidding.
    Job Bidding: Add bidding functionality where freelancers can bid on jobs.
    Messaging: Implement a chat system for communication between employers and freelancers.
    Payment Integration: Use Stripe or PayPal for secure payments between freelancers and employers.

By following these steps, you’ll have the foundation of a beautiful freelancer marketplace app on the MERN stack.
