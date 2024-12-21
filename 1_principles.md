
### **1. Clarity**
**Write code that explains itself. Avoid ambiguous or unclear expressions.**

#### Node.js Example:
Poor:
```javascript
function x(a, b) {
  return a * b * 0.05;
}
```

Clean:
```javascript
function calculateDiscountedPrice(price, discountRate) {
  return price * discountRate * 0.05;
}
```

#### Laravel Example:
Poor:
```php
$user = User::find($id);
if ($user->status == 1) {
    // do something
}
```

Clean:
```php
$user = User::find($id);
if ($user->isActive()) {
    // Perform necessary action
}
```
*Add an `isActive()` method in the User model to encapsulate the condition.*

---

### **2. Consistency**
**Use consistent naming conventions and formatting throughout the codebase.**

#### Node.js Example:
Poor:
```javascript
const getUserDetails = () => { /* ... */ };
function fetch_user_data() { /* ... */ }
```

Clean:
```javascript
const getUserDetails = () => { /* ... */ };
const fetchUserData = () => { /* ... */ };
```

#### Laravel Example:
Poor:
```php
public function ShowUserInfo($id) {
    $user = User::Find($id);
    return view('userProfile')->with('user', $user);
}
```

Clean:
```php
public function showUserInfo($id) {
    $user = User::find($id);
    return view('user.profile', compact('user'));
}
```
*Consistent camelCase method names and standardized view naming conventions.*

---

### **3. Separation of Concerns**
**Keep logic modular and responsibilities clearly separated.**

#### Node.js Example:
Poor:
```javascript
app.post('/register', async (req, res) => {
  const user = new User(req.body);
  await user.save();
  sendWelcomeEmail(user.email);
  res.status(201).send(user);
});
```

Clean:
```javascript
// controllers/userController.js
async function registerUser(req, res) {
  const user = await userService.createUser(req.body);
  await emailService.sendWelcomeEmail(user.email);
  res.status(201).send(user);
}

// services/userService.js
async function createUser(userData) {
  const user = new User(userData);
  await user.save();
  return user;
}

// app.js
app.post('/register', userController.registerUser);
```

#### Laravel Example:
Poor:
```php
public function store(Request $request) {
    $data = $request->validate([
        'name' => 'required',
        'email' => 'required|email',
    ]);
    $user = User::create($data);
    Mail::to($user->email)->send(new WelcomeMail($user));
    return redirect()->back();
}
```

Clean:
```php
public function store(StoreUserRequest $request) {
    $user = $this->userService->create($request->validated());
    $this->emailService->sendWelcomeEmail($user->email);
    return redirect()->back();
}
```
*Move validation to `StoreUserRequest`, and user creation/email logic to respective services.*

---

### **4. Avoiding Over-Engineering**
**Keep it simple and avoid unnecessary abstractions.**

#### Node.js Example:
Poor:
```javascript
class UserHandler {
  constructor(user) {
    this.user = user;
  }
  getName() {
    return this.user.name;
  }
}
```

Clean:
```javascript
function getUserName(user) {
  return user.name;
}
```

#### Laravel Example:
Poor:
```php
class UserHelper {
    public static function getUserEmail(User $user) {
        return $user->email;
    }
}
```

Clean:
```php
// Just use the property directly when needed.
$userEmail = $user->email;
```

---

### **5. Readability Through Simplicity**
**Break down complex logic into smaller, simpler functions.**

#### Node.js Example:
Poor:
```javascript
function processUserData(users) {
  const result = [];
  for (let user of users) {
    if (user.age > 18 && user.status === 'active') {
      result.push(user.name);
    }
  }
  return result;
}
```

Clean:
```javascript
function isEligibleUser(user) {
  return user.age > 18 && user.status === 'active';
}

function getEligibleUserNames(users) {
  return users.filter(isEligibleUser).map(user => user.name);
}
```

#### Laravel Example:
Poor:
```php
$activeUsers = [];
foreach (User::all() as $user) {
    if ($user->isActive()) {
        $activeUsers[] = $user->name;
    }
}
```

Clean:
```php
$activeUsers = User::where('status', 'active')->pluck('name');
```
