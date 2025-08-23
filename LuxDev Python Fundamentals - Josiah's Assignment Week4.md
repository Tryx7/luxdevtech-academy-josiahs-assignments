 

## Python Fundamentals Exercises 

**Q1. VARIABLES EXERCISES**

```
# 1. Declare a variable named age and assign your age to it

age = 25
print(f"My age is: {age}")

# 2. Create two variables x = 10 and y = 5. Print their sum

x = 10
y = 5
print(f"Sum of x and y: {x + y}")

# 3. Try using an invalid variable name like 2ndName and observe the error

2ndName = "John"  # This will cause SyntaxError: invalid decimal literal

# 4. Assign a string to a variable and print it

message = "Hello, Python!"
print(message)
```

 **Q2. DATA TYPES EXERCISES**

```
# 1. Print the type of 42, 3.14, and 'hello'

print(f"Type of 42: {type(42)}")
print(f"Type of 3.14: {type(3.14)}")
print(f"Type of 'hello': {type('hello')}")

# 2. Convert a string '100' to an integer

number_str = '100'
number_int = int(number_str)
print(f"Converted integer: {number_int}, type: {type(number_int)}")

# 3. Add an integer and a float together

result = 10 + 3.5
print(f"10 + 3.5 = {result}, type: {type(result)}")  # Result is float

# 4. Multiply a string by a number

repeated_string = "hello" * 3
print(f"'hello' * 3 = '{repeated_string}'")  # Repeats the string
```

**Q3. DATA STRUCTURES EXERCISES**

```
# 1. Create a list of 5 fruits and print the third fruit

fruits = ["apple", "banana", "orange", "grape", "mango"]
print(f"Third fruit: {fruits[2]}")  # Index 2 is third element

# 2. Create a dictionary with keys: name, age

person = {"name": "Alice", "age": 30}
print(f"Age: {person['age']}")

# 3. Define a tuple with three numbers. Try modifying it

numbers_tuple = (1, 2, 3)
print(f"Original tuple: {numbers_tuple}")
# numbers_tuple[0] = 10  # This will cause TypeError: 'tuple' object does not support item assignment

# 4. Create a set from a list with duplicate values

duplicate_list = [1, 2, 2, 3, 4, 4, 5]
unique_set = set(duplicate_list)
print(f"List with duplicates: {duplicate_list}")
print(f"Set (unique values): {unique_set}")
```


**Q4. LOOPS EXERCISES** 

```
# 1. Use a for loop to print numbers from 1 to 10

print("Numbers 1 to 10:")
for i in range(1, 11):
    print(i, end=" ")
print()

# 2. Use a while loop to print numbers until the user enters stop

print("Enter numbers (type 'stop' to end):")
while True:
    user_input = input("Enter a number: ")
    if user_input.lower() == 'stop':
        break
    print(f"You entered: {user_input}")

# 3. Write a loop that prints even numbers from 1 to 20

print("Even numbers from 1 to 20:")
for num in range(2, 21, 2):
    print(num, end=" ")
print()

# 4. Explain break and continue:

# break: Immediately exits the current loop
# continue: Skips the rest of current iteration and moves to next iteration
```

**Q5. CONTROL FLOWS EXERCISES**

```
# 1. Check if a number is positive, negative, or zero

def check_number(num):
    if num > 0:
        return "positive"
    elif num < 0:
        return "negative"
    else:
        return "zero"

print(f"5 is {check_number(5)}")
print(f"-3 is {check_number(-3)}")
print(f"0 is {check_number(0)}")

# 2. Check voting eligibility

def check_voting_eligibility(age):
    if age >= 18:
        return "Eligible to vote"
    else:
        return "Not eligible to vote"

print(f"Age 20: {check_voting_eligibility(20)}")
print(f"Age 16: {check_voting_eligibility(16)}")

# 3. Find the largest of 3 numbers

def find_largest(a, b, c):
    if a >= b and a >= c:
        return a
    elif b >= a and b >= c:
        return b
    else:
        return c

print(f"Largest of (10, 25, 15): {find_largest(10, 25, 15)}")

# 4. Practice and, or, not

x = 10
y = 20
z = 30

print(f"x > 5 and y < 25: {x > 5 and y < 25}")  # True
print(f"x > 15 or y < 25: {x > 15 or y < 25}")  # True
print(f"not (x > 15): {not (x > 15)}")  # True
```


**Q6. FUNCTIONS EXERCISES**

```
# 1. Function greet(name) that prints "Hello, [name]"

def greet(name):
    print(f"Hello, {name}!")

greet("John")

# 2. Function add(a, b) that returns the sum

def add(a, b):
    return a + b

result = add(5, 3)
print(f"5 + 3 = {result}")

# 3. Modify add() to print "even" or "odd" based on the result

def add_with_even_odd(a, b):
    result = a + b
    if result % 2 == 0:
        print(f"{result} is even")
    else:
        print(f"{result} is odd")
    return result

add_with_even_odd(4, 5)  # 9 is odd
add_with_even_odd(3, 3)  # 6 is even

# 4. Call a function from within another function

def calculate_total(a, b, c):
    def add_two_numbers(x, y):
        return x + y
    
    partial_sum = add_two_numbers(a, b)
    total = add_two_numbers(partial_sum, c)
    return total

print(f"Total of 1, 2, 3: {calculate_total(1, 2, 3)}")
```

**BONUS: ADDITIONAL PRACTICE**

```
print("\n=== BONUS PRACTICE ===")

 Combining everything: A simple calculator
def calculator():
    print("Simple Calculator")
    print("Available operations: +, -, *, /")
    
     Get user input (simulated)
    operations = [
        (10, 5, '+'),
        (15, 3, '-'),
        (4, 7, '*'),
        (20, 4, '/')
    ]
    
    for num1, num2, op in operations:
        if op == '+':
            result = num1 + num2
        elif op == '-':
            result = num1 - num2
        elif op == '*':
            result = num1 * num2
        elif op == '/':
            if num2 != 0:
                result = num1 / num2
            else:
                result = "Error: Division by zero"
        else:
            result = "Error: Invalid operation"
        
        print(f"{num1} {op} {num2} = {result}")

calculator()
```

## **EXERCISE SOLUTIONS COMPLETE**

**Key Learning Points**


**Variables:** Store data, follow naming rules

**Data Types:** int, float, str, bool - Python handles conversions

**Data Structures:** Lists (mutable), Tuples (immutable), Dicts (key-value), Sets (unique)

**Loops:** for (definite), while (indefinite), break (exit), continue (skip)

**Control Flow:** if/elif/else with logical operators

**Functions:** Reusable code blocks, can call other functions





