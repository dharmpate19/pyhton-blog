# FastAPI Learning Notes

## 1. uv

### What is uv?

`uv` is a fast Python package and project manager.

It can handle many tasks that we traditionally use tools such as:

* `pip` → installing Python packages
* `venv` → creating virtual environments
* `pip-tools` → managing dependencies
* Python version/project management

The main reason we are using `uv` in this project is that it makes creating and managing a Python project and its environment much easier and faster.

---

### Why do we need an environment?

Suppose we have two Python projects:

**Project A**

* FastAPI version X
* Pydantic version X

**Project B**

* FastAPI version Y
* Pydantic version Y

If both projects use the same global Python environment, their dependencies can conflict.

For example:

```text
Project A
FastAPI 0.x
       ↓
needs Pydantic version A

Project B
FastAPI 1.x
       ↓
needs Pydantic version B
```

Installing everything globally can cause one project to break when another project needs a different package version.

A **virtual environment** gives each project its own isolated environment.

```text
Computer
│
├── Project A
│   └── .venv
│       ├── FastAPI version A
│       └── Pydantic version A
│
└── Project B
    └── .venv
        ├── FastAPI version B
        └── Pydantic version B
```

So each project can have its own dependencies without interfering with other projects.

---

### What does "creating an environment" mean?

When we create a Python virtual environment, we are creating an isolated Python environment for that project.

For example:

```bash
uv venv
```

This creates a `.venv` directory:

```text
my-fastapi-project/
│
├── .venv/
└── ...
```

The `.venv` contains the environment that the project will use.

It is **not a completely separate operating system or computer**.

It is an isolated Python environment with its own Python executable and installed packages.

---

### Why use uv instead of manually using venv + pip?

Traditional approach:

```bash
python -m venv .venv
```

Then activate the environment and use:

```bash
pip install fastapi
```

With `uv`, project management becomes simpler and faster.

For example:

```bash
uv venv
```

and packages can be installed using:

```bash
uv add fastapi
```

`uv` also keeps track of project dependencies and can create/update the project's lock information.

---

### Important idea

Think of it like this:

```text
Python
  ↓
Virtual Environment
  ↓
Project Dependencies
  ↓
FastAPI Application
```

The virtual environment isolates the dependencies of our project from the rest of the system.

---

### Commands we will use

Create a project:

```bash
uv init
```

Create a virtual environment:

```bash
uv venv
```

Add a package:

```bash
uv add fastapi
```

Run a command inside the project environment:

```bash
uv run <command>
```

---

### Remember

**uv = Python project and package manager**

**`.venv` = isolated Python environment for the project**

**Virtual environment = prevents project dependencies from conflicting with each other**

So:

```text
uv
 ↓
manages the Python project
 ↓
creates/manages the environment
 ↓
manages dependencies
 ↓
runs the application
```

Pydantic Sqlalchemy
Pydantic checks and structures API request/response data. SQLAlchemy model defines how Python data maps to the database. The SQLAlchemy model is not the API's data-validation layer.


# Pydantic Schema vs SQLAlchemy Model

## 1. Schema — Pydantic

A **Pydantic schema** defines the structure and rules for data that comes **into or goes out of an API**.

It acts as an **API contract**.

```python
from pydantic import BaseModel

class PostCreate(BaseModel):
    title: str
    content: str
```

If a client sends:

```json
{
    "title": "FastAPI",
    "content": "Learning schemas"
}
```

FastAPI/Pydantic uses `PostCreate` to:

* Check whether the required fields are present.
* Check whether the values have the expected types/rules.
* Convert compatible input into the appropriate Python types when possible.
* Create a Pydantic object that your Python code can work with.

For example:

```python
post.title
post.content
```

### Schema can also be used for responses

A schema is not only for request data.

You can define a response schema:

```python
class PostResponse(BaseModel):
    id: int
    title: str
    content: str
```

