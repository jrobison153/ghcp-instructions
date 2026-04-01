# Software Craft Instructions

These instructions capture stack-agnostic software engineering practices from the project guidance.

## Code Quality Standards

### Modularity
- Break functionality into small, focused modules.
- Ensure each module has a single, well-defined responsibility.
- Prefer composition over inheritance where practical.
- Keep files small and cohesive (target 100-200 lines; above 300 is a code smell).

**Example - Single Responsibility:**
```
// ❌ Bad: Multiple responsibilities
class UserManager {
  validateUser() { ... }
  saveToDatabase() { ... }
  sendEmail() { ... }
  generateReport() { ... }
}

// ✅ Good: Separate concerns
class UserValidator { validate(user) { ... } }
class UserRepository { save(user) { ... } }
class EmailService { send(to, message) { ... } }
class ReportGenerator { generate(user) { ... } }
```

**Example - Composition over Inheritance:**
```
// ❌ Bad: Deep inheritance
class Animal { }
class Mammal extends Animal { }
class Dog extends Mammal { }
class Puppy extends Dog { }

// ✅ Good: Composition
class Animal {
  constructor(traits) {
    this.traits = traits;
  }
}
const puppy = new Animal({
  canBark: true,
  isYoung: true,
  feedsMilk: true
});
```

### Testing Requirements
- Ensure every production source file has corresponding unit tests.
- Place unit tests in a consistent, project-level test location (for example, `tests/unit`).
- Use a consistent test file naming convention (for example, `*.test` or `*.spec` with your project's extension).
- Mirror source directory structure in test directories where possible.
- Target meaningful coverage (for example, at least 80% branch coverage).
- Keep tests independent so they can run in any order.
- Create explicit test doubles to isolate units under test (prefer explicit doubles over framework mock functions).
- Follow test-driven development (TDD) when feasible.
- Add acceptance/end-to-end tests for critical user flows.
- Place acceptance/end-to-end tests in a dedicated folder (for example, `tests/acceptance` or `tests/e2e`).

**Test Double Organization:**
- Place test doubles in `tests/doubles/` with subdirectories by type:
  - `tests/doubles/dummies/` - Objects passed but never used
  - `tests/doubles/stubs/` - Provide canned responses to calls
  - `tests/doubles/spies/` - Record information about calls
  - `tests/doubles/mocks/` - Pre-programmed with expectations
  - `tests/doubles/fakes/` - Working implementations with shortcuts

**Example - Test Organization:**
```
src/
  services/
    userService.js
    orderService.js
  utils/
    validator.js

tests/
  unit/
    services/
      userService.test.js
      orderService.test.js
    utils/
      validator.test.js
  acceptance/
    userCheckout.e2e.test.js
  doubles/
    dummies/
      dummyLogger.js
    stubs/
      emailServiceStub.js
      paymentGatewayStub.js
    spies/
      emailServiceSpy.js
    mocks/
      databaseMock.js
    fakes/
      inMemoryDatabase.js
      inMemoryCache.js
```

**Example - Test Independence:**
```
// ❌ Bad: Tests depend on each other
test('create user', () => {
  user = createUser('alice');  // Sets shared state
});
test('update user', () => {
  updateUser(user);  // Depends on previous test
});

// ✅ Good: Each test is independent
test('create user', () => {
  const user = createUser('alice');
  expect(user.name).toBe('alice');
});
test('update user', () => {
  const user = createUser('bob');  // Creates own data
  const updated = updateUser(user, { name: 'robert' });
  expect(updated.name).toBe('robert');
});
```

### Dependency Inversion
- Invert dependencies to improve testability and maintainability.
- Depend on abstractions (interfaces, contracts, protocols), not concrete implementations.
- Use dependency injection for external dependencies.
- Make dependencies explicit through constructors, function parameters, or configuration.

**Example - Depend on Abstractions:**
```
// ❌ Bad: Depends on concrete implementation
class OrderService {
  constructor() {
    this.database = new MySQLDatabase();  // Tightly coupled
    this.emailer = new SendGridEmailer();
  }
}

// ✅ Good: Depends on abstractions
class OrderService {
  constructor(database, emailer) {  // Inject dependencies
    this.database = database;  // Any database implementation
    this.emailer = emailer;    // Any email implementation
  }
}

// Usage with different implementations
const prodService = new OrderService(
  new MySQLDatabase(),
  new SendGridEmailer()
);

const testService = new OrderService(
  new InMemoryDatabase(),
  new MockEmailer()
);
```

**Example - Explicit Dependencies:**
```
// ❌ Bad: Hidden dependencies
function processPayment(amount) {
  const gateway = new StripeGateway();  // Hidden dependency
  return gateway.charge(amount);
}

// ✅ Good: Explicit dependencies
function processPayment(amount, paymentGateway) {
  return paymentGateway.charge(amount);
}
```

### Cyclomatic Complexity
- Keep function and method complexity low (for example, target 3 or less when practical).
- Limit functions/methods to approximately 30 lines of code (one screen in a typical IDE).
- Extract blocks of code into well-named helper functions.
- Compose functions from other well-named functions so code reads like a book.
- Each function should tell a clear story at a single level of abstraction.
- Avoid deeply nested conditionals and loops.
- Use early returns to reduce nesting.
- Prefer guard clauses over nested condition blocks.

**Example - Reduce Nesting with Guard Clauses:**
```
// ❌ Bad: Deeply nested (complexity = 5)
function processOrder(order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.isPaid) {
        if (order.isValidAddress) {
          return shipOrder(order);
        } else {
          throw new Error('Invalid address');
        }
      } else {
        throw new Error('Order not paid');
      }
    } else {
      throw new Error('No items');
    }
  } else {
    throw new Error('No order');
  }
}

// ✅ Good: Guard clauses (complexity = 3)
function processOrder(order) {
  if (!order) throw new Error('No order');
  if (order.items.length === 0) throw new Error('No items');
  if (!order.isPaid) throw new Error('Order not paid');
  if (!order.isValidAddress) throw new Error('Invalid address');
  
  return shipOrder(order);
}
```

**Example - Extract Complex Logic:**
```
// ❌ Bad: Complex single function
function calculateDiscount(customer, cart) {
  let discount = 0;
  if (customer.isPremium) {
    if (cart.total > 100) {
      discount = 0.2;
    } else if (cart.total > 50) {
      discount = 0.1;
    }
  } else {
    if (cart.total > 200) {
      discount = 0.1;
    }
  }
  if (customer.hasReferralCode) {
    discount += 0.05;
  }
  return discount;
}

// ✅ Good: Extracted helper functions
function calculateDiscount(customer, cart) {
  const tierDiscount = getTierDiscount(customer, cart);
  const referralDiscount = getReferralDiscount(customer);
  return tierDiscount + referralDiscount;
}

function getTierDiscount(customer, cart) {
  if (customer.isPremium) return getPremiumDiscount(cart.total);
  return getStandardDiscount(cart.total);
}

function getPremiumDiscount(total) {
  if (total > 100) return 0.2;
  if (total > 50) return 0.1;
  return 0;
}

function getStandardDiscount(total) {
  return total > 200 ? 0.1 : 0;
}

function getReferralDiscount(customer) {
  return customer.hasReferralCode ? 0.05 : 0;
}
```

**Example - Functions That Read Like a Book:**
```
// ❌ Bad: Long function with mixed abstraction levels (60+ lines)
function processOrder(orderId) {
  const order = database.query('SELECT * FROM orders WHERE id = ?', [orderId]);
  if (!order) throw new Error('Order not found');
  
  if (order.status !== 'pending') throw new Error('Order already processed');
  
  let totalAmount = 0;
  for (const item of order.items) {
    const product = database.query('SELECT * FROM products WHERE id = ?', [item.productId]);
    if (!product) throw new Error('Product not found');
    if (product.stock < item.quantity) throw new Error('Insufficient stock');
    totalAmount += product.price * item.quantity;
  }
  
  const taxRate = order.shippingAddress.country === 'US' ? 0.08 : 0.20;
  const tax = totalAmount * taxRate;
  const finalAmount = totalAmount + tax;
  
  const payment = paymentGateway.charge(order.customerId, finalAmount);
  if (!payment.success) throw new Error('Payment failed');
  
  for (const item of order.items) {
    database.execute(
      'UPDATE products SET stock = stock - ? WHERE id = ?',
      [item.quantity, item.productId]
    );
  }
  
  database.execute('UPDATE orders SET status = ? WHERE id = ?', ['completed', orderId]);
  
  const customer = database.query('SELECT * FROM customers WHERE id = ?', [order.customerId]);
  emailService.send(
    customer.email,
    'Order Confirmation',
    `Your order ${orderId} has been processed. Total: $${finalAmount}`
  );
  
  return { orderId, amount: finalAmount, status: 'completed' };
}

// ✅ Good: Reads like a book (each function ~10-30 lines)
function processOrder(orderId) {
  const order = getValidatedOrder(orderId);
  const totalAmount = calculateOrderTotal(order);
  const finalAmount = applyTaxes(totalAmount, order.shippingAddress);
  
  processPayment(order.customerId, finalAmount);
  updateInventory(order.items);
  markOrderComplete(orderId);
  sendConfirmationEmail(order, orderId, finalAmount);
  
  return createOrderResult(orderId, finalAmount);
}

function getValidatedOrder(orderId) {
  const order = findOrderById(orderId);
  ensureOrderIsPending(order);
  return order;
}

function findOrderById(orderId) {
  const order = database.query('SELECT * FROM orders WHERE id = ?', [orderId]);
  if (!order) throw new Error('Order not found');
  return order;
}

function ensureOrderIsPending(order) {
  if (order.status !== 'pending') {
    throw new Error('Order already processed');
  }
}

function calculateOrderTotal(order) {
  let total = 0;
  for (const item of order.items) {
    validateItemAvailability(item);
    total += calculateItemPrice(item);
  }
  return total;
}

function validateItemAvailability(item) {
  const product = findProductById(item.productId);
  ensureSufficientStock(product, item.quantity);
}

function findProductById(productId) {
  const product = database.query('SELECT * FROM products WHERE id = ?', [productId]);
  if (!product) throw new Error('Product not found');
  return product;
}

function ensureSufficientStock(product, requestedQuantity) {
  if (product.stock < requestedQuantity) {
    throw new Error('Insufficient stock');
  }
}

function calculateItemPrice(item) {
  const product = findProductById(item.productId);
  return product.price * item.quantity;
}

function applyTaxes(amount, shippingAddress) {
  const taxRate = getTaxRate(shippingAddress.country);
  const tax = amount * taxRate;
  return amount + tax;
}

function getTaxRate(country) {
  return country === 'US' ? 0.08 : 0.20;
}

function processPayment(customerId, amount) {
  const payment = paymentGateway.charge(customerId, amount);
  if (!payment.success) throw new Error('Payment failed');
}

function updateInventory(items) {
  for (const item of items) {
    decrementProductStock(item.productId, item.quantity);
  }
}

function decrementProductStock(productId, quantity) {
  database.execute(
    'UPDATE products SET stock = stock - ? WHERE id = ?',
    [quantity, productId]
  );
}

function markOrderComplete(orderId) {
  database.execute('UPDATE orders SET status = ? WHERE id = ?', ['completed', orderId]);
}

function sendConfirmationEmail(order, orderId, amount) {
  const customer = findCustomerById(order.customerId);
  const message = buildConfirmationMessage(orderId, amount);
  emailService.send(customer.email, 'Order Confirmation', message);
}

function findCustomerById(customerId) {
  return database.query('SELECT * FROM customers WHERE id = ?', [customerId]);
}

function buildConfirmationMessage(orderId, amount) {
  return `Your order ${orderId} has been processed. Total: $${amount}`;
}

function createOrderResult(orderId, amount) {
  return { orderId, amount, status: 'completed' };
}
```

### Code Documentation
- Document all public functions, methods, classes, or modules.
- Include:
  - A brief description of behavior and intent.
  - Parameter descriptions.
  - Return value description.
  - Possible error/failure conditions.
  - Examples for complex behavior.
- Document non-obvious business logic with concise inline comments.
- Keep comments and documentation in sync with code changes.

**Example - Function Documentation:**
```
/**
 * Calculates the total price including tax and applicable discounts.
 * 
 * @param {Object} cart - The shopping cart object
 * @param {Array} cart.items - Array of items in cart
 * @param {Object} customer - Customer information
 * @param {boolean} customer.isPremium - Premium membership status
 * @param {string} taxRegion - Tax region code (e.g., 'US-CA', 'EU-FR')
 * @returns {number} Final price including tax and discounts
 * @throws {Error} If cart is empty or taxRegion is invalid
 * 
 * @example
 * const total = calculateTotal(
 *   { items: [{ price: 10 }, { price: 20 }] },
 *   { isPremium: true },
 *   'US-CA'
 * );
 * // Returns: 27.60 (with tax and premium discount)
 */
function calculateTotal(cart, customer, taxRegion) {
  // Implementation
}
```

**Example - Inline Comments for Business Logic:**
```
function processRefund(order) {
  // Per company policy: refunds over $500 require manager approval
  if (order.total > 500 && !order.hasManagerApproval) {
    return requestManagerApproval(order);
  }
  
  // Stripe charges 2.9% + $0.30 per transaction; we absorb this on refunds
  const refundAmount = order.total;
  const processingFee = (order.total * 0.029) + 0.30;
  
  return issueRefund(refundAmount, processingFee);
}
```

### Expressive Code
- Use descriptive names for functions, variables, classes, and methods.
- Ensure names communicate intent and domain meaning.
- Avoid abbreviations unless widely understood in your domain.
- Use verb phrases for behavior (for example, `calculateTotal`, `fetchUserData`).
- Use noun phrases for data and entities (for example, `userProfile`, `OrderProcessor`).
- Prefix boolean names with `is`, `has`, `should`, or similar forms.
- Avoid generic names like `data`, `info`, `handle`, or `process` without context.

**Example - Descriptive Names:**
```
// ❌ Bad: Vague and unclear
function proc(d) {
  const r = d.filter(x => x.a);
  return r;
}

// ✅ Good: Clear and expressive
function getActiveUsers(users) {
  const activeUsers = users.filter(user => user.isActive);
  return activeUsers;
}
```

**Example - Boolean Naming:**
```
// ❌ Bad: Unclear boolean names
const premium = user.status === 'premium';
const validated = checkData(input);
const empty = items.length === 0;

// ✅ Good: Clear boolean names
const isPremiumMember = user.status === 'premium';
const hasValidatedEmail = checkEmailVerification(input);
const shouldShowEmptyState = items.length === 0;
```

**Example - Verb vs Noun Phrases:**
```
// ❌ Bad: Inconsistent naming
function userValidation(user) { ... }      // Function should be verb
const processOrder = { id: 1, items: [] }; // Data should be noun

// ✅ Good: Consistent naming
function validateUser(user) { ... }        // Verb for function
const order = { id: 1, items: [] };        // Noun for data
class OrderProcessor { ... }               // Noun for class
```

**Example - Avoid Abbreviations:**
```
// ❌ Bad: Unclear abbreviations
const usrMgr = new UserManager();
const qty = item.quantity;
const addr = customer.address;

// ✅ Good: Full words (exceptions: id, url, api, etc.)
const userManager = new UserManager();
const quantity = item.quantity;
const address = customer.address;
const userId = user.id;  // 'id' is acceptable
```

## Type and Contract Safety
- Enable strict static checks where available.
- Model data shapes and contracts explicitly.
- Prefer immutability for values that should not change.
- Avoid untyped or weakly typed escape hatches unless necessary.
- Use language tooling to catch errors at build/compile time whenever possible.

**Example - Explicit Contracts:**
```
// ❌ Bad: No type contract
function createUser(data) {
  return {
    name: data.name,
    email: data.email
  };
}

// ✅ Good: Explicit contract (TypeScript example)
interface UserData {
  name: string;
  email: string;
  age?: number;
}

function createUser(data: UserData): User {
  return {
    name: data.name,
    email: data.email
  };
}

// ✅ Good: Runtime validation (JavaScript)
function createUser(data) {
  if (typeof data.name !== 'string') {
    throw new Error('name must be a string');
  }
  if (typeof data.email !== 'string') {
    throw new Error('email must be a string');
  }
  return { name: data.name, email: data.email };
}
```

**Example - Immutability:**
```
// ❌ Bad: Mutating data
function addItem(cart, item) {
  cart.items.push(item);  // Mutates original
  return cart;
}

// ✅ Good: Immutable approach
function addItem(cart, item) {
  return {
    ...cart,
    items: [...cart.items, item]  // New array
  };
}

// ✅ Good: Frozen objects (when appropriate)
const CONFIG = Object.freeze({
  API_URL: 'https://api.example.com',
  TIMEOUT: 5000
});
```

## Testing Guidelines
- Use descriptive test names that state expected behavior.
- Follow Arrange-Act-Assert (AAA) or an equivalent clear structure.
- Isolate external systems in tests using explicit test doubles.
- Cover edge cases, invalid input, and error paths.
- Write tests before implementation when practical.

### Test Double Guidelines
Prefer creating explicit test doubles over using framework mock functions. Each type serves a specific purpose:

**Dummy** - Object passed but never actually used (satisfies parameter requirements):
- Use when a parameter is required but not relevant to the test
- Contains no logic, just fulfills interface requirements
- Example: Logger passed to a service when testing logic that doesn't log

**Stub** - Provides canned responses to calls:
- Use when you need controlled responses from dependencies
- Returns predetermined values regardless of input
- No assertion logic, just provides data
- Example: Payment gateway that always returns success

**Spy** - Records information about calls made to it:
- Use when you need to verify interactions happened
- Tracks all calls, arguments, and invocation order
- Can provide answers like stubs
- Example: Email service that records all sent emails for verification

**Mock** - Pre-programmed with expectations about calls:
- Use when you need to verify specific call patterns
- Contains assertion logic and expectations
- Fails if expectations aren't met
- Example: Database that expects specific queries in order

**Fake** - Working implementation with shortcuts:
- Use when you need realistic behavior without external dependencies
- Contains business logic but uses simpler implementation
- More complex than other doubles but still controlled
- Example: In-memory database instead of real database

**Choosing the Right Double:**
- Default to **Stub** for simple return values
- Use **Spy** when you need to verify calls were made
- Use **Mock** when call order and specific arguments matter
- Use **Fake** when you need stateful, realistic behavior
- Use **Dummy** when you just need to satisfy a parameter

**Example - Descriptive Test Names:**
```
// ❌ Bad: Vague test names
test('user test', () => { ... });
test('it works', () => { ... });
test('test2', () => { ... });

// ✅ Good: Descriptive test names
test('should create user with valid email and name', () => { ... });
test('should throw error when email is invalid', () => { ... });
test('should return null when user is not found', () => { ... });
```

**Example - Arrange-Act-Assert Pattern:**
```
test('should calculate discount for premium members', () => {
  // Arrange: Set up test data and dependencies
  const customer = { isPremium: true, hasReferralCode: false };
  const cart = { total: 150, items: [{ price: 150 }] };
  
  // Act: Execute the function under test
  const discount = calculateDiscount(customer, cart);
  
  // Assert: Verify the expected outcome
  expect(discount).toBe(0.2);
});
```

**Example - Using Test Doubles:**
```
// ❌ Bad: Testing with real external dependencies
test('should send email after order', async () => {
  const order = { id: 1, email: 'test@example.com' };
  await processOrder(order);  // Actually sends email!
});

// ❌ Bad: Using framework mock functions
test('should send email after order', async () => {
  const mockEmailer = {
    send: jest.fn().mockResolvedValue(true)  // Framework-dependent
  };
  const orderService = new OrderService(mockEmailer);
  await orderService.processOrder({ id: 1, email: 'test@example.com' });
  expect(mockEmailer.send).toHaveBeenCalled();
});

// ✅ Good: Using explicit test doubles
// tests/doubles/spies/emailServiceSpy.js
class EmailServiceSpy {
  constructor() {
    this.calls = [];
  }
  
  async send(to, subject, body) {
    this.calls.push({ to, subject, body, timestamp: Date.now() });
    return true;
  }
  
  wasCalledWith(to) {
    return this.calls.some(call => call.to === to);
  }
  
  getCallCount() {
    return this.calls.length;
  }
}

// tests/unit/services/orderService.test.js
test('should send email after order', async () => {
  // Arrange: Use explicit spy
  const emailSpy = new EmailServiceSpy();
  const orderService = new OrderService(emailSpy);
  const order = { id: 1, email: 'test@example.com' };
  
  // Act
  await orderService.processOrder(order);
  
  // Assert: Verify using spy's interface
  expect(emailSpy.wasCalledWith('test@example.com')).toBe(true);
  expect(emailSpy.getCallCount()).toBe(1);
});
```

**Example - Test Double Types:**
```
// Dummy: Passed but never actually used
// tests/doubles/dummies/dummyLogger.js
class DummyLogger {
  log() {}
  error() {}
  warn() {}
}

// Stub: Returns canned responses
// tests/doubles/stubs/paymentGatewayStub.js
class PaymentGatewayStub {
  async charge(amount) {
    return { success: true, transactionId: 'stub-12345' };
  }
}

// Spy: Records information about calls
// tests/doubles/spies/emailServiceSpy.js
class EmailServiceSpy {
  constructor() {
    this.sentEmails = [];
  }
  
  async send(to, subject, body) {
    this.sentEmails.push({ to, subject, body });
    return true;
  }
  
  getSentCount() {
    return this.sentEmails.length;
  }
}

// Mock: Pre-programmed with expectations
// tests/doubles/mocks/databaseMock.js
class DatabaseMock {
  constructor() {
    this.expectations = [];
    this.responses = new Map();
  }
  
  expect(method, args, response) {
    this.expectations.push({ method, args });
    this.responses.set(method, response);
  }
  
  async query(sql) {
    const expectation = this.expectations.find(e => e.method === 'query');
    if (!expectation) {
      throw new Error('Unexpected call to query');
    }
    return this.responses.get('query');
  }
  
  verify() {
    if (this.expectations.length === 0) {
      throw new Error('Not all expectations were met');
    }
  }
}

// Fake: Working implementation with shortcuts
// tests/doubles/fakes/inMemoryDatabase.js
class InMemoryDatabase {
  constructor() {
    this.data = new Map();
  }
  
  async save(key, value) {
    this.data.set(key, value);
    return value;
  }
  
  async get(key) {
    return this.data.get(key) || null;
  }
  
  async delete(key) {
    return this.data.delete(key);
  }
  
  clear() {
    this.data.clear();
  }
}
```

**Example - Testing Edge Cases:**
```
describe('calculateTotal', () => {
  test('should handle empty cart', () => {
    const result = calculateTotal({ items: [] });
    expect(result).toBe(0);
  });
  
  test('should handle null items', () => {
    expect(() => calculateTotal({ items: null }))
      .toThrow('Invalid cart');
  });
  
  test('should handle negative prices', () => {
    const cart = { items: [{ price: -10 }] };
    expect(() => calculateTotal(cart))
      .toThrow('Invalid price');
  });
  
  test('should handle very large numbers', () => {
    const cart = { items: [{ price: Number.MAX_SAFE_INTEGER }] };
    const result = calculateTotal(cart);
    expect(result).toBeLessThan(Infinity);
  });
});
```

## Error Handling
- Handle runtime and asynchronous errors explicitly.
- Return or surface user-friendly error messages at system boundaries.
- Log errors with enough context for debugging and observability.
- Avoid swallowing exceptions; either handle them or propagate them appropriately.

**Example - Explicit Error Handling:**
```
// ❌ Bad: Swallowing errors
function fetchUserData(id) {
  try {
    return database.getUser(id);
  } catch (error) {
    return null;  // Error is lost
  }
}

// ✅ Good: Explicit error handling
function fetchUserData(id) {
  try {
    return database.getUser(id);
  } catch (error) {
    logger.error('Failed to fetch user', { userId: id, error });
    throw new Error(`Unable to retrieve user ${id}`);
  }
}
```

**Example - Async Error Handling:**
```
// ❌ Bad: Unhandled promise rejection
async function loadUserProfile(id) {
  const user = await fetchUser(id);  // May throw
  const orders = await fetchOrders(id);  // May throw
  return { user, orders };
}

// ✅ Good: Proper async error handling
async function loadUserProfile(id) {
  try {
    const user = await fetchUser(id);
    const orders = await fetchOrders(id);
    return { user, orders };
  } catch (error) {
    logger.error('Failed to load user profile', { userId: id, error });
    throw new UserProfileError(`Could not load profile for user ${id}`, {
      cause: error
    });
  }
}
```

**Example - User-Friendly Error Messages:**
```
// ❌ Bad: Technical error exposed to user
app.post('/api/users', (req, res) => {
  try {
    const user = createUser(req.body);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.stack });  // Leaks internals
  }
});

// ✅ Good: User-friendly messages
app.post('/api/users', (req, res) => {
  try {
    const user = createUser(req.body);
    res.json(user);
  } catch (error) {
    logger.error('User creation failed', { body: req.body, error });
    
    if (error instanceof ValidationError) {
      return res.status(400).json({
        error: 'Invalid user data',
        details: error.validationErrors
      });
    }
    
    res.status(500).json({
      error: 'Unable to create user. Please try again later.'
    });
  }
});
```

**Example - Contextual Error Logging:**
```
// ❌ Bad: Insufficient logging context
catch (error) {
  logger.error(error.message);
}

// ✅ Good: Rich context for debugging
catch (error) {
  logger.error('Payment processing failed', {
    orderId: order.id,
    customerId: customer.id,
    amount: order.total,
    paymentMethod: payment.method,
    timestamp: new Date().toISOString(),
    error: {
      message: error.message,
      stack: error.stack,
      code: error.code
    }
  });
}
```

**Example - Error Propagation:**
```
// ❌ Bad: Converting all errors to generic error
function processData(data) {
  try {
    return transform(data);
  } catch (error) {
    throw new Error('Processing failed');  // Loses context
  }
}

// ✅ Good: Preserving error context
function processData(data) {
  try {
    return transform(data);
  } catch (error) {
    if (error instanceof ValidationError) {
      throw error;  // Re-throw specific errors
    }
    throw new ProcessingError('Data transformation failed', {
      cause: error,  // Preserve original error
      data: sanitize(data)
    });
  }
}
```

