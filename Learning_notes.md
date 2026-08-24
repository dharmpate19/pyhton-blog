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
