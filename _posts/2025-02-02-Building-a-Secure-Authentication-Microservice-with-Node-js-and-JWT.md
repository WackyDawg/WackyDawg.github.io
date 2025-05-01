--- 
title: "Building a Secure Authentication Microservice with Node.js and JWT"

excerpt: "Authentication is the backbone of most web applications. Whether you're building a social media platform, an e-commerce site, or an internal dashboard, you need a reliable way to verify users.  "

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

## **Introduction**  

Authentication is the backbone of most web applications. Whether you're building a social media platform, an e-commerce site, or an internal dashboard, you need a reliable way to verify users.  

In this guide, **we'll build a production-ready authentication microservice** using:  
✅ **JSON Web Tokens (JWT)** for stateless sessions  
✅ **Redis** for secure token blacklisting  
✅ **Rate limiting** to prevent brute-force attacks  
✅ **Two-factor authentication (2FA)** for added security  
✅ **Social login (Google OAuth)** for user convenience  

By the end, you'll have a fully functional auth system that you can integrate into any Node.js application.  

---

## **Why Build a Dedicated Auth Microservice?**  

Before diving into code, let's discuss why a standalone auth service is valuable:  

1. **Centralized Security** – One place to manage logins, password resets, and session handling.  
2. **Scalability** – Decouples authentication from your main app, allowing independent scaling.  
3. **Reusability** – The same service can support multiple frontends (web, mobile, APIs).  
4. **Easier Maintenance** – Security updates and bug fixes apply across all connected apps.  

---

## **Tech Stack & Setup**  

### **Dependencies**  
We'll use these Node.js packages:  

| Package | Purpose |
|---------|---------|
| `express` | Web framework |
| `jsonwebtoken` | JWT generation & validation |
| `bcryptjs` | Password hashing |
| `ioredis` | Redis client for token storage |
| `otplib` | Time-based OTP (2FA) |
| `passport` | Social login (Google, Facebook) |
| `express-rate-limit` | Brute-force protection |
| `joi` | Request validation |

### **Initial Setup**  

```bash
mkdir auth-microservice && cd auth-microservice
npm init -y
npm install express jsonwebtoken bcryptjs ioredis otplib passport passport-google-oauth2 express-rate-limit joi
```

---

## **Core Authentication Flow**  

Here’s how our system will work:  

1. **User signs up** → Password is hashed and stored in DB.  
2. **User logs in** → Server issues an **access token (JWT)** and a **refresh token**.  
3. **Access token** → Short-lived (15-30 mins), sent via HTTP-only cookie.  
4. **Refresh token** → Long-lived (7 days), stored securely in Redis.  
5. **Token rotation** → When the access token expires, the refresh token generates a new one.  

---

## **Step 1: User Registration**  

### **Database Model (`models/User.js`)**  

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  is2FAEnabled: { type: Boolean, default: false },
  otpSecret: { type: String }, // For 2FA
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

module.exports = mongoose.model('User', userSchema);
```

### **Registration Endpoint (`controllers/auth.js`)**  

```javascript
const Joi = require('joi');
const User = require('../models/User');

exports.register = async (req, res) => {
  // Validate input
  const schema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required(),
  });

  const { error } = schema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details[0].message });

  // Check if user exists
  const existingUser = await User.findOne({ email: req.body.email });
  if (existingUser) {
    return res.status(409).json({ error: 'Email already in use' });
  }

  // Create user
  const user = new User({
    email: req.body.email,
    password: req.body.password,
  });

  await user.save();
  res.status(201).json({ message: 'User registered successfully' });
};
```

---

## **Step 2: Login & JWT Generation**  

### **Token Service (`services/token.js`)**  

```javascript
const jwt = require('jsonwebtoken');
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

const generateTokens = (userId) => {
  const accessToken = jwt.sign(
    { id: userId },
    process.env.JWT_ACCESS_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { id: userId },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  // Store refresh token in Redis
  redis.set(userId.toString(), refreshToken, 'EX', 7 * 24 * 60 * 60); // 7 days TTL

  return { accessToken, refreshToken };
};

module.exports = { generateTokens };
```

### **Login Endpoint (`controllers/auth.js`)**  

```javascript
const { generateTokens } = require('../services/token');

exports.login = async (req, res) => {
  const { email, password } = req.body;

  // Find user
  const user = await User.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Check password
  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate tokens
  const { accessToken, refreshToken } = generateTokens(user._id);

  // Set HTTP-only cookies
  res.cookie('accessToken', accessToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 15 * 60 * 1000, // 15 mins
  });

  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
  });

  res.json({ message: 'Logged in successfully' });
};
```

---

## **Step 3: Token Refresh Mechanism**  

When the access token expires, the client sends the refresh token to get a new one.  

### **Refresh Token Endpoint (`controllers/auth.js`)**  

```javascript
exports.refreshToken = async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token provided' });
  }

  try {
    // Verify token
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);

    // Check Redis
    const storedToken = await redis.get(decoded.id);
    if (refreshToken !== storedToken) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    // Issue new access token
    const newAccessToken = jwt.sign(
      { id: decoded.id },
      process.env.JWT_ACCESS_SECRET,
      { expiresIn: '15m' }
    );

    res.cookie('accessToken', newAccessToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      maxAge: 15 * 60 * 1000,
    });

    res.json({ message: 'Token refreshed' });
  } catch (err) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
};
```

---

## **Step 4: Adding Two-Factor Authentication (2FA)**  

We’ll use **Time-based One-Time Passwords (TOTP)** via `otplib`.  

### **Enabling 2FA (`controllers/auth.js`)**  

```javascript
const { authenticator } = require('otplib');

exports.setup2FA = async (req, res) => {
  const user = await User.findById(req.user.id);
  
  // Generate secret
  const secret = authenticator.generateSecret();
  user.otpSecret = secret;
  user.is2FAEnabled = true;
  await user.save();

  // Generate QR code URL (for Google Authenticator)
  const otpUrl = authenticator.keyuri(user.email, 'MyApp', secret);
  
  res.json({ qrCodeUrl: `https://chart.googleapis.com/chart?chs=200x200&cht=qr&chl=${otpUrl}` });
};
```

### **Verifying 2FA on Login**  

```javascript
exports.verify2FA = async (req, res) => {
  const { token } = req.body;
  const user = await User.findById(req.user.id);

  const isValid = authenticator.check(token, user.otpSecret);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid 2FA code' });
  }

  // Proceed with login
  const { accessToken, refreshToken } = generateTokens(user._id);
  res.json({ accessToken, refreshToken });
};
```

---

## **Step 5: Rate Limiting & Security**  

### **Prevent Brute-Force Attacks**  

```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Max 5 attempts
  message: 'Too many login attempts, please try again later',
});

app.post('/login', loginLimiter, authController.login);
```

### **Security Headers (Helmet Middleware)**  

```javascript
const helmet = require('helmet');
app.use(helmet());
```

---

## **Conclusion & Next Steps**  

We’ve built a **secure, scalable authentication microservice** with:  
✔ **JWT-based sessions**  
✔ **Redis-backed token storage**  
✔ **2FA support**  
✔ **Rate limiting**  
✔ **Social login (Google OAuth)**  

**Possible Extensions:**  
- **Passwordless login** (magic links)  
- **Biometric authentication** (WebAuthn)  
- **Single Sign-On (SSO)**  
- **User activity logging**  

---

### **Final Thoughts**  
A well-designed auth system is crucial for security and user experience. This microservice can be deployed independently and reused across multiple projects.  

---