This tells FastAPI what the API response should look like.

Therefore:

```text
Schema = API data structure + validation
```

---

# 2. Model — SQLAlchemy

A **SQLAlchemy model** represents a **database table in Python**.

It defines how Python objects and their attributes correspond to database tables and columns.

```python
from sqlalchemy import Column, Integer, String

class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String)
    content = Column(String)
```

This tells SQLAlchemy:

```text
Python Post object
       ↓
       maps to
       ↓
Database "posts" table

id       → id column
title    → title column
content  → content column
```

The model is mainly responsible for **mapping Python objects to database records** and allowing SQLAlchemy to generate/execute database operations.

For example:

```python
post = Post(
    title="FastAPI",
    content="Learning SQLAlchemy"
)

db.add(post)
db.commit()
```

SQLAlchemy uses the model's mapping to know how this object corresponds to a row in the `posts` table.

---

# 3. Schema vs Model

The easiest way to understand the difference is:

```text
             API
              ↕
       Pydantic Schema
              ↕
           Python
              ↕
      SQLAlchemy Model
              ↕
          Database
```

### Pydantic Schema

Answers:

> **"What data should the API accept or return, and does it satisfy the API's schema?"**

### SQLAlchemy Model

Answers:

> **"How does this Python object correspond to the database table and its columns?"**

---

# 4. Complete Example

Suppose the client wants to create a post.

### Step 1 — Client sends request

```json
{
    "title": "FastAPI",
    "content": "Learning FastAPI"
}
```

### Step 2 — Pydantic validates the request

```python
class PostCreate(BaseModel):
    title: str
    content: str
```

FastAPI receives the request and uses `PostCreate`.

```text
JSON request
     ↓
PostCreate
     ↓
Pydantic validation
     ↓
Pydantic object
```

Now Python can work with:

```python
post.title
post.content
```

---

### Step 3 — Create SQLAlchemy object

You can then use the validated data to create a SQLAlchemy model object:

```python
db_post = Post(
    title=post.title,
    content=post.content
)
```

Now:

```text
Pydantic object
      ↓
validated API data
      ↓
SQLAlchemy Post object
      ↓
database mapping
      ↓
posts table
```

---

# 5. Important: The Model Is Not the Same as API Validation

Don't think:

> "SQLAlchemy model checks whether the API request is correct."

That's primarily the job of **Pydantic** in a FastAPI application.

Instead:

```text
Pydantic Schema
→ validates/structures API data

SQLAlchemy Model
→ maps Python objects to database tables
```

However, SQLAlchemy models can define **database-related constraints**, such as:

```python
title = Column(String, nullable=False, unique=True)
```

These constraints describe requirements for the database and can result in database/ORM errors if violated.

That is different from using Pydantic to validate an incoming API request.

---

# 6. One-Line Memory Trick

> **Schema = API contract**

> **Model = Database mapping**

Or even simpler:

```text
Pydantic Schema → API ↔ Python

SQLAlchemy Model → Python ↔ Database
```

So in a typical FastAPI application:

```text
Client
  ↓
JSON
  ↓
Pydantic Schema
  ↓
Validated Python data
  ↓
SQLAlchemy Model
  ↓
Database
```

And for a response:

```text
Database
  ↓
SQLAlchemy Model
  ↓
Python data
  ↓
Pydantic Response Schema
  ↓
JSON response
  ↓
Client
```

Engine connects SQLAlchemy to the database.
Session performs database operations.
Model defines the database mapping.
Schema handles API data.


When a request reaches a route that needs database access, FastAPI creates a SQLAlchemy Session through get_db(). The Session allows the route to perform database operations, using the Engine to communicate with the database. After the request is finished, the Session is closed.


Model
 ↓
"what does this Python object represent?"
 ↓
Session
 ↓
"manage what I want to do with this object"
 ↓
Engine
 ↓
"manage the database connection"
 ↓
Database