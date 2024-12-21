
### **1. Modular Design**
**Break code into smaller, reusable modules.**

#### Node.js Example:
**Poor:**
```javascript
// Single large file with all logic
const express = require('express');
const app = express();

app.post('/register', (req, res) => {
  const user = { name: req.body.name, email: req.body.email };
  // Save user to database
  res.status(201).send(user);
});

app.listen(3000);
```

**Clean:**
```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.post('/register', userController.register);

module.exports = router;

// controllers/userController.js
exports.register = (req, res) => {
  const user = { name: req.body.name, email: req.body.email };
  res.status(201).send(user);
};

// app.js
const express = require('express');
const app = express();
const userRoutes = require('./routes/userRoutes');

app.use(express.json());
app.use('/api/users', userRoutes);

app.listen(3000);
```

---

### **2. Object-Oriented Programming**
**Utilize classes and objects for modularity and reusability.**

#### Node.js Example:
**Poor:**
```javascript
const dog = { name: 'Buddy', bark: () => console.log('Woof!') };
const cat = { name: 'Kitty', meow: () => console.log('Meow!') };

dog.bark();
cat.meow();
```

**Clean:**
```javascript
class Animal {
  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  makeSound() {
    console.log(`${this.sound}!`);
  }
}

const dog = new Animal('Buddy', 'Woof');
const cat = new Animal('Kitty', 'Meow');

dog.makeSound(); // Woof!
cat.makeSound(); // Meow!
```

#### Laravel Example:
**Poor:**
```php
$user = DB::table('users')->where('id', $id)->first();
if ($user->status == 'active') {
    // Do something
}
```

**Clean:**
```php
class User extends Model {
    public function isActive() {
        return $this->status === 'active';
    }
}

$user = User::find($id);
if ($user->isActive()) {
    // Do something
}
```

---

### **3. Function Decomposition**
**Divide code into smaller, focused functions.**

#### Node.js Example:
**Poor:**
```javascript
function processUsers(users) {
  for (const user of users) {
    if (user.isActive) {
      console.log(user.name);
    }
  }
}
```

**Clean:**
```javascript
function isActive(user) {
  return user.isActive;
}

function printActiveUserNames(users) {
  users.filter(isActive).forEach(user => console.log(user.name));
}
```

#### Laravel Example:
**Poor:**
```php
foreach ($users as $user) {
    if ($user->status === 'active') {
        echo $user->name;
    }
}
```

**Clean:**
```php
function getActiveUsers($users) {
    return $users->filter(fn($user) => $user->isActive());
}

function printUserNames($users) {
    foreach ($users as $user) {
        echo $user->name;
    }
}

$activeUsers = getActiveUsers($users);
printUserNames($activeUsers);
```
