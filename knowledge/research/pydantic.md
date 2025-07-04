Windsurf: Pydantic Best Practices and Guide
Introduction
Windsurf is a project aimed at helping beginner to intermediate Python developers harness the power of Pydantic for data validation and settings management. Pydantic is a popular library that uses Python type hints to validate and parse data into Python objects. This README provides a comprehensive guide to using Pydantic effectively, with an emphasis on simplicity and clarity. We assume you are using the latest Pydantic v2.x and highlight differences from Pydantic v1.x along the way. Whether you're writing standalone scripts, building web APIs with FastAPI, or integrating with AI frameworks like Agno, this guide will walk you through best practices, common gotchas, and recommended patterns for Pydantic.
Getting Started with Pydantic
What is Pydantic? Pydantic allows you to define data models (similar to Python's dataclasses) with type annotations, and it will automatically validate and convert input data to those types. If data doesn't conform to the types or specified constraints, Pydantic raises clear errors. This makes it invaluable for applications where you parse JSON or user input (e.g., API requests or config files) and want to ensure the data is correct and well-structured. Installation: Pydantic can be installed via pip:
pip install pydantic
This will install Pydantic v2 (current production release). Pydantic v2 is significantly faster than v1 (4x to 50x faster in benchmarks) thanks to a complete rewrite of its core in Rust
medium.com
. It also introduces a "strict mode" for stricter type validation and clearer distinctions between required and optional fields
medium.com
. Basic Example: Let's start with a simple example of using Pydantic in a standalone Python script. We'll define a Pydantic model and see how it validates input data:
from pydantic import BaseModel, Field, ValidationError

# Define a data model using Pydantic
class User(BaseModel):
    name: str
    age: int = Field(..., ge=0, description="Age must be non-negative")  # '...' means required
    email: str  # will validate as string; we can use EmailStr for stricter email validation
    is_active: bool = True  # default value provided

# Valid input example
data = {"name": "Alice", "age": 30, "email": "alice@example.com"}
user = User(**data)  # Pydantic parses the dict into a User object
print(user)  
#> name='Alice' age=30 email='alice@example.com' is_active=True

# Invalid input example (age is negative and email is missing)
bad_data = {"name": "Bob", "age": -5}
try:
    User(**bad_data)
except ValidationError as e:
    print(e)
    '''
    2 validation errors for User
    age
      Value error, Age must be non-negative (type=value_error) 【...†L...】
    email
      Field required (type=value_error.missing)
    '''
In this example, we defined a User model with four fields. Pydantic automatically enforced that:
age must be an integer and non-negative (we set ge=0 which means greater or equal to 0). The special value ... in Field(..., ...) marks age as required (no default).
email is required (no default given), and must be a string (we could use a stricter type for email, shown later).
name is required and must be a string.
is_active has a default value True, so it's optional in input (if not provided, it defaults to True).
Running this, the first User instantiation succeeds. The second one raises a ValidationError because age was negative and email was missing; Pydantic provides detailed error messages pinpointing each issue. In a real application, you might catch these errors to return a friendly message or handle invalid input gracefully.
Defining Pydantic Models and Validation
Pydantic models are Python classes that inherit from BaseModel. Each class attribute represents a field in the model with an associated type annotation. Here's how to define models and some best practices:
Basic Field Types: Use standard Python types (str, int, float, bool, list, dict, etc.) or typing constructs (List[int], Optional[str], etc.) to declare fields. Pydantic will validate and convert incoming data to these types when possible.
Field Constraints: Pydantic provides a Field() function to add metadata or validation constraints to a field (e.g., min/max values, regex patterns, descriptions). You can also use Pydantic's built-in types for common constraints (like EmailStr for emails, HttpUrl for URLs, PositiveInt for positive integers, etc.) which validate format automatically.
Defaults and Required Fields: If a field has a default value (or default factory), it becomes optional (not required in input). If no default is provided, the field is required (Pydantic will error if missing). Use Field(..., ...) or simply no default to indicate a required field. If you want a field to be optional (nullable), use Optional[type] and provide a default of None or use Field(default=None); otherwise, Pydantic will still consider it required
medium.com
. For example, price: Optional[float] = None means the price field can be omitted or set to None, whereas price: Optional[float] with no default would still trigger "field required" if not provided.
Below is a more detailed example demonstrating various field definitions:
from typing import Optional, List
from pydantic import BaseModel, Field, EmailStr, PositiveInt

class Product(BaseModel):
    name: str  # required string
    price: float = Field(..., gt=0, description="Price must be > 0")  # required float with constraint
    quantity: int = Field(0, ge=0)  # optional int with default 0 (non-negative)
    tags: List[str] = []  # optional list of strings, defaults to empty list
    description: Optional[str] = None  # optional string, default None (can be omitted)
    seller_email: EmailStr  # required email, will validate format automatically

# Create an instance with valid data
item = Product(
    name="Laptop", 
    price=999.99, 
    quantity=5, 
    tags=["electronics", "computer"], 
    seller_email="sales@shop.com"
)
print(item.dict())  # .dict() is legacy in v2, but still works; prefer .model_dump()
#> {'name': 'Laptop', 'price': 999.99, 'quantity': 5, 'tags': ['electronics', 'computer'], 
#>  'description': None, 'seller_email': 'sales@shop.com'}

# Demonstrate validation:
try:
    Product(name="Table", price=-10, seller_email="not-an-email")
except ValidationError as e:
    print(e)
    '''
    2 validation errors for Product
    price
      Input should be greater than 0 [type=greater_than, input_value=-10, input_type=int] 
    seller_email
      Input should be a valid email [type=value_error.email] :contentReference[oaicite:3]{index=3}
    '''
In this Product model:
name: required string.
price: required float, must be greater than 0 (gt=0).
quantity: optional int, defaults to 0 (and cannot be negative due to ge=0).
tags: optional list of strings, default []. Each Product instance gets its own copy of the list. Pydantic handles mutable default values by internally deep copying them, so you won't get shared list issues across instances
stackoverflow.com
. For example, if you append to item.tags, other instances won't see that change.
description: optional string, default None.
seller_email: required email address string. We used EmailStr (from pydantic[email] extra) which will automatically validate the email format
datacamp.com
.
This example also shows how Pydantic error messages reference the field and the nature of the error (like "greater_than" for price, and a specific email format error).
Required vs Optional Fields
One common confusion is the difference between "required field" and "field that can be null/None". In Pydantic (both v1 and v2):
If a field has no default value, it is required. Even if the type is Optional[...], you must supply it (or explicitly set default to None). In code, age: Optional[int] still means Pydantic expects an age key unless you do age: Optional[int] = None.
To truly make a field optional (in the sense that it can be omitted), give it a default value (e.g., = None or some default). If you want to allow None as a value, use an optional type and default None.
Required fields with a type that allows None (Optional) but no default will be treated as required. If omitted, you'll get a "field required" error, not automatically assumed to be None.
In summary: Optional in the type hint means the value can be null, but you still need to provide it unless a default is set. Always provide a default (like Field(default=None)) for truly optional fields
medium.com
.
Field Constraints and Built-in Types
Pydantic's Field() helps enforce constraints without writing custom logic:
gt / ge for greater-than / greater-or-equal (and lt / le for less-than constraints on numbers).
min_length / max_length for string lengths.
regex or pattern for pattern matching on strings.
description and example for documentation purposes (useful in FastAPI to auto-document).
Moreover, Pydantic offers specialized types:
EmailStr: Validates that a string is a valid email format.
AnyUrl/HttpUrl: Validates URL strings.
conint, constr, conlist: Factory functions to construct "constrained int/str/list" types for advanced constraints (like constr(min_length=1, regex=r'^[A-Z]') for a non-empty string starting with capital letter).
DateTime, UUID, Decimal: Pydantic can automatically parse standard library types. If you expect a datetime string, just use datetime type; Pydantic will attempt to parse ISO timestamps into actual datetime objects.
Extra Types: Pydantic v2 moved some less-common types (like Color, PaymentCardNumber) to a separate pydantic-extra-types package
docs.pydantic.dev
docs.pydantic.dev
. If you need them, install that package or use alternatives.
Using these built-ins is preferred over manual regex or custom validation when available, as they are well-tested and clearly communicate intent.
Nested Models and Complex Types
Pydantic models can be nested within each other. For example, you might have an Address model and a User model that contains an address: Address field. Pydantic will automatically validate nested models. You simply use the model class as the type of the field:
class Address(BaseModel):
    city: str
    zip: str

class UserProfile(BaseModel):
    username: str
    address: Address  # nested model
    tags: dict[str, int]  # a dict field, keys are str and values are int

profile = UserProfile(username="jdoe", address={"city": "Paris", "zip": "75001"}, tags={"posts": 42})
print(profile.address.city)  # "Paris"
print(profile.model_dump())  # nested model is dumped as a dict
Here, we passed a dict for address and Pydantic converted it into an Address object for us. If the nested data was invalid (e.g., missing a city), the ValidationError would cascade indicating address.city is required, etc. This makes it easy to compose complex schemas. Gotcha: In Pydantic v2, if a field is declared as a simple type like dict, Pydantic will no longer automatically convert a passed BaseModel into a dict as it did in v1
stackoverflow.com
stackoverflow.com
. For example, if tags: dict and you pass another BaseModel instance, v1 might have implicitly used its .dict(), but v2 expects an actual dict. If you have to accept either a model or dict, you should handle conversion (perhaps with a validator or using Annotated with a pre-validator as in the Stack Overflow solution
stackoverflow.com
). The better approach is to type the field to the specific model if you expect that (e.g., pet: Pet instead of pet: dict). In general, being explicit with types avoids ambiguity.
Custom Validation (Validators)
While field constraints cover many common cases, sometimes you need custom logic. Pydantic allows you to define validator methods to enforce more complex rules or to transform data.
Field Validators (single-field)
In Pydantic v1, you used @validator on model classes to create validation logic for fields. In Pydantic v2, this has been replaced by @field_validator, and the older @validator is deprecated
docs.pydantic.dev
. The new approach is very similar but offers more control (like pre- and post-validation). Here's how to use it:
from pydantic import field_validator

class User(BaseModel):
    username: str
    password: str

    # Ensure username is all lowercase
    @field_validator("username")
    def username_lowercase(cls, v):
        return v.lower()

    # Custom validation on password
    @field_validator("password")
    def strong_password(cls, v):
        if len(v) < 8:
            raise ValueError("Password must be at least 8 characters long")
        if v.isnumeric():
            raise ValueError("Password should not be all numbers")
        return v
In this example:
The username_lowercase validator will run after standard validation for username and force the value to lowercase (note: if you want to run a validator before built-in validation/coercion, use @field_validator(..., mode="before")).
The strong_password validator checks the password length and content, raising ValueError with an explanation if it doesn't meet criteria. Pydantic will include this message in the error output.
All validators should either return the value (possibly transformed) or raise an exception. If an exception is raised, Pydantic will report a validation error for that field. Multiple Field Validators: Sometimes validation of one field may depend on another. In Pydantic v2, you can use @model_validator for cross-field validation. (In v1, this was done with @root_validator). For example, let's ensure that either an email or a phone is provided in a contact form model:
from pydantic import model_validator

class Contact(BaseModel):
    email: Optional[str] = None
    phone: Optional[str] = None

    # After all fields are validated, ensure at least one of email/phone is given
    @model_validator(mode="after")
    def check_one_method(cls, self):
        if not (self.email or self.phone):
            raise ValueError("At least one of email or phone must be provided")
        return self
Here mode="after" means the validator runs after the whole model is validated (i.e., after individual fields are processed). We access self (the model instance) and impose a custom rule. If the rule fails, raising ValueError marks the model as invalid. You could also use mode="before" and take values: dict if you need to validate prior to field parsing. Tip: Use @field_validator for checks on individual fields and simple dependent checks (since it can access others via cls.model_fields if needed), and @model_validator when you need the whole model built (e.g., to set one field's default based on another, or ensure consistency among multiple fields). The example below shows using a model validator to set a default based on another field (a pattern for dependent defaults):
class Product(BaseModel):
    name: str
    category: str
    price: float
    discount: Optional[float] = None

    @model_validator(mode="after")
    def default_discount(cls, self):
        # Set discount based on category if not provided
        if self.discount is None:
            if self.category == 'electronic':
                self.discount = 0.10
            elif self.category == 'clothing':
                self.discount = 0.20
            else:
                self.discount = 0.05
        return self

prod = Product(name="Smartphone", category="electronic", price=500.0)
print(prod.discount)  # 0.10 (10% discount auto-applied for electronics)
In this code, if discount was not set by the caller, we assign a default discount after validation. This showcases that model validators can act like post-init hooks for complex default logic
medium.com
medium.com
. (Note: If a user does provide a discount, we leave it untouched as shown with a different input scenario.)
Custom Data Types
Sometimes you want to validate a completely custom type or object. Pydantic v2 allows integration via the pydantic.CoreSchema system (advanced), but for most cases you can use simpler means:
If you have a simple type alias or a specific format (e.g., a hex string of length 8), consider using constr or Field(regex=...) to enforce it.
If you have a custom class and you want Pydantic to accept it as a field, you might need to enable arbitrary_types_allowed in the model config (see Config and Settings below) and possibly write a validator to check or convert it. By default, Pydantic will not validate the internals of arbitrary classes. It will just ensure the value is an instance of that class if you set arbitrary_types_allowed=True
docs.pydantic.dev
docs.pydantic.dev
. For example:
from pydantic import ConfigDict

class Pet:
    def __init__(self, name: str):
        self.name = name

class Owner(BaseModel):
    model_config = ConfigDict(arbitrary_types_allowed=True)
    pet: Pet
    owner_name: str

owner = Owner(pet=Pet(name="Hedwig"), owner_name="Harry")
print(owner.pet.name)  # "Hedwig"
With arbitrary_types_allowed=True, Pydantic will accept Pet objects for the pet field (and reject other types), but it won't deep-validate Pet (e.g., if Pet.name is not a string, that would go unchecked unless you add your own validation)
docs.pydantic.dev
. A better approach for complex objects is often to write a Pydantic model for them if possible, or provide a custom validator that knows how to validate/construct that object from basic data.
Pydantic also supports using Annotated from typing to attach validation functions directly to types for reuse, using BeforeValidator and AfterValidator (this is an advanced v2 feature). For example, you could define JsonString = Annotated[str, BeforeValidator(lambda v: json.loads(v))] to automatically load a JSON string into a Python object when validating that type. This can be useful for custom parsing logic.
In summary, for custom types:
Prefer using Pydantic's built-in types or your own BaseModel subclasses.
If you must include raw instances of arbitrary classes, set arbitrary_types_allowed=True and be aware that you'll need to manage their validation/serialization (possibly with custom validators or by implementing __get_pydantic_json__ in the class, etc.).
You can also instruct Pydantic how to serialize custom types via model_config['json_encoders']. For example, if you had a datetime in your model and wanted a custom format, or a Decimal to serialize as str, you could specify that in json_encoders. (In Pydantic v2, json_encoders is still supported but marked deprecated, pending a new serialization system
docs.pydantic.dev
.)
Using Pydantic with FastAPI
One of Pydantic's most famous use cases is with FastAPI. FastAPI leverages Pydantic models for request validation and response rendering. By integrating Pydantic, FastAPI can automatically ensure that incoming request data is valid and convert it to Python objects for you, and also generate interactive API docs (OpenAPI) with schema definitions. Let's see an example of a simple FastAPI app using Pydantic models:
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

# Define a Pydantic model for an item
class Item(BaseModel):
    name: str
    price: float = Field(..., gt=0)
    description: Optional[str] = None

# Define another model for response (e.g., without internal fields)
class ItemPublic(BaseModel):
    name: str
    price: float

items_db = {}  # pretend database

@app.post("/items/", response_model=ItemPublic)
def create_item(item: Item):
    """Create a new item and store it."""
    item_id = len(items_db) + 1
    items_db[item_id] = item  # store the Pydantic model (or item.dict())
    return item  # You can return the model directly; FastAPI will serialize it

@app.get("/items/{item_id}", response_model=ItemPublic)
def read_item(item_id: int):
    """Get an item by ID."""
    item = items_db.get(item_id)
    if not item:
        return {"error": "Item not found"}
    return item  # returning Pydantic model or dict is fine
Key points in this FastAPI integration:
We defined an Item Pydantic model to represent the data we expect in requests when creating an item. FastAPI will automatically:
Read the JSON body of a POST request to /items/.
Validate and parse it into an Item object (or return a 422 error to the client if validation fails, with details about which fields are wrong).
Pass that item: Item parameter into our create_item function.
In the create_item endpoint, we declare response_model=ItemPublic. This tells FastAPI that the successful response should be an ItemPublic model. FastAPI will take whatever we return (here we return the Item model instance, which is fine because it has the same fields as ItemPublic or more) and serialize it to JSON. It will also omit any fields not in ItemPublic if there were extra (e.g., if Item had a secret field and ItemPublic omitted it, the client would not see it).
Using a separate model for response (ItemPublic) is a recommended pattern to avoid exposing fields that callers shouldn't see (like internal IDs, or sensitive info like cost price vs selling price, etc.).
When you run this app (using uvicorn), the automatic docs (at /docs) will show the schema for Item (with its constraints) and ItemPublic, thanks to Pydantic's schema generation. FastAPI & Pydantic Tips:
You can reuse Pydantic models for both request and response if appropriate. In simple cases, you might just use one model.
FastAPI will call model_dump() (or equivalent) on the model to convert it to dict for JSON serialization. So returning a Pydantic model is convenient. If your model contains non-serializable types (like a custom Timer object as in an Agno scenario), FastAPI/Pydantic will error when trying to serialize it
github.com
. The fix is to convert those to serializable types (e.g., exclude them or provide a json_encoders for them in the model config).
Use Field(..., example="value") in your models to provide example values in the docs.
Pydantic models can also be used for query parameters and path parameters (via Depends or so), but typically primitive types are used there. For complex query payloads, you can use a model as @app.get(..., params: Model) and FastAPI will parse query args into that model.
Using Pydantic with Agno (AI Agents Integration)
Agno is a high-performance framework for building AI agents (multi-agent systems) with memory and knowledge integration. While Agno focuses on speed and can handle various data types, it does utilize Pydantic for basic validation and especially for structured outputs from LLMs
medium.com
medium.com
. Integrating Pydantic with Agno allows you to enforce that the AI's responses conform to a certain schema. A common pattern in Agno is to specify a response_model for an agent. This model is a Pydantic BaseModel that defines the structure of the output you expect from the AI (for example, a summary with specific fields, a parsed answer, etc.). Agno can operate in a "JSON mode" where it prompts the AI to output JSON corresponding to that model, or it will parse the output into the model. Let's illustrate this with an example. Suppose we want an AI agent that generates a short movie idea with a title, genre, and a list of characters:
from agno.agent import Agent
from agno.models.openai import OpenAIChat  # Agno's wrapper for OpenAI Chat models
from pydantic import BaseModel, Field

# Define the structured output model using Pydantic
class MovieIdea(BaseModel):
    title: str = Field(..., description="The title of the movie")
    genre: str = Field(..., description="Genre of the movie")
    characters: list[str] = Field(..., description="Main character names")
    plot: str = Field(..., description="Short plot summary")

# Create an agent that should produce outputs matching MovieIdea
agent = Agent(
    model=OpenAIChat(id="gpt-4"),  # using GPT-4 via OpenAI
    instructions="You are a creative movie idea generator.",  # the prompt instructions for the agent
    response_model=MovieIdea,      # tell Agno what structure we expect
    use_json_mode=True             # agent will output strict JSON for reliable parsing
)

result = agent.run("Generate a movie idea set in New York City")
parsed_output = result.content  # Agno will parse JSON into a MovieIdea instance
print(parsed_output.title)
print(parsed_output.genre)
In this code:
We defined a Pydantic model MovieIdea with four fields. We even included descriptions for each field. (Descriptions aren't used by Pydantic for validation, but Agno might use them in the prompt to guide the AI, and they're useful for documentation or interactive prompts).
We created an Agent from Agno, specifying response_model=MovieIdea and use_json_mode=True. Under the hood, Agno will format the GPT-4 prompt so that the assistant's answer should be a JSON that fits the MovieIdea schema.
When agent.run() is called, it returns a RunResponse (in Agno) where content is the parsed Pydantic model instance (Agno uses Pydantic to parse the JSON into MovieIdea).
We can then use parsed_output as a normal MovieIdea object (access its attributes like .title, .genre, etc.) confidently, since it has been validated. If the AI's output was missing a field or had the wrong type (e.g., characters came as a single string), the Pydantic validation would fail and Agno would likely raise an error or indicate the validation issue.
Note: For this to work, you need to have Agno and an OpenAI API key set up (as shown in Agno's documentation)
docs.agno.com
docs.agno.com
. The example uses GPT-4 (OpenAIChat(id="gpt-4")). If you don't have access to GPT-4, you could use a different model id or a local LLM integration, as supported by Agno. Agno & Pydantic Best Practices:
Always define a clear response_model for agents when you need structured data back. It makes post-processing much easier.
Keep the model fields as simple types (str, int, list of str, etc.) so that the AI can output them easily and Pydantic can parse them. Complex nested models can be used too, but ensure the prompt and model account for that structure.
If you encounter serialization errors when using Pydantic within FastAPI or other frameworks (e.g., Agno's objects like a Timer or other internal classes appear in your model), consider updating your model to exclude those fields or provide a serialization method. For instance, you might not want to include an agno.utils.timer.Timer in a FastAPI response directly
github.com
github.com
. Instead, store simple numeric timings or exclude it via Pydantic's model_dump(exclude={...}) when returning a response.
By integrating Pydantic into Agno workflows, you add a layer of reliability — your AI agent won't just return free-form text, but rather well-defined data structures that your code can immediately work with or pass to other systems.
Version History and Differences: Pydantic v1 vs v2
Pydantic has evolved from v1.x to v2.x with significant improvements and some breaking changes. Here's a summary of the key differences and a brief history:
Performance: Pydantic v2 (released June 2023) introduced a Rust-based core for validation, resulting in a massive speed boost (5x-50x faster serialization/validation compared to v1)
medium.com
. For most users, upgrading to v2 yields performance gains "for free" without code changes, especially when handling many or large models.
Strict Mode & Type Coercion: In v1, Pydantic would often coerce types silently (e.g., string "123" to int 123). In v2, the default is still to coerce when possible (non-strict mode), but a global strict mode was introduced. If you enable model_config = ConfigDict(strict=True) on a model, Pydantic will not perform any type coercion for that model: inputs must exactly match the field types or validation will fail
docs.pydantic.dev
docs.pydantic.dev
. You can also use Field(strict=True) on individual fields. By default (strict=False), Pydantic v2 continues to convert types (with clearer rules defined in the docs' conversion table). Enabling strict mode can catch data that is of the wrong type rather than casting it (e.g., a float in a string field would error instead of becoming a string)
docs.pydantic.dev
.
Method Name Changes: Many BaseModel method names have changed for clarity in v2. The new naming scheme is model_* for instance methods and validate_* or model_validate for class methods. Some important changes:
BaseModel.dict() -> BaseModel.model_dump()
docs.pydantic.dev
.
BaseModel.json() -> BaseModel.model_dump_json()
docs.pydantic.dev
.
BaseModel.parse_obj() (class method to create a model from a dict) -> BaseModel.model_validate()
docs.pydantic.dev
.
BaseModel.construct() (create model without validation) -> BaseModel.model_construct()
docs.pydantic.dev
.
These old methods from v1 are generally available in v2 but marked as deprecated. It's best to switch to the new names. For example, use user.model_dump() instead of user.dict(). If you forget, v2 might still allow it but could warn or eventually remove them.
Validation Decorators: As mentioned earlier, @validator and @root_validator are deprecated in v2
docs.pydantic.dev
. They still work (with a deprecation warning) but it's encouraged to use @field_validator and @model_validator for new code. The new validators offer clearer control over when they run (before or after standard validation) and better type hints.
Config and Settings Changes: In v1, model configuration was done via an inner class Config. In v2, that still works (as class Config or using model_config attribute with a ConfigDict), but the style has shifted. For example, instead of:
class MyModel(BaseModel):
    class Config:
        validate_assignment = True
        arbitrary_types_allowed = True
you would do:
class MyModel(BaseModel):
    model_config = ConfigDict(validate_assignment=True, arbitrary_types_allowed=True)
The effect is the same. Many config options remain, some defaults changed slightly. Notably, BaseSettings (for reading environment variables) was moved out of Pydantic into a new package pydantic-settings. If you used BaseSettings in v1 for config management, you should install pydantic-settings and use SettingsConfigDict (or continue using pydantic.v1.BaseSettings from the compatibility module). The idea is to separate the concerns and allow a lighter Pydantic core.
Required vs Optional Fields: v2 clarified the handling of Optional fields. As discussed, Optional without a default is still required. The migration guide even provided a code mod to add = None to Optional fields to avoid confusion
medium.com
. So if you had Optional[X] fields, make sure to explicitly set a default if you intend them to be truly optional.
Type Changes: Some types were adjusted:
Dict and other generics might behave slightly differently in edge cases (e.g., as noted, a BaseModel isn't auto-converted to dict in a dict field
stackoverflow.com
).
Union (the | operator types) have more predictable error messages and no longer preserve the input type in BaseModel (v1 used to sometimes keep the input if it was already a model).
Constrained types (ConstrainedInt, etc.) have a new implementation but function similarly; you often use Field or type annotations to achieve the same.
Some exotic types (IPvAnyAddress, etc.) moved to pydantic-extra-types.
Error Handling: The ValidationError in v2 has some improved methods. One change is that in custom validators, if you raise a TypeError, v1 would wrap it as a ValidationError, but v2 does not do that automatically
docs.pydantic.dev
. So you should raise ValueError or ValueError subclasses for validation issues to be safe.
Deprecations and Backwards Compatibility: Pydantic v2 includes a compatibility layer for v1. If needed, you can import from pydantic.v1 import BaseModel to continue using v1-style code within v2
docs.pydantic.dev
. This is mainly to help in large projects migrating gradually. New projects should stick to the v2 interface. The Pydantic team provided a bump-pydantic tool to assist with migration by automating some code transformations
medium.com
medium.com
.
In short, Pydantic v2 brings better performance and a cleaner API, at the cost of some rename/deprecation adjustments. Most of the core concepts (models, fields, validation) remain the same, so if you learn with v2 in mind (as in this guide), you are set for the future. If you come across older code or tutorials (v1.x), be aware of the differences: method names and validator syntax being the main ones. The official migration guide
docs.pydantic.dev
 is very helpful if you need to port code between versions.
Best Practices and Common Gotchas
To wrap up, here are some best practices and potential pitfalls to keep in mind when using Pydantic (summary of things we've discussed and additional tips):
Leverage Type Hints for Validation: Design your models with clear type annotations. This makes your intent explicit and lets Pydantic do the heavy lifting. Use standard types and Pydantic types (like EmailStr, HttpUrl, PositiveInt, etc.) to get validation for free.
Field Defaults vs Required: Remember that a field is required if and only if it has no default. If a field can be omitted, give it a default (even if that default is None). Conversely, use ... (Ellipsis) or just omit a default to require a field. This is crucial for correct model usage and avoiding unintended "field required" errors or acceptance of nulls
medium.com
.
Optional Fields (Nullable vs Optional): Mark fields as Optional[X] (or X | None in Python 3.10+ syntax) and use a default of None if you mean "this field can be null or missing." If you mean "it can be null but must be provided", use Optional without default but that’s a rarer case. Understand the difference between "not provided" and "provided but null".
Avoid Any when possible: A field of type Any will accept anything and perform no validation. This might be convenient, but you lose safety. Try to specify a type or use a Union of expected types. If you truly want to allow anything, consider whether Pydantic is needed for that field at all. When interacting with external data (like JSON from disk or network), it's better to be specific.
Use Field Metadata: Add constraints and metadata using Field(...). This improves validation (e.g., Field(gt=0) ensures positivity), and documents your model. The description and example metadata are especially useful in FastAPI, as they propagate to API docs.
Mutable Defaults Are Safe (with Pydantic): As demonstrated earlier, Pydantic will deep copy mutable defaults like lists/dicts for each instance
stackoverflow.com
. So you won't get the classic bug of all instances sharing one list. Still, for unique dynamic defaults (like a timestamp of creation or a unique ID), use default_factory. E.g., id: UUID = Field(default_factory=uuid4) ensures each instance gets a fresh UUID.
Validation on Assignment: By default, once a model is created, assigning to its attributes does not trigger validation. If you want model fields to re-validate on assignment, set model_config = ConfigDict(validate_assignment=True) (v2) or Config.validate_assignment = True (v1) in the model. This can be handy to catch errors if your program mutates models. But turn it on only if needed, as it incurs overhead on each set attribute.
Immutability: If you want to make a model (or certain fields) immutable (read-only after creation), Pydantic allows that. You can set frozen=True in model_config to make the entire model immutable (attempts to set attributes will raise errors). This is like making a dataclass frozen. Alternatively, in v1 you would use allow_mutation = False in Config.
Custom JSON Serialization: When sending Pydantic models over the network (e.g., in FastAPI or writing to file), ensure all fields are serializable (JSON-friendly types). If you have a custom type (like a datetime or Decimal), Pydantic will handle many common ones automatically. For truly custom classes, implement how to serialize them by either:
Converting them to built-in types in a model model_dump (you can override BaseModel.model_dump in a subclass or use __get_pydantic_json__ on the custom class if supported).
Using json_encoders in the model config to map a class to a serializer function (still available in v2 for now)
docs.pydantic.dev
docs.pydantic.dev
.
Or simply excluding those fields from output and not returning them in an API. For example, if an Agno Timer object cannot be serialized, you might mark it with exclude=True in Field or pop it out before returning the data.
Performance Tips: Pydantic v2 is very fast, so in most cases you don't need special adjustments. But if you're in a tight loop or performance-critical path:
Use model_construct() to create models without validation when you are sure data is already valid (e.g., stored data from a previous model). This saves validation cost
docs.pydantic.dev
.
Use model_copy(update={...}) (v2) to copy a model with changes instead of reconstructing from scratch.
Consider pydantic.TypeAdapter for parsing large lists of primitives or repeating validations. TypeAdapter lets you reuse the parsing logic without creating full models each time.
Batch process data where possible (e.g., if reading a JSON array of objects, you can use model_validate on a list of dicts with List[MyModel] annotation through TypeAdapter).
Finally, profile if needed; Pydantic v2's performance often rivals handwritten parsing, so clear code is usually worth any tiny cost.
Keep Models Focused: It's often better to have multiple smaller models than one giant model that does everything. This aids reuse and clarity. For instance, create separate models for input vs output if they differ (as shown with Item vs ItemPublic). Compose models (nesting) rather than duplicating fields.
Error Handling and User Feedback: When using Pydantic in a script or non-FastAPI context, catch ValidationError exceptions to provide friendly feedback. You can use e.errors() which gives a list of error details for each field, or simply cast it to string as we did in examples to get a human-readable summary. FastAPI automatically catches these and turns them into a JSON response for API clients, so you usually don't handle it manually there.
Stay Updated: Pydantic is actively maintained. By following best practices in v2, your code should remain stable. Keep an eye on deprecation warnings (run your tests with warnings on) — for example, using @validator in v2 will emit a deprecation warning so you know to switch to @field_validator.
Utilize Documentation: Pydantic's documentation and community recipes are excellent. If you're unsure how to do something (like allow one field or another, enforce a conditional requirement, etc.), chances are there's an example of it. Refer to the official docs for topics like Strict Mode
docs.pydantic.dev
, Alias Generators (for automatically converting field names like first_name to firstName in JSON), Dataclasses support, and more.
By following these guidelines and being aware of the differences between Pydantic versions, you'll be able to use Pydantic to write cleaner, safer, and more maintainable Python code for data handling. Happy modeling!
Further Reading
For more in-depth information, you may consult the official Pydantic documentation and resources:
Pydantic Docs: The official docs cover all topics (models, validators, settings, etc.) with examples. It's a great reference
docs.pydantic.dev
docs.pydantic.dev
.
Migration Guide: If you are coming from Pydantic v1, read the migration guide
docs.pydantic.dev
 for detailed explanations of changes.
FastAPI Docs: FastAPI documentation has a section on using Pydantic for request/response models, including advanced scenarios (e.g., dynamic Union of different models for different payloads).
Agno Documentation: Agno's docs and examples show various agent setups. Look for "structured output" examples to see Pydantic in action with LLMs
docs.agno.com
docs.agno.com
.
Community Tutorials: Many blogs and tutorials (on Medium, Dev.to, etc.) provide Pydantic tips and patterns (e.g., handling mutable defaults, complex relationships, etc.). These can be helpful for practical scenarios.
By understanding and applying the concepts in this guide, you'll be well-equipped to use Pydantic effectively in your projects, whether it's for configuration, data exchange, or ensuring your AI agents' outputs are as structured as you expect. Good luck, and enjoy the confidence that comes with type-validated data!
Citations

Migrating to Pydantic V2. On the 30th of June 2023, the second… | by Brecht Verhoeve | CodeX | Medium

https://medium.com/codex/migrating-to-pydantic-v2-5a4b864621c3

Migrating to Pydantic V2. On the 30th of June 2023, the second… | by Brecht Verhoeve | CodeX | Medium

https://medium.com/codex/migrating-to-pydantic-v2-5a4b864621c3

Pydantic Essentials and Tips: Default Values | by Shreyansh | Medium

https://medium.com/@sj.shreyanshjain2002/pydantic-essentials-and-tips-default-values-c0ba7b3e2508

Pydantic: A Guide With Practical Examples | DataCamp

https://www.datacamp.com/tutorial/pydantic

python - How to give a Pydantic list field a default value? - Stack Overflow

https://stackoverflow.com/questions/63793662/how-to-give-a-pydantic-list-field-a-default-value

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

python - pydantic v1 vs v2: dict field inside a model - Stack Overflow

https://stackoverflow.com/questions/77946717/pydantic-v1-vs-v2-dict-field-inside-a-model

python - pydantic v1 vs v2: dict field inside a model - Stack Overflow

https://stackoverflow.com/questions/77946717/pydantic-v1-vs-v2-dict-field-inside-a-model

python - pydantic v1 vs v2: dict field inside a model - Stack Overflow

https://stackoverflow.com/questions/77946717/pydantic-v1-vs-v2-dict-field-inside-a-model

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/#changes-to-config

Pydantic Essentials and Tips: Default Values | by Shreyansh | Medium

https://medium.com/@sj.shreyanshjain2002/pydantic-essentials-and-tips-default-values-c0ba7b3e2508

Pydantic Essentials and Tips: Default Values | by Shreyansh | Medium

https://medium.com/@sj.shreyanshjain2002/pydantic-essentials-and-tips-default-values-c0ba7b3e2508

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Pydantic error on fast api · Issue #2286 · agno-agi/agno · GitHub

https://github.com/agno-agi/agno/issues/2286

PydanticAI, Agno, or CrewAI? Choosing the Right Framework for Your AI Agent | by Nayeem Islam | Medium

https://medium.com/@nomannayeem/pydanticai-agno-or-crewai-choosing-the-right-framework-for-your-ai-agent-e41dea879ce6

PydanticAI, Agno, or CrewAI? Choosing the Right Framework for Your AI Agent | by Nayeem Islam | Medium

https://medium.com/@nomannayeem/pydanticai-agno-or-crewai-choosing-the-right-framework-for-your-ai-agent-e41dea879ce6

Agent with Structured Outputs - Agno

https://docs.agno.com/examples/models/openai/responses/structured_output

Agent with Structured Outputs - Agno

https://docs.agno.com/examples/models/openai/responses/structured_output

Pydantic error on fast api · Issue #2286 · agno-agi/agno · GitHub

https://github.com/agno-agi/agno/issues/2286

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Migrating to Pydantic V2. On the 30th of June 2023, the second… | by Brecht Verhoeve | CodeX | Medium

https://medium.com/codex/migrating-to-pydantic-v2-5a4b864621c3

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Migrating to Pydantic V2. On the 30th of June 2023, the second… | by Brecht Verhoeve | CodeX | Medium

https://medium.com/codex/migrating-to-pydantic-v2-5a4b864621c3

Migrating to Pydantic V2. On the 30th of June 2023, the second… | by Brecht Verhoeve | CodeX | Medium

https://medium.com/codex/migrating-to-pydantic-v2-5a4b864621c3

Configuration - Pydantic

https://docs.pydantic.dev/latest/api/config/

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Migration Guide - Pydantic

https://docs.pydantic.dev/latest/migration/

Agent with Structured Outputs - Agno

https://docs.agno.com/examples/models/openai/responses/structured_output

Agent with Structured Outputs - Agno

https://docs.agno.com/examples/models/openai/responses/structured_output