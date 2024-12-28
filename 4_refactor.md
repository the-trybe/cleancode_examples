

## **1. Extract Methods to Improve Code Organization**

### **Node.js**

**Bad Example:**
```javascript
app.post('/create-order', async (req, res) => {
  const { user, items } = req.body;

  // Validate input
  if (!items || items.length === 0) {
    return res.status(400).send('Order must contain at least one item');
  }
  if (!user || !user.id) {
    return res.status(400).send('User information is required');
  }

  // Calculate total price
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

  // Save order
  const order = await OrderModel.create({ user, items, total });
  res.status(201).send(order);
});
```

**Good Example:**
```javascript
class Order {
  constructor(user, items) {
    this.user = user;
    this.items = items;
    this.total = this.calculateTotal();
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  isValid() {
    if (!this.items || this.items.length === 0) {
      throw new Error('Order must contain at least one item');
    }
    if (!this.user || !this.user.id) {
      throw new Error('User information is required');
    }
  }
}

async function saveOrder(order) {
  return await OrderModel.create(order);
}

app.post('/create-order', async (req, res) => {
  try {
    const { user, items } = req.body;

    const order = new Order(user, items);
    order.isValid();

    const savedOrder = await saveOrder(order);
    res.status(201).send(savedOrder);
  } catch (error) {
    res.status(400).send({ error: error.message });
  }
});
```

---

### **Laravel**

**Bad Example:**
```php
public function store(Request $request) {
    $data = $request->all();

    // Validate input
    if (!isset($data['name']) || !isset($data['email'])) {
        return response()->json(['error' => 'Name and email are required'], 400);
    }

    // Save user
    $user = User::create($data);

    // Calculate total
    $total = collect($data['items'])->reduce(function ($sum, $item) {
        return $sum + ($item['price'] * $item['quantity']);
    }, 0);

    // Save order
    Order::create(['user_id' => $user->id, 'total' => $total]);

    return response()->json(['message' => 'Order created successfully']);
}
```

**Good Example:**
```php
// Form Request: StoreOrderRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreOrderRequest extends FormRequest {
    public function rules() {
        return [
            'user_id' => 'required|exists:users,id',
            'items' => 'required|array|min:1',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ];
    }
}

// Helper: OrderHelper.php
namespace App\Helpers;

use App\Models\Order;
use App\Models\Product;

class OrderHelper {
    public static function calculateTotal($items) {
        return collect($items)->reduce(function ($total, $item) {
            $product = Product::findOrFail($item['product_id']);
            return $total + ($product->price * $item['quantity']);
        }, 0);
    }

    public static function saveOrder($userId, $items, $total) {
        return Order::create([
            'user_id' => $userId,
            'items' => json_encode($items),
            'total' => $total,
        ]);
    }
}

// Controller: OrderController.php
namespace App\Http\Controllers;

use App\Helpers\OrderHelper;
use App\Http\Requests\StoreOrderRequest;

class OrderController extends Controller {
    public function store(StoreOrderRequest $request) {
        $data = $request->validated();

        $total = OrderHelper::calculateTotal($data['items']);
        $order = OrderHelper::saveOrder($data['user_id'], $data['items'], $total);

        return response()->json(['message' => 'Order created successfully', 'order' => $order]);
    }
}
```

---

## **2. Refactor Code to Reduce Duplication**

### **Node.js**

**Bad Example (with Pagination):**
```javascript
app.get('/active-users', async (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const skip = (page - 1) * limit;

  try {
    const users = await User.find({ status: 'active' }).skip(skip).limit(Number(limit));
    const total = await User.countDocuments({ status: 'active' });

    res.send({
      data: users,
      pagination: {
        total,
        page: Number(page),
        limit: Number(limit),
      },
    });
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});

app.get('/inactive-users', async (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const skip = (page - 1) * limit;

  try {
    const users = await User.find({ status: 'inactive' }).skip(skip).limit(Number(limit));
    const total = await User.countDocuments({ status: 'inactive' });

    res.send({
      data: users,
      pagination: {
        total,
        page: Number(page),
        limit: Number(limit),
      },
    });
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});
```

**Good Example:**
```javascript
function paginate(query, page, limit) {
  const skip = (page - 1) * limit;
  return query.skip(skip).limit(limit);
}

app.get('/:status-users', async (req, res) => {
  const { status } = req.params;
  const { page = 1, limit = 10 } = req.query;

  try {
    const usersQuery = User.find({ status });
    const paginatedUsers = await paginate(usersQuery, Number(page), Number(limit));
    const total = await User.countDocuments({ status });

    res.send({
      data: paginatedUsers,
      pagination: {
        total,
        page: Number(page),
        limit: Number(limit),
      },
    });
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});
```

---

### **Laravel**

**Bad Example (with Pagination):**
```php
public function activeUsers(Request $request) {
    $perPage = $request->input('perPage', 10);
    $page = $request->input('page', 1);

    $query = User::where('status', 'active');
    $users = $query->paginate($perPage, ['*'], 'page', $page);

    return response()->json([
        'data' => $users->items(),
        'pagination' => [
            'total' => $users->total(),
            'current_page' => $users->currentPage(),
            'per_page' => $users->perPage(),
        ],
    ]);
}

public function inactiveUsers(Request $request) {
    $perPage = $request->input('perPage', 10);
    $page = $request->input('page', 1);

    $query = User::where('status', 'inactive');
    $users = $query->paginate($perPage, ['*'], 'page', $page);

    return response()->json([
        'data' => $users->items(),
        'pagination' => [
            'total' => $users->total(),
            'current_page' => $users->currentPage(),
            'per_page' => $users->perPage(),
        ],
    ]);
}
```

**Good Example:**
```php
namespace App\Helpers;

class PaginationHelper {
    public static function paginate($query, $perPage, $page) {
        return $query->paginate($perPage, ['*'], 'page', $page);
    }
}

namespace App\Http\Controllers;

use App\Models\User;
use App\Helpers\PaginationHelper;
use Illuminate\Http\Request;

class UserController extends Controller {
    public function getUsersByStatus(Request $request, $status) {
        $perPage = $request->input('perPage', 10);
        $page = $request->input('page', 1);

        $query = User::where('status', $status);
        $users = PaginationHelper::paginate($query, $perPage, $page);

        return response()->json([
            'data' => $users->items(),
            'pagination' => [
                'total' => $users->total(),
                'current_page' => $users->currentPage(),
                'per_page' => $users->perPage(),
            ],
        ]);
    }
}
