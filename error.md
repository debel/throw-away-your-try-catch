# Error Handling Across Languages

## A Comparative Study

---

## Introduction

* Error handling is a critical aspect of robust software development
* Different languages adopt various approaches
* We'll explore the strengths and weaknesses of each

---

## Throw-Catch Mechanism in JavaScript

```javascript
try {
  if (someCondition) {
    throw new Error("Something went wrong");
  }
  doSomething();
} catch (error) {
  console.error("Error caught:", error.message);
} finally {
  cleanup();
}
```

---

## Throw-Catch: Stack Unwinding

* When an exception is thrown, execution immediately jumps to the nearest catch block
* The call stack "unwinds" - all functions between throw and catch exit
* Local variables are cleaned up (destructors called in languages with explicit cleanup)
* Execution resumes at the catch block

```javascript
function outer() {
  try {
    middle();
  } catch (e) {
    // Execution jumps here, skipping the rest of middle() and inner()
    console.log("Caught:", e.message);
  }
}

function middle() {
  inner();
  console.log("This won't execute if inner() throws");
}

function inner() {
  throw new Error("Something failed");
}
```

---

## Problems with Exceptions

* Invisible control flow paths
* Function signatures don't indicate exception behavior (in JavaScript)
* Easy to forget error handling
* Performance overhead in some implementations
* Complicates reasoning about code execution paths

---

## Exceptions in Type Declarations (Java)

```java
// Method declares it may throw specific exceptions
public void readFile(String path) throws IOException, FileNotFoundException {
  // Method implementation
}

// Caller must handle or propagate exceptions
try {
  readFile("config.txt");
} catch (FileNotFoundException e) {
  System.err.println("Config file not found: " + e.getMessage());
} catch (IOException e) {
  System.err.println("Error reading config: " + e.getMessage());
}
```

---

## Algebraic Effects and Exceptions

* Algebraic effects are a generalization of exceptions
* Allow defining and handling computational effects
* More structured than simple throw/catch
* Enable resumable exceptions

```javascript
// Conceptual example (not actual JavaScript)
function divide(a, b) {
  if (b === 0) perform DivisionByZero();
  return a / b;
}

try {
  const result = divide(10, 0);
  console.log(result);
} handle DivisionByZero {
  resume with 1; // Continue execution with a default value
}
```

---

## Errors as Values

* Treating errors as regular return values
* Explicit error checking
* No special control flow mechanisms

---

## Node.js Callbacks Pattern

```javascript
// Traditional Node.js error-first callback pattern
fs.readFile('file.txt', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  
  // Process data if no error
  console.log(data.toString());
});
```

---

## Go's Error Handling

```go
// Go returns errors as values
func readConfig(path string) ([]byte, error) {
    data, err := ioutil.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("reading config: %w", err)
    }
    return data, nil
}

// Caller must check errors explicitly
data, err := readConfig("config.json")
if err != nil {
    log.Fatalf("Failed to read config: %v", err)
}
```

---

## Zig's Error Handling

```zig
// Zig has explicit error unions and catch
fn readValue() !u32 {
    const file = try std.fs.cwd().openFile("data.txt", .{});
    defer file.close();
    
    var buffer: [10]u8 = undefined;
    const size = try file.read(&buffer);
    
    return std.fmt.parseUnsigned(u32, buffer[0..size], 10);
}

// Usage with try/catch
fn main() !void {
    const value = readValue() catch |err| {
        std.debug.print("Error: {}\n", .{err});
        return;
    };
    std.debug.print("Value: {}\n", .{value});
}
```

---

## Promises and Rejections in JavaScript

```javascript
fetch('https://api.example.com/data')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    return response.json();
  })
  .then(data => console.log('Data:', data))
  .catch(error => console.error('Fetch error:', error));
```

---

## Promise.all vs Promise.allSettled

```javascript
// Promise.all - fails fast if any promise rejects
Promise.all([
  fetch('/api/users'),
  fetch('/api/products'),
  fetch('/api/orders')
])
.then(([usersRes, productsRes, ordersRes]) => {
  // All succeeded
})
.catch(error => {
  // At least one failed
});

// Promise.allSettled - waits for all promises regardless of outcome
Promise.allSettled([
  fetch('/api/users'),
  fetch('/api/products'),
  fetch('/api/orders')
])
.then(results => {
  // results is an array of objects with status 'fulfilled' or 'rejected'
  results.forEach(result => {
    if (result.status === 'fulfilled') {
      console.log('Success:', result.value);
    } else {
      console.log('Error:', result.reason);
    }
  });
});
```

