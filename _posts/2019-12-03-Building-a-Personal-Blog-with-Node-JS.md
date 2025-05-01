--- 
title: "Building a Personal Blog with Node.js and Express: A Developer's Journey"

excerpt: "A step-by-step tutorial for creating your own Personal Blog with Node.js, Express, and MongoDB."

categories:
- Node.js
- Backend-Development

tags:
- REST API
- Node.js
- Express
- MongoDB
- API Development
- Backend

--- 
## Introduction

In this article, I'll walk you through my experience creating a personal blog application using Node.js and Express. This project helped me solidify my backend development skills while implementing core web application features.

## Getting Started

### Project Setup

I began by initializing a new Node.js project:

```bash
npm init -y
```

Then installed essential dependencies:

```bash
npm install express mongoose passport ejs express-session multer dotenv bcryptjs
```

Key packages include:
- **Express**: Web framework
- **Mongoose**: MongoDB object modeling
- **Passport**: Authentication middleware
- **EJS**: Templating engine
- **Multer**: File upload handling
- **dotenv**: Environment variable management
- **bcryptjs**: Password hashing

## Application Architecture

### MVC Structure

I organized the project using the Model-View-Controller pattern:

```
project/
├── models/         # Database models
├── views/          # EJS templates
├── controllers/    # Route handlers
├── public/         # Static assets
├── config/         # Configuration files
└── app.js          # Main application file
```

### Entry Point Configuration

The `app.js` file serves as the application's foundation:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const passport = require('passport');
const session = require('express-session');

const app = express();

// Database connection
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Middleware setup
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// View engine configuration
app.set('view engine', 'ejs');

// Server initialization
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Core Features Implementation

### Database Models

I created Mongoose schemas for posts and users:

```javascript
// models/Post.js
const postSchema = new mongoose.Schema({
  title: { type: String, required: true },
  content: { type: String, required: true },
  createdAt: { type: Date, default: Date.now },
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
});

// models/User.js
const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  email: { type: String, required: true, unique: true }
});
```

### Authentication System

Using Passport.js, I implemented local authentication:

```javascript
// config/passport.js
const LocalStrategy = require('passport-local').Strategy;
const User = require('../models/User');
const bcrypt = require('bcryptjs');

passport.use(
  new LocalStrategy(async (username, password, done) => {
    try {
      const user = await User.findOne({ username });
      if (!user) return done(null, false);
      
      const isMatch = await bcrypt.compare(password, user.password);
      if (!isMatch) return done(null, false);
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  })
);
```

### File Uploads with Multer

Configured Multer for handling image uploads:

```javascript
// config/upload.js
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, path.join(__dirname, '../public/uploads'));
  },
  filename: (req, file, cb) => {
    cb(null, `${Date.now()}-${file.originalname}`);
  }
});

const upload = multer({ storage });

module.exports = upload;
```

## Key Learnings

Through this project, I gained valuable experience with:

1. **Express Framework**: Routing, middleware, and request handling
2. **Database Management**: MongoDB schema design and Mongoose operations
3. **Authentication**: Secure user registration and login flows
4. **File Handling**: Uploading and serving static files
5. **Security Practices**: Environment variables and password hashing
6. **Project Structure**: MVC architecture and code organization

## Future Enhancements

Potential improvements include:
- Adding comment functionality
- Implementing search capabilities
- Creating an admin dashboard
- Developing a REST API
- Adding social media authentication
- Improving error handling

## Conclusion

Building this blog application provided hands-on experience with full-stack JavaScript development. The process helped me understand how different components interact in a web application and gave me confidence to tackle more complex projects.