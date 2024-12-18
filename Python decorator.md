In Python, a **decorator** is a design pattern that allows you to **modify or extend the behavior of a function or a class method** without changing its actual code. Decorators are often used to add functionality, such as logging, validation, or performance measurement, in a clean and reusable way.

---

### **How Decorators Work**

1. A **decorator** is a callable (function) that takes another function as an argument and usually returns a modified or wrapped version of that function.  
2. It allows you to "wrap" functionality around an existing function, making it easy to reuse and apply without altering the original function.  
3. Decorators are applied using the **`@decorator_name`** syntax above a function definition.

---

### **Syntax of a Decorator**

Here’s a simple example:

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

**Output**:  
```
Something is happening before the function is called.
Hello!
Something is happening after the function is called.
```

---

### **Explanation**:
1. **`my_decorator`**: This is a decorator function that takes `func` as its argument.  
2. **`wrapper`**: The inner function adds behavior before and after calling the original function `func`.  
3. **`@my_decorator`**: This applies the decorator to the `say_hello()` function.  
4. When you call `say_hello()`, it executes the `wrapper` function defined in the decorator, wrapping the additional behavior around the original function.

---

### **Common Use Cases of Decorators**

1. **Logging**: Add logs before/after a function runs.  
2. **Authentication**: Check if a user is authenticated before accessing a resource.  
3. **Caching**: Cache the results of expensive function calls.  
4. **Timing**: Measure the execution time of functions.  
5. **Validation**: Validate inputs to a function.

---

### **Decorator with Arguments**
If your function takes arguments, you need to modify the decorator to accept `*args` and `**kwargs`:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def add(a, b):
    return a + b

print(add(2, 3))
```

**Output**:  
```
Before function call
After function call
5
```

---

### **Built-in Decorators in Python**  
Python provides some built-in decorators:  
- `@staticmethod`  
- `@classmethod`  
- `@property`

Example with `@staticmethod`:

```python
class MyClass:
    @staticmethod
    def greet():
        print("Hello!")

MyClass.greet()
```

---

### **Key Points**:
- Decorators add behavior to functions or methods.  
- They use the `@decorator_name` syntax.  
- They are powerful for writing clean, reusable, and modular code.  
- Inner functions (`wrapper`) use `*args` and `**kwargs` to handle dynamic inputs.
