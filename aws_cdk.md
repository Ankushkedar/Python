# Python Classes & AWS CDK — Concept Notes

General, reusable notes on Python object-oriented basics and how AWS CDK
builds on them. No project-specific details — just the concepts.

---

## Part 1 — Python OOP Basics

### Class vs Object

- A **class** is a blueprint / template. It's just instructions; nothing is built yet.
- An **object** (also called an *instance*) is a real thing created from that class.
- You can create many objects from one class.

**Analogy — ordering a cake:**
- Class = the recipe / order form (paper instructions, not a cake).
- Object = the actual baked cake.
- `__init__` = the questions on the order form (flavor? size? message?).
- Creating the object = placing the order, which triggers the baking.

```python
class Dog:                     # the class (blueprint)
    def __init__(self, name, age):
        self.name = name
        self.age = age

rex = Dog("Rex", 3)            # an object (instance) of class Dog
buddy = Dog("Buddy", 5)        # another, independent object
print(rex.name)                # -> "Rex"
```

### `def __init__(` — the constructor

- `def` starts a function/method definition.
- `__init__` is a special "dunder" (double-underscore) method: the **constructor**.
- It runs **automatically** when you create an object. You never call it by name.
- Its job is to set up the object's starting state.

The arguments you pass when creating the object line up with the parameters in `__init__`:

```python
rex = Dog("Rex", 3)
#          |     |
#          name  age   (self is supplied automatically)
```

### `self`

- `self` means "this particular object being built/used right now."
- Python passes it automatically — you never provide it yourself.
- Inside the class you use `self` to store and read the object's own data:

```python
class Counter:
    def __init__(self):
        self.count = 0          # data stored ON this object

    def increment(self):
        self.count += 1         # read/update this object's own data
```

### Methods and attributes

- **Attribute** = a piece of data stored on an object (`rex.name`).
- **Method** = a function that belongs to an object (`rex.bark()`).

```python
rex.name                 # attribute (data)
rex.increment()          # method (behavior)
```

### Inheritance and `super()`

- A class can **inherit** from another class, reusing its behavior.
- `ChildClass(ParentClass)` means "Child is a specialized Parent."
- `super().__init__(...)` calls the parent's constructor so the parent's setup runs too.

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):                 # Dog inherits from Animal
    def __init__(self, name, breed):
        super().__init__(name)     # run Animal's constructor first
        self.breed = breed
```

### Keyword-only arguments (`*`)

- A bare `*` in a parameter list forces everything after it to be passed **by keyword**.
- This prevents mixing up many same-typed arguments by position.

```python
def make_user(name, *, email, is_admin=False):
    ...

make_user("Alice", email="a@x.com", is_admin=True)   # OK
make_user("Alice", "a@x.com")                        # ERROR: email must be keyword
```

### `*args` and `**kwargs`

- `*args` collects extra **positional** arguments into a tuple.
- `**kwargs` collects extra **keyword** arguments into a dict.
- Commonly forwarded to a parent class: `super().__init__(**kwargs)`.

---

## Part 2 — AWS CDK Concepts

AWS CDK (Cloud Development Kit) lets you define cloud infrastructure using a real
programming language (here, Python). It is just a Python library, so all the OOP
rules above apply.

### Constructs and the construct tree

- CDK models infrastructure as a **tree of constructs**.
- A **construct** is any building block (an app, a stack, a single resource).
- Each construct has a **parent** (its *scope*) and a **name** (its *construct ID*).

```
App                 <- root of the tree
 └── Stack
      ├── Resource A
      ├── Resource B
      └── ...
```

### `scope` and `construct_id`

Every construct is created with two standard first arguments:

```python
SomeResource(scope, construct_id, ...)
```

- `scope` = the **parent** construct (who this hangs off of in the tree).
- `construct_id` = a **name unique among siblings** under that scope.

Together they give each construct a unique "address" in the tree, like a folder path
plus a filename. CDK uses this address to generate CloudFormation **logical IDs**.

> Important: `construct_id` is the CDK *tree name*, which is usually **different** from
> the real AWS resource name (e.g. a function's `function_name`). Changing a
> `construct_id` can force CloudFormation to replace the resource, so keep IDs stable.

### `app = cdk.App()`

- Creates the **root** of the construct tree.
- Nothing is deployed yet — you are only declaring objects in memory.

```python
app = cdk.App()
MyStack(app, "MyStack")      # app is the scope; "MyStack" is the construct_id
```

### `app.synth()`

- **Synthesizes** (generates) the deployable **CloudFormation template** and assets.
- Walks the whole tree, turns constructs into real cloud resources on paper, and writes
  the output to a cloud-assembly folder (by default `cdk.out/`).
- Deployment happens afterward when the CDK CLI reads that output.

```python
app = cdk.App()      # 1. build the tree in memory
MyStack(app, "...")  # 2. populate the tree
app.synth()          # 3. emit CloudFormation + assets to disk
```

### A typical stack class

```python
class MyStack(cdk.Stack):
    def __init__(self, scope, construct_id, *, some_setting, **kwargs):
        super().__init__(scope, construct_id, **kwargs)   # register in the tree first

        # create resources; pass `self` as their scope so they live inside this stack
        fn = lambda_.Function(self, "MyLambda", ...)
        fn.add_to_role_policy(...)                        # method call on the object
```

### The one CDK twist to remember

In plain Python, creating an object just holds data in memory. In CDK, **creating an
object with a `scope` also registers it into the construct tree**. So this single line:

```python
fn = lambda_.Function(self, "MyLambda", ...)
```

does two things: (1) makes a `Function` object, and (2) attaches it to the stack so it
becomes a real resource at synth time. You never have to "add" it anywhere — passing
`self` as the scope did that.

---

## Part 3 — Environment Variables (`os.environ`)

- Environment variables are key/value settings that live **outside** your code, in the
  environment a program runs in (your shell, a CI job, or a cloud runtime).
- In Python you read them with the `os` module:

```python
import os

# Raises KeyError if the variable is not set (use when it is REQUIRED):
secret_name = os.environ["SECRET_NAME"]

# Returns a default if not set (use when it is OPTIONAL):
region = os.environ.get("AWS_REGION", "eu-central-1")
```

- The same `os.environ` call reads **different values depending on where the code runs**.
  A script on your laptop reads your shell's variables; code running in a cloud function
  reads the variables configured on that function.
- Common pattern: configuration/tooling code reads env vars from your shell or CI, then
  passes those values forward so that deployed code later reads them from its own runtime
  environment.

---

## Quick reference

| Term | Meaning |
|------|---------|
| Class | Blueprint/template for objects. |
| Object / instance | A concrete thing built from a class. |
| `__init__` | Constructor; runs automatically when an object is created. |
| `self` | The specific object being built/used; supplied automatically. |
| Attribute | Data stored on an object (`obj.x`). |
| Method | Function that belongs to an object (`obj.do()`). |
| Inheritance | A class reusing/extending another class. |
| `super().__init__()` | Calls the parent class's constructor. |
| `*` (bare) | Forces following args to be keyword-only. |
| `**kwargs` | Collects extra keyword arguments into a dict. |
| Construct (CDK) | A building block in the infrastructure tree. |
| `scope` (CDK) | The parent construct. |
| `construct_id` (CDK) | A construct's name, unique among its siblings. |
| `cdk.App()` | Root of the CDK construct tree. |
| `app.synth()` | Generates the CloudFormation template + assets. |
| `os.environ[...]` | Reads a required environment variable. |
| `os.environ.get(...)` | Reads an optional environment variable with a default. |
