Python Coding Standards & Best Practices
This guide covers PEP 8 style rules and clean‐code principles for Python, with a focus on web apps and AI projects. It discusses naming conventions, formatting, comments/docstrings, writing maintainable functions/classes (SRP, DRY), common gotchas (mutable defaults, exceptions), project layout for web (e.g. Flask blueprints) and AI code, plus recommended linters/formatters.
PEP 8 Compliance
Indentation: Use 4 spaces per level (never tabs)
realpython.com
. Be consistent (Python 3 forbids mixing tabs and spaces).
Line Length: Limit code lines to ≈79 characters
realpython.com
. (Docstrings/comments to ~72 chars.) Use implicit line continuation in () or hanging indents rather than backslashes.
Blank Lines: Separate top-level class and def with two blank lines; methods inside a class with one blank line
peps.python.org
. Use extra blank lines sparingly to group related code.
Naming Conventions:
Modules/Packages: short, lowercase names (underscores if needed)
peps.python.org
.
Classes: CapWords (PascalCase)
peps.python.org
.
Functions/Variables: snake_case (lowercase with underscores)
peps.python.org
.
Constants: UPPER_SNAKE (CAPITALS with underscores)
peps.python.org
.
Use self and cls for instance/class method arguments. Use a trailing underscore (class_) if a name clashes with a Python keyword
peps.python.org
.
Whitespace: Surround operators with single spaces (a = b + c), but avoid extra spaces inside parentheses or brackets
testdriven.io
. Don’t leave trailing whitespace.
Imports: One per line (import os; from module import name), grouped (stdlib, third-party, local) with blank lines between groups
peps.python.org
.
Comments: Keep comments up-to-date and clear. Write full sentences with a capital letter (except code identifiers) and a period
peps.python.org
. Inline comments (# comment) should be separated by two spaces and used sparingly
peps.python.org
. Prefer explaining why code does something, not the obvious. Write comments in English.
Docstrings: Provide triple-quoted """Docstrings""" for all public modules, classes, functions/methods
peps.python.org
. Start with a one-line summary (closing """ on same line if brief), followed by details if needed (see PEP 257). For private methods you can use a brief comment.
Clean Code Principles
Single Responsibility (SRP): Each function or class should have one clear purpose (one reason to change)
realpython.com
dev.to
. Break complex tasks into smaller functions. This makes code easier to test and maintain. For example, don’t mix unrelated steps in one function – instead split them into load_data(), clean_data(), etc.
dev.to
.
Don’t Repeat Yourself (DRY): Avoid duplicating code. Common logic should be factored into a single function or class
testdriven.io
dev.to
. Duplicate code is hard to maintain (one fix should apply everywhere).
Modular and Readable: Use descriptive names and simple structures. Keep functions short and focused. Follow the “Zen of Python” (use import this) for guidance (e.g. “Readability counts”, “Simple is better than complex”). Write explicit, straightforward code (avoid one-liners that obscure logic).
Example – DRY principle: Instead of writing separate functions that differ only by literal strings, combine them with parameters
dev.to
:
# Bad: duplicated code
def print_user_details(name, age):
    print(f"Name: {name}")
    print(f"Age: {age}")

def print_product_details(product, price):
    print(f"Product: {product}")
    print(f"Price: {price}")

# Good: DRY – one function parameterizes the label
def print_details(label, value):
    print(f"{label}: {value}")
Here the “good” code uses one function for both cases, avoiding repetition
dev.to
. Example – Single Responsibility: Rather than one function doing multiple steps (bad), split into focused functions (good)
dev.to
:
# Bad: one function doing everything
def process_data(data):
    # Load data
    # Clean data
    # Analyze data
    # Save results
    pass

# Good: separate concerns into four functions
def load_data(path):
    pass

def clean_data(data):
    pass

def analyze_data(data):
    pass

def save_results(results):
    pass
Each function now has a single task (Load, Clean, Analyze, Save), improving clarity
dev.to
.
Common Python Gotchas
Mutable Default Arguments: Default parameter values are evaluated once at function definition, not each call. Avoid using mutable defaults like lists or dicts. Otherwise all calls share the same object
docs.python-guide.org
. Instead use None and inside the function do:
# Bad – mutable default
def append_item(x, lst=[]):
    lst.append(x)
    return lst

# Good – use None and create a new list each call
def append_item(x, lst=None):
    if lst is None:
        lst = []
    lst.append(x)
    return lst
In the bad version, successive calls without lst argument keep appending to the same list
docs.python-guide.org
.
Exception Handling: Always catch specific exceptions, not a bare except:, to avoid hiding bugs
medium.com
. As a rule, catch the smallest scope (e.g. except FileNotFoundError:) and handle it (log or notify), then allow others to propagate or re-raise
docs.python.org
. For example:
try:
    with open('data.txt', 'r') as file:
        contents = file.read()
except FileNotFoundError:
    print("File not found. Check the path.")
except Exception as e:
    print(f"An unexpected error occurred: {e}")
    raise
This catches a specific error and provides a user-friendly message, while not swallowing all exceptions
medium.com
. Use else: for code that should run only if no exception occurred, and finally: for cleanup. Logging exceptions (via the logging module) is recommended for debugging.
Web App Project Structure (e.g. Flask)
Package Layout: Organize the app in a package (e.g. app/) rather than one file. Common submodules include models/ (database models), routes/ or views/ (Flask view functions), and static/ & templates/ directories for assets and HTML. Put initialization code in app/__init__.py. See the example structure below (where main, posts, etc. are Blueprints):
myproject/
├── app/
│   ├── __init__.py      # create_app function, extension setup
│   ├── models/
│   │   └── user.py
│   ├── main/
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── posts/
│   │   ├── __init__.py
│   │   └── routes.py
│   └── templates/
│       ├── base.html
│       ├── index.html
│       └── posts/
│           └── index.html
├── config.py
└── run.py
Example: DigitalOcean’s Flask tutorial shows a similar structure using Blueprints, e.g. a users blueprint, posts blueprint, etc., each in its own folder
digitalocean.com
digitalocean.com
. This isolates related code and makes the project easy to navigate.
Blueprints: Use Flask Blueprints to group related routes and handlers (e.g., all user routes in a users Blueprint)
digitalocean.com
. Register blueprints in the app factory (create_app).
Views and Templates: Keep view functions (route handlers) lightweight – call services or models rather than embedding all logic in the route. Organize HTML templates in templates/, possibly with subfolders per blueprint.
App Factory: Use the application factory pattern (a create_app() function) to configure the app, initialize extensions (e.g. DB, login), and register blueprints. This makes testing and configuration easier.
AI Project Structure
Modular Design: Separate different concerns into modules. For example, put data loading/preprocessing in data/, model definitions in models/, training/evaluation routines in trainers/ or experiments/, and utility scripts in other modules. This mirrors standard software engineering practices applied to ML/AI.
Separate Model and Logic: Keep your AI/ML model code (neural network classes, training loops, etc.) separate from the application logic or interface code. This makes it easier to swap or update models. For instance, you might have a models/resnet.py and a separate app/agent.py that uses it.
Example Structure: One common pattern is the “Fantastic Four” AI modules – models/, trainers/, experiments/, and data/ directories
ai.plainenglish.io
. For example:
ai_project/
├── data/         # data processing and loading
├── models/       # model architectures (e.g. resnet.py)
├── trainers/     # training scripts or classes
├── experiments/  # configs and experiment runners
└── utils/        # helper functions
In this setup, each folder is a Python package (has __init__.py) and exposes clean interfaces (as shown in [33]
ai.plainenglish.io
). This makes code reusable and maintainable, and clearly separates data, model, and training code.
Configuration and Reproducibility: Keep hyperparameters and experiment configs in a separate file or arguments, not hard-coded. Use version control for code and consider tools like dvc or notebook documentation for datasets.
Linters and Formatters
Use automated tools to enforce style consistently:
Flake8: A popular linter that checks PEP 8 compliance, undefined names, and more
flake8.pycqa.org
. Run it (e.g. flake8 .) to catch errors before running code.
Black: An “opinionated” code formatter. It reformats entire files to a consistent style (a subset of PEP 8)
black.readthedocs.io
. Run black . to auto-format code (often integrated via pre-commit). It ensures uniform formatting with minimal configuration.
isort: A tool to sort and group imports alphabetically, split into sections (stdlib, third-party, local)
pycqa.github.io
. It plays well with Black (uses a compatible style). Running isort . will neatly order import statements, improving readability.
These tools can be integrated into your editor or CI pipeline. They help beginners follow best practices without manual effort (and make code reviews smoother by reducing style debates).
Summary: Adhering to PEP 8 and clean-code principles makes Python code more readable and maintainable. Use clear naming, consistent formatting, and small well-named functions. Avoid common pitfalls like mutable defaults and overly broad except. Organize web apps into logical packages/blueprints and AI code into modular components. Finally, automate style checks with linters (flake8) and formatters (Black/isort) to keep the codebase clean. Sources: Style and structure guidelines are based on the official PEP 8 and Python documentation
peps.python.org
peps.python.org
realpython.com
realpython.com
, community tutorials
digitalocean.com
ai.plainenglish.io
, and best-practice articles
dev.to
docs.python-guide.org
medium.com
. Each recommendation above is supported by these references.
Citations

How to Write Beautiful Python Code With PEP 8 – Real Python

https://realpython.com/python-pep8/

How to Write Beautiful Python Code With PEP 8 – Real Python

https://realpython.com/python-pep8/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/

Clean Code in Python | TestDriven.io

https://testdriven.io/blog/clean-code-python/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/
PEP 8 – Style Guide for Python Code | peps.python.org

https://peps.python.org/pep-0008/

SOLID Principles: Improve Object-Oriented Design in Python – Real Python

https://realpython.com/solid-principles-python/

Python Best Practices: Writing Clean, Efficient, and Maintainable Code - DEV Community

https://dev.to/devasservice/python-best-practices-writing-clean-efficient-and-maintainable-code-34bj

Clean Code in Python | TestDriven.io

https://testdriven.io/blog/clean-code-python/

Python Best Practices: Writing Clean, Efficient, and Maintainable Code - DEV Community

https://dev.to/devasservice/python-best-practices-writing-clean-efficient-and-maintainable-code-34bj

Python Best Practices: Writing Clean, Efficient, and Maintainable Code - DEV Community

https://dev.to/devasservice/python-best-practices-writing-clean-efficient-and-maintainable-code-34bj

Python Best Practices: Writing Clean, Efficient, and Maintainable Code - DEV Community

https://dev.to/devasservice/python-best-practices-writing-clean-efficient-and-maintainable-code-34bj

Common Gotchas — The Hitchhiker's Guide to Python

https://docs.python-guide.org/writing/gotchas/

5 Best Practices for Python Exception Handling | by Saad Jamil | Medium

https://medium.com/@saadjamilakhtar/5-best-practices-for-python-exception-handling-5e54b876a20
8. Errors and Exceptions — Python 3.13.5 documentation

https://docs.python.org/3/tutorial/errors.html

How To Structure a Large Flask Application with Flask Blueprints and Flask-SQLAlchemy | DigitalOcean

https://www.digitalocean.com/community/tutorials/how-to-structure-a-large-flask-application-with-flask-blueprints-and-flask-sqlalchemy

How To Structure a Large Flask Application with Flask Blueprints and Flask-SQLAlchemy | DigitalOcean

https://www.digitalocean.com/community/tutorials/how-to-structure-a-large-flask-application-with-flask-blueprints-and-flask-sqlalchemy

The Ultimate Deep Learning Project Structure: A Software Engineer’s Guide into the Land of AI | by Alee | Artificial Intelligence in Plain English

https://ai.plainenglish.io/the-ultimate-deep-learning-project-structure-a-software-engineers-guide-into-the-land-of-ai-c383f234fd2f?gi=6aeabfb16ad6

Flake8: Your Tool For Style Guide Enforcement — flake8 7.3.0 documentation

https://flake8.pycqa.org/en/latest/

The Black code style - Black 25.1.0 documentation

https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html