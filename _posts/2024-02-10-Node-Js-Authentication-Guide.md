---
title: "Building An Authenthication System with Node.js and Express - Explained"

excerpt: "An in-depth guide to creating your first Authenthication System using Node.js, Express. Learn step-by-step how to build, secure, and document your API!"

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
---

# Node.js Authentication: JWT, OAuth, and Passport.js  

---

## Introduction to Authentication

**Authentication** and **Authorization** are often confused but serve different purposes:

- **Authentication**: Verifying user identity (e.g., login).
- **Authorization**: Granting access to resources (e.g., admin-only routes).

**Common authentication methods:**

- **Session-Based**: Store user data on the server.
- **Token-Based**: Use tokens like JWT for stateless authentication.
- **OAuth**: Allow third parties (Google, GitHub) to authenticate users.

---

## Prerequisites

✅ Node.js and Express setup  
✅ MongoDB (or another database)  
✅ Google/GitHub developer accounts for OAuth integrations  

---

## Step 1: JWT Authentication (JSON Web Tokens)

**Install necessary packages:**

```bash
npm install jsonwebtoken bcryptjs
```

**Create a User model with password hashing:**

```javascript
const userSchema = new mongoose.Schema({
  email: { type: String, unique: true },
  password: String,
});

// Hash password before saving
userSchema.pre('save', async function (next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 8);
  }
  next();
});
```

**Create `/signup` and `/login` routes:**

```javascript
// Signup
app.post('/signup', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).send(user);
  } catch (err) {
    res.status(400).send(err.message);
  }
});

// Login
app.post('/login', async (req, res) => {
  try {
    const user = await User.findOne({ email: req.body.email });
    if (!user || !await bcrypt.compare(req.body.password, user.password)) {
      throw new Error('Invalid credentials');
    }
    const token = jwt.sign({ userId: user._id }, 'your-secret-key', { expiresIn: '1h' });
    res.send({ token });
  } catch (err) {
    res.status(401).send(err.message);
  }
});
```

---

## Step 2: Protecting Routes with JWT Middleware

**Add a middleware function to verify tokens:**

```javascript
const auth = (req, res, next) => {
  try {
    const token = req.header('Authorization').replace('Bearer ', '');
    const decoded = jwt.verify(token, 'your-secret-key');
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).send('Unauthorized');
  }
};

// Example protected route
app.get('/profile', auth, (req, res) => {
  res.send(`Welcome, user ${req.user.userId}`);
});
```

---

## Step 3: OAuth with Passport.js (Google Example)

**Install Passport.js and the Google strategy:**

```bash
npm install passport passport-google-oauth20
```

**Configure Passport in your app:**

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback',
}, (accessToken, refreshToken, profile, done) => {
  User.findOne({ googleId: profile.id }, async (err, user) => {
    if (!user) {
      user = await User.create({ googleId: profile.id, email: profile.emails[0].value });
    }
    return done(null, user);
  });
}));

passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser((id, done) => User.findById(id, done));
```

**Create OAuth routes:**

```javascript
// Redirect user to Google for authentication
app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));

// Handle Google callback
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/profile');
  }
);
```

---

## Step 4: Session Management (for OAuth)

**Install express-session:**

```bash
npm install express-session
```

**Configure session middleware:**

```javascript
const session = require('express-session');

app.use(session({
  secret: 'your-session-secret',
  resave: false,
  saveUninitialized: false,
}));

app.use(passport.initialize());
app.use(passport.session());
```

---

## Step 5: Adding Social Logins (GitHub and Facebook)

You can extend Passport strategies easily:

- **GitHub**:  
  Install `passport-github2`
  
  ```bash
  npm install passport-github2
  ```

- **Facebook**:  
  Install `passport-facebook`
  
  ```bash
  npm install passport-facebook
  ```

Configuration is similar to the Google setup, just adjust the strategy fields accordingly (client ID, secret, callback URL).

---

## Step 6: Role-Based Access Control (RBAC)

**Add a `role` field to your User model and create authorization middleware:**

```javascript
const authorize = (roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).send('Forbidden');
  }
  next();
};

// Example: Admin-only route
app.get('/admin', auth, authorize(['admin']), (req, res) => {
  res.send('Admin dashboard');
});
```

---

## Conclusion

You’ve now built a Node.js authentication system that includes:

✅ JWT Authentication  
✅ OAuth with Passport.js (Google, GitHub, Facebook)  
✅ Session Management  
✅ Role-Based Access Control  

---

**Next Steps:**

- Implement password reset flows
- Add refresh tokens for longer JWT sessions
- Explore biometric authentication (e.g., Touch ID, Face ID)

---

# 📊 Summary Infographic: Node.js Authentication Flow

Here's the **summary infographic** you asked for, just like the first one:

![Node.js Authentication Infographic](attachment://file_00000000a9b861f78002f62c21d851f6)

---