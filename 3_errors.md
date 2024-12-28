# Error Handling Best Practices

> "Error handling is not an afterthought—it's an integral part of software design."

## Table of Contents
1. [Input Validation](#1-input-validation)
2. [Error Handling Strategies](#2-error-handling-strategies)
3. [Logging and Monitoring](#3-logging-and-monitoring)
4. [Custom Error Types](#4-custom-error-types)

---

## 1. Input Validation
### 💡 Principle
Validate all input data at the system boundaries to prevent invalid data from propagating through your application.

### ✨ Key Points
- Validate Early
- Use Type Checking
- Provide Clear Error Messages
- Sanitize User Input

### 📝 Examples

#### Node.js
❌ **Poor: Minimal Validation**
```javascript
app.post('/users', (req, res) => {
  const { name, email, age } = req.body;

  if (!name || !email) {
    return res.status(400).send('Name and email are required');
  }

  // Proceed with saving user to database
});
```

✅ **Clean: Comprehensive Validation**
```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  name: Joi.string()
    .min(3)
    .max(50)
    .required()
    .messages({
      'string.min': 'Name must be at least 3 characters long',
      'string.max': 'Name cannot exceed 50 characters',
      'any.required': 'Name is required'
    }),
  email: Joi.string()
    .email()
    .required()
    .messages({
      'string.email': 'Please provide a valid email address',
      'any.required': 'Email is required'
    }),
  age: Joi.number()
    .integer()
    .min(0)
    .max(120)
    .optional()
    .messages({
      'number.min': 'Age cannot be negative',
      'number.max': 'Age cannot exceed 120 years'
    })
});

const validateUser = async (req, res, next) => {
  try {
    const validated = await userSchema.validateAsync(req.body);
    req.validatedData = validated;
    next();
  } catch (error) {
    res.status(400).json({
      status: 'error',
      message: error.details[0].message
    });
  }
};

app.post('/users', validateUser, async (req, res) => {
  try {
    const user = await User.create(req.validatedData);
    res.status(201).json({
      status: 'success',
      data: { user }
    });
  } catch (error) {
    next(error);
  }
});
```

#### Laravel
❌ **Poor: Inline Validation**
```php
public function store(Request $request) {
    if (!$request->has('name') || !$request->has('email')) {
        return response()->json(['error' => 'Name and email are required'], 400);
    }

    // Proceed with saving user to database
}
```

✅ **Clean: Form Request Validation**
```php
// app/Http/Requests/StoreUserRequest.php
class StoreUserRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => ['required', 'string', 'min:3', 'max:50'],
            'email' => ['required', 'email', 'unique:users'],
            'age' => ['nullable', 'integer', 'min:0', 'max:120'],
        ];
    }

    public function messages()
    {
        return [
            'name.required' => 'A name is required',
            'name.min' => 'Name must be at least 3 characters',
            'email.required' => 'An email is required',
            'email.email' => 'Please provide a valid email address',
            'email.unique' => 'This email is already registered',
            'age.integer' => 'Age must be a whole number',
            'age.min' => 'Age cannot be negative',
            'age.max' => 'Age cannot exceed 120 years'
        ];
    }
}

// app/Http/Controllers/UserController.php
public function store(StoreUserRequest $request)
{
    $user = User::create($request->validated());
    
    return response()->json([
        'status' => 'success',
        'message' => 'User created successfully',
        'data' => UserResource::make($user)
    ], 201);
}
```

---

## 2. Error Handling Strategies
### 💡 Principle
Implement consistent error handling patterns throughout your application to ensure reliability and maintainability.

### ✨ Key Points
- Use Try-Catch Blocks
- Create Custom Error Types
- Handle Async Errors
- Implement Global Error Handlers

### 📝 Examples

#### Node.js
❌ **Poor: Inconsistent Error Handling**
```javascript
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    res.status(404).send('Not found');
    return;
  }
  res.json(user);
});
```

✅ **Clean: Structured Error Handling**
```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    throw new AppError('User not found', 404);
  }
  
  res.json({
    status: 'success',
    data: { user }
  });
}));

// Global error handler
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack
    });
  } else {
    // Production
    if (err.isOperational) {
      res.status(err.statusCode).json({
        status: err.status,
        message: err.message
      });
    } else {
      // Programming or unknown error
      console.error('ERROR 💥', err);
      res.status(500).json({
        status: 'error',
        message: 'Something went wrong'
      });
    }
  }
});
```

---

## 3. Logging and Monitoring
### 💡 Principle
Implement comprehensive logging to track errors, debug issues, and monitor application health.

### ✨ Key Points
- Structured Log Format
- Different Log Levels
- Centralized Logging
- Error Tracking

### 📝 Examples

#### Node.js
❌ **Poor: Console Logging**
```javascript
try {
  await processOrder(orderData);
} catch (error) {
  console.log('Error:', error);
}
```

✅ **Clean: Structured Logging**
```javascript
const winston = require('winston');
const { combine, timestamp, json, errors } = winston.format;

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: combine(
    errors({ stack: true }),
    timestamp(),
    json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ 
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    })
  ]
});

const processOrderWithLogging = async (orderData) => {
  try {
    logger.info('Processing order', { 
      orderId: orderData.id,
      amount: orderData.amount 
    });
    
    await processOrder(orderData);
    
    logger.info('Order processed successfully', { 
      orderId: orderData.id 
    });
  } catch (error) {
    logger.error('Order processing failed', {
      orderId: orderData.id,
      error: error.message,
      stack: error.stack
    });
    throw error;
  }
};
```

## 4. Custom Error Types
### 💡 Principle
Create specific error types for different categories of errors to improve error handling and debugging.

### ✨ Key Points
- Descriptive Error Names
- Meaningful Error Messages
- Error Categorization
- Stack Trace Preservation

### 📝 Examples

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
    this.statusCode = 400;
  }
}

class NotFoundError extends Error {
  constructor(resource) {
    super(`${resource} not found`);
    this.name = 'NotFoundError';
    this.statusCode = 404;
  }
}

class DatabaseError extends Error {
  constructor(operation, originalError) {
    super(`Database ${operation} failed: ${originalError.message}`);
    this.name = 'DatabaseError';
    this.statusCode = 500;
    this.originalError = originalError;
  }
}

// Usage
async function getUserById(id) {
  try {
    if (!id) {
      throw new ValidationError('User ID is required');
    }

    const user = await User.findById(id);
    if (!user) {
      throw new NotFoundError('User');
    }

    return user;
  } catch (error) {
    if (error instanceof ValidationError || 
        error instanceof NotFoundError) {
      throw error;
    }
    throw new DatabaseError('query', error);
  }
}
```

## 📚 Additional Resources
- [Error Handling Practices in Node.js](https://www.joyent.com/node-js/production/design/errors)
- [Laravel Error Handling Documentation](https://laravel.com/docs/errors)
- [JavaScript Error Handling Best Practices](https://www.toptal.com/javascript/error-handling-best-practices)
