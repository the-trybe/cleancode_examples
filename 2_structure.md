# Code Structure Best Practices

> "The structure of your code today determines the maintainability of your system tomorrow."

## Table of Contents
1. [Modular Design](#1-modular-design)
2. [Object-Oriented Programming](#2-object-oriented-programming)
3. [Function Decomposition](#3-function-decomposition)
4. [File Organization](#4-file-organization)

---

## 1. Modular Design
### 💡 Principle
Break code into smaller, reusable modules with clear responsibilities. Each module should be independent and interchangeable.

### ✨ Key Points
- Single Responsibility Principle
- High Cohesion, Low Coupling
- Clear Module Interfaces
- Dependency Management

### 📝 Examples

#### Node.js
❌ **Poor: Everything in one file**
```javascript
// app.js - Everything mixed together
const express = require('express');
const app = express();
const mongoose = require('mongoose');

// Database connection
mongoose.connect('mongodb://localhost/myapp');

// User model
const userSchema = new mongoose.Schema({
  name: String,
  email: String
});
const User = mongoose.model('User', userSchema);

// Routes
app.post('/register', async (req, res) => {
  const user = new User(req.body);
  await user.save();
  res.send(user);
});

app.listen(3000);
```

✅ **Clean: Modular Structure**
```javascript
// config/database.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log('Database connected successfully');
  } catch (error) {
    console.error('Database connection failed:', error.message);
    process.exit(1);
  }
};

module.exports = connectDB;

// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: String
}, { timestamps: true });

module.exports = mongoose.model('User', userSchema);

// controllers/userController.js
const User = require('../models/User');

exports.register = async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ success: true, data: user });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.post('/register', userController.register);

module.exports = router;

// app.js
const express = require('express');
const connectDB = require('./config/database');
const userRoutes = require('./routes/userRoutes');

const app = express();
connectDB();

app.use('/api/users', userRoutes);
app.listen(3000);
```

---

## 2. Object-Oriented Programming
### 💡 Principle
Use classes and objects to create maintainable, reusable, and extensible code structures.

### ✨ Key Points
- Encapsulation
- Inheritance
- Polymorphism
- Interface Segregation

### 📝 Examples

#### Node.js
❌ **Poor: No Structure**
```javascript
const processPayment = (type, amount) => {
  if (type === 'credit') {
    // Process credit payment
  } else if (type === 'paypal') {
    // Process PayPal payment
  }
};
```

✅ **Clean: OOP Structure**
```javascript
// Abstract base class
class PaymentProcessor {
  constructor(amount) {
    this.amount = amount;
  }

  async process() {
    throw new Error('Process method must be implemented');
  }

  validate() {
    if (this.amount <= 0) {
      throw new Error('Invalid amount');
    }
  }
}

class CreditCardProcessor extends PaymentProcessor {
  constructor(amount, cardDetails) {
    super(amount);
    this.cardDetails = cardDetails;
  }

  async process() {
    this.validate();
    // Process credit card payment
    return { success: true, method: 'credit' };
  }
}

class PayPalProcessor extends PaymentProcessor {
  constructor(amount, paypalEmail) {
    super(amount);
    this.paypalEmail = paypalEmail;
  }

  async process() {
    this.validate();
    // Process PayPal payment
    return { success: true, method: 'paypal' };
  }
}

// Usage
const processPayment = async (processor) => {
  try {
    return await processor.process();
  } catch (error) {
    console.error('Payment failed:', error.message);
    return { success: false, error: error.message };
  }
};
```

---

## 3. Function Decomposition
### 💡 Principle
Break down complex operations into smaller, focused functions that each do one thing well.

### ✨ Key Points
- Single Responsibility
- Maximum 20-30 Lines per Function
- Clear Input/Output
- Descriptive Names

### 📝 Examples

#### Node.js
❌ **Poor: Monolithic Function**
```javascript
async function handleUserRegistration(data) {
  // Validate input
  if (!data.email || !data.password || !data.name) {
    throw new Error('Missing required fields');
  }
  if (data.password.length < 8) {
    throw new Error('Password too short');
  }
  if (!data.email.includes('@')) {
    throw new Error('Invalid email');
  }

  // Hash password
  const salt = await bcrypt.genSalt(10);
  const hashedPassword = await bcrypt.hash(data.password, salt);

  // Create user
  const user = new User({
    email: data.email,
    password: hashedPassword,
    name: data.name
  });

  // Save user
  await user.save();

  // Send welcome email
  const msg = {
    to: data.email,
    subject: 'Welcome!',
    text: `Welcome ${data.name}!`
  };
  await sendEmail(msg);

  return user;
}
```

✅ **Clean: Decomposed Functions**
```javascript
class UserRegistrationService {
  validateInput(data) {
    if (!data.email || !data.password || !data.name) {
      throw new Error('Missing required fields');
    }
    this.validatePassword(data.password);
    this.validateEmail(data.email);
  }

  validatePassword(password) {
    if (password.length < 8) {
      throw new Error('Password too short');
    }
  }

  validateEmail(email) {
    if (!email.includes('@')) {
      throw new Error('Invalid email');
    }
  }

  async hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return bcrypt.hash(password, salt);
  }

  async createUser(data, hashedPassword) {
    const user = new User({
      email: data.email,
      password: hashedPassword,
      name: data.name
    });
    return user.save();
  }

  async sendWelcomeEmail(email, name) {
    const msg = {
      to: email,
      subject: 'Welcome!',
      text: `Welcome ${name}!`
    };
    return sendEmail(msg);
  }

  async register(data) {
    try {
      this.validateInput(data);
      const hashedPassword = await this.hashPassword(data.password);
      const user = await this.createUser(data, hashedPassword);
      await this.sendWelcomeEmail(data.email, data.name);
      return user;
    } catch (error) {
      throw new Error(`Registration failed: ${error.message}`);
    }
  }
}
```

---

## 4. File Organization
### 💡 Principle
Organize files in a logical, consistent structure that makes it easy to locate and maintain code.

### ✨ Key Points
- Group Related Files
- Consistent Naming
- Clear Directory Structure
- Separation by Feature/Module

### 📝 Example Project Structure

```
project-root/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── app.js
│   ├── models/
│   │   ├── User.js
│   │   └── Product.js
│   ├── controllers/
│   │   ├── userController.js
│   │   └── productController.js
│   ├── services/
│   │   ├── userService.js
│   │   └── emailService.js
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── productRoutes.js
│   ├── middleware/
│   │   ├── auth.js
│   │   └── validation.js
│   └── utils/
│       ├── logger.js
│       └── helpers.js
├── tests/
│   ├── unit/
│   └── integration/
├── public/
│   ├── css/
│   └── js/
└── package.json
```

### 🔑 Best Practices
1. **Group by Feature**: Keep related files together
2. **Clear Naming**: Use consistent, descriptive names
3. **Shallow Nesting**: Avoid deep directory structures
4. **Logical Separation**: Separate concerns into appropriate directories

## 📚 Additional Resources
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Laravel Architecture Patterns](https://github.com/alexeymezenin/laravel-best-practices)
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