---

## Async/Await in JavaScript

```javascript
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error('Error fetching user data:', error);
    throw error; // Re-throw or handle appropriately
  }
}

// Using the async function
async function displayUser(userId) {
  try {
    const user = await fetchUserData(userId);
    console.log('User:', user);
  } catch (error) {
    console.error('Failed to display user:', error);
  }
}
```

---

## Maybe Monad

* Represents a computation that might fail
* Encapsulates the presence or absence of a value
* Common in functional programming languages

---

## Rust's Result and Option Types

```rust
// Result<T, E> represents success (Ok) or failure (Err)
fn read_config_file(path: &str) -> Result<Config, io::Error> {
    let file = File::open(path)?; // ? operator propagates errors
    let config: Config = serde_json::from_reader(file)?;
    Ok(config)
}

// Option<T> represents Some(value) or None
fn find_user(id: UserId) -> Option<User> {
    if id.is_valid() {
        Some(User::new(id))
    } else {
        None
    }
}

// Using pattern matching
match read_config_file("config.json") {
    Ok(config) => println!("Config loaded: {}", config.name),
    Err(e) => eprintln!("Failed to load config: {}", e),
}
```

---

## OCaml's Error Handling

```ocaml
(* Using Result type *)
let read_file filename =
  try
    let channel = open_in filename in
    let content = really_input_string channel (in_channel_length channel) in
    close_in channel;
    Ok content
  with
  | Sys_error msg -> Error ("System error: " ^ msg)
  | _ -> Error "Unknown error while reading file"

(* Pattern matching on result *)
match read_file "data.txt" with
| Ok content -> Printf.printf "File content: %s\n" content
| Error msg -> Printf.eprintf "Error: %s\n" msg
```

---

## Error Handling in HTTP Request Handlers

```javascript
// Express.js with middleware for error handling
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const userId = req.params.id;
    const user = await database.findUser(userId);
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    // Pass to error-handling middleware
    next(error);
  }
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error('API Error:', err);
  res.status(500).json({ error: 'Internal server error' });
});
```

---

## Cascading Errors Across Service Boundaries

* Errors can propagate across multiple services
* Need to handle failures in dependent services
* Important to maintain context across boundaries

```javascript
async function getUserProfile(userId) {
  try {
    // Call user service
    const user = await userService.getUser(userId);
    
    // Call billing service
    const billing = await billingService.getBillingInfo(userId);
    
    // Call preferences service
    const preferences = await preferencesService.getPreferences(userId);
    
    return { user, billing, preferences };
  } catch (error) {
    if (error.service === 'billing') {
      // Handle billing service failure gracefully
      return { user, preferences, billing: { status: 'unavailable' } };
    }
    throw error; // Re-throw other errors
  }
}
```

---

## Circuit Breaker Pattern

* Prevents cascading failures
* Monitors for failures in external service calls
* Automatically opens circuit when failure threshold reached
* Gradually tests service recovery

```javascript
const breaker = new CircuitBreaker({
  service: 'payment-api',
  failureThreshold: 5,
  resetTimeout: 30000
});

async function processPayment(order) {
  try {
    return await breaker.execute(() => paymentApi.process(order));
  } catch (error) {
    if (error.type === 'CIRCUIT_OPEN') {
      // Circuit is open, use fallback
      return await fallbackPaymentProcess(order);
    }
    throw error;
  }
}
```

---

## Encapsulating Behavior in Error Objects

```javascript
class DatabaseError extends Error {
  constructor(message, query, code) {
    super(message);
    this.name = 'DatabaseError';
    this.query = query;
    this.code = code;
    this.timestamp = new Date();
  }
  
  // Self-reporting capability
  report() {
    console.error(`[${this.timestamp.toISOString()}] DB Error ${this.code}: ${this.message}`);
    console.error(`Query: ${this.query}`);
    
    // Send to monitoring system
    monitoring.captureException(this);
  }
  
  // Alert capability
  alert() {
    if (this.code >= 5000) {
      alerting.sendAlert({
        level: 'critical',
        service: 'database',
        message: this.message,
        details: this
      });
    }
  }
}

// Usage
try {
  // Database operation
} catch (error) {
  if (error instanceof DatabaseError) {
    error.report();
    error.alert();
  }
}
```

---

## Conclusion

* Different error handling approaches suit different scenarios
* Consider:
  * Developer experience
  * Performance requirements
  * Language idioms
  * System boundaries
  * Failure modes
* Best approach depends on your specific needs
