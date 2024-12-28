# Core Principles of Clean Code

> "Clean code is not written by following a set of rules. You don't become a software craftsman by learning a list of heuristics. Professionalism and craftsmanship come from values that drive disciplines." - Robert C. Martin

## Table of Contents
1. [Clarity](#1-clarity)
2. [Consistency](#2-consistency)
3. [Separation of Concerns](#3-separation-of-concerns)
4. [Avoiding Over-Engineering](#4-avoiding-over-engineering)
5. [Readability Through Simplicity](#5-readability-through-simplicity)

---

## 1. Clarity
### 💡 Principle
Write code that explains itself. Avoid ambiguous or unclear expressions. The best code is self-documenting and requires minimal additional explanation.

### ✨ Key Points
- Use meaningful and descriptive names
- Make intentions clear
- Avoid magic numbers and cryptic abbreviations

### 📝 Examples

#### Node.js
❌ **Poor:**
```javascript
function x(a, b) {
  return a * b * 0.05;
}
```

✅ **Clean:**
```javascript
function calculateDiscountedPrice(price, discountRate) {
  const TAX_RATE = 0.05;
  return price * discountRate * TAX_RATE;
}
```

#### Laravel
❌ **Poor:**
```php
$user = User::find($id);
if ($user->status == 1) {
    // do something
}
```

✅ **Clean:**
```php
$user = User::find($id);
if ($user->isActive()) {
    // Perform necessary action
}

// In User model:
public function isActive(): bool
{
    return $this->status === self::STATUS_ACTIVE;
}
```

---

## 2. Consistency
### 💡 Principle
Maintain consistent naming conventions, formatting, and patterns throughout the codebase. Consistency makes code more predictable and easier to understand.

### ✨ Key Points
- Follow established naming conventions
- Use consistent formatting
- Maintain consistent file structure
- Apply consistent error handling patterns

### 📝 Examples

#### Node.js
❌ **Poor:**
```javascript
const getUserDetails = () => { /* ... */ };
function fetch_user_data() { /* ... */ }
const get_UserAge = () => { /* ... */ };
```

✅ **Clean:**
```javascript
const getUserDetails = () => { /* ... */ };
const fetchUserData = () => { /* ... */ };
const getUserAge = () => { /* ... */ };
```

#### Laravel
❌ **Poor:**
```php
public function ShowUserInfo($id) {
    $user = User::Find($id);
    return view('userProfile')->with('user', $user);
}
```

✅ **Clean:**
```php
public function showUserInfo($id): View
{
    $user = User::findOrFail($id);
    return view('user.profile', compact('user'));
}
```

---

## 3. Separation of Concerns
### 💡 Principle
Keep different aspects of functionality separate and modular. Each component should have a single, well-defined responsibility.

### ✨ Key Points
- Single Responsibility Principle
- Modular design
- Clear boundaries between layers
- Dependency injection

### 📝 Examples

#### Node.js
❌ **Poor:**
```javascript
app.post('/register', async (req, res) => {
  const user = new User(req.body);
  await user.save();
  sendWelcomeEmail(user.email);
  res.status(201).send(user);
});
```

✅ **Clean:**
```javascript
// controllers/userController.js
async function registerUser(req, res) {
  try {
    const user = await userService.createUser(req.body);
    await emailService.sendWelcomeEmail(user);
    res.status(201).json({ success: true, data: user });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
}

// services/userService.js
class UserService {
  async createUser(userData) {
    const user = new User(userData);
    return await user.save();
  }
}

// services/emailService.js
class EmailService {
  async sendWelcomeEmail(user) {
    // Email sending logic
  }
}
```

---

## 4. Avoiding Over-Engineering
### 💡 Principle
Keep solutions simple and avoid unnecessary complexity. Don't add abstractions until they're needed.

### ✨ Key Points
- YAGNI (You Aren't Gonna Need It)
- Start simple, evolve as needed
- Avoid premature optimization
- Question each abstraction

### 📝 Examples

#### Node.js
❌ **Poor:**
```javascript
class StringUtils {
  static concatenate(str1, str2) {
    return str1 + str2;
  }
}

const fullName = StringUtils.concatenate(firstName, lastName);
```

✅ **Clean:**
```javascript
const fullName = `${firstName} ${lastName}`;
```

---

## 5. Readability Through Simplicity
### 💡 Principle
Write code that is easy to read and understand. Complex logic should be broken down into smaller, well-named pieces.

### ✨ Key Points
- Short, focused functions
- Clear variable names
- Minimal nesting
- Descriptive comments when needed

### 📝 Examples

#### Node.js
❌ **Poor:**
```javascript
function processData(data) {
  const result = data.filter(x => x.a > 10)
                    .map(x => ({ val: x.a * 2, id: x.b }))
                    .filter(x => x.val < 50)
                    .reduce((acc, x) => acc + x.val, 0);
  return result;
}
```

✅ **Clean:**
```javascript
function filterLargeValues(items) {
  return items.filter(item => item.value > 10);
}

function transformData(items) {
  return items.map(item => ({
    value: item.value * 2,
    id: item.id
  }));
}

function filterExcessiveValues(items) {
  return items.filter(item => item.value < 50);
}

function sumValues(items) {
  return items.reduce((total, item) => total + item.value, 0);
}

function processData(data) {
  const largeValues = filterLargeValues(data);
  const transformedData = transformData(largeValues);
  const filteredData = filterExcessiveValues(transformedData);
  return sumValues(filteredData);
}
```

## 📚 Additional Resources
- [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [The Pragmatic Programmer](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/)
- [Laravel Best Practices](https://github.com/alexeymezenin/laravel-best-practices)
