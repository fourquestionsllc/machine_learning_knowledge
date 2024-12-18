In Python, **exceptions** are handled using the `try`, `except`, `else`, and `finally` blocks. This mechanism allows you to catch errors, handle them gracefully, and ensure certain code runs regardless of an exception.

---

### **Basic Syntax for Exception Handling**

```python
try:
    # Code that may raise an exception
    result = 10 / 0
except ZeroDivisionError:  # Specific exception
    print("Error: Division by zero is not allowed.")
```

---

### **Key Blocks in Exception Handling**

1. **`try` block**: Code that might raise an exception is placed here.  
2. **`except` block**: Handles the exception if it occurs.  
3. **`else` block** *(optional)*: Executes only if no exception occurs.  
4. **`finally` block** *(optional)*: Runs regardless of whether an exception occurred or not.

---

### **Handling Specific Exceptions**

You can catch specific types of exceptions:

```python
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except ValueError:
    print("Error: Please enter a valid number.")
except ZeroDivisionError:
    print("Error: Division by zero is not allowed.")
```

---

### **Catching Multiple Exceptions**

To handle multiple exceptions in one block:

```python
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except (ValueError, ZeroDivisionError) as e:
    print(f"Error: {e}")
```

---

### **Using `else` and `finally`**

- **`else`**: Runs only if no exception is raised.  
- **`finally`**: Runs always, even if there’s an exception.

```python
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except ZeroDivisionError:
    print("Error: Division by zero.")
else:
    print(f"Result: {result}")
finally:
    print("Execution complete.")
```

**Output**:  
- If `num = 2`:  
  ```
  Result: 5.0
  Execution complete.
  ```
- If `num = 0`:  
  ```
  Error: Division by zero.
  Execution complete.
  ```

---

### **Raising Exceptions**

You can raise exceptions using the `raise` keyword.

```python
try:
    age = int(input("Enter your age: "))
    if age < 0:
        raise ValueError("Age cannot be negative.")
except ValueError as e:
    print(f"Error: {e}")
```

---

### **Creating Custom Exceptions**

You can define custom exceptions by subclassing the `Exception` class:

```python
class NegativeAgeError(Exception):
    pass

try:
    age = int(input("Enter your age: "))
    if age < 0:
        raise NegativeAgeError("Age cannot be negative!")
except NegativeAgeError as e:
    print(f"Custom Error: {e}")
```

---

### **Catching All Exceptions**

To catch all exceptions, use `except Exception`:

```python
try:
    result = 10 / 0
except Exception as e:
    print(f"An error occurred: {e}")
```

---

### **Best Practices**:
1. Catch specific exceptions whenever possible.  
2. Avoid using a bare `except` (e.g., `except:`) as it can hide unexpected errors.  
3. Use `finally` for cleanup actions (e.g., closing files or connections).  
4. Avoid overusing exceptions for normal program flow.  

---

This is the core of exception handling in Python. 
