# Fast-Features: A Modular Approach to FastAPI Development

`fastfeatures` is a powerful toolkit for FastAPI projects designed to accelerate development by providing scaffolding, feature generation, and automatic discovery of project components. It promotes a modular and organized project structure, allowing developers to focus on business logic rather than boilerplate code.

## Why a Modular Structure?

As applications grow, maintaining a single, monolithic codebase becomes increasingly challenging. A modular, feature-based architecture offers several advantages:

* **Scalability**: By organizing your code into self-contained features, you can easily scale your application by adding, removing, or modifying features without affecting other parts of the system.
* **Maintainability**: A modular structure makes it easier to understand, debug, and test your code. Each feature has a clear responsibility, reducing cognitive load and simplifying maintenance.
* **Separation of Concerns**: By separating your application into distinct features, you enforce a clean separation of concerns, leading to more robust and reliable code.
* **Team Collaboration**: A modular architecture allows multiple developers to work on different features simultaneously with minimal conflicts, improving team productivity.

`fast-features` is designed to help you achieve these benefits by providing a solid foundation for building modular FastAPI applications.

## Key Features

* **Project Scaffolding**: Kickstart your FastAPI project with a production-ready, modular structure in seconds. The `fastfeatures-scaffold` command generates a new project with a logical directory structure, including a core application setup and an empty `features` directory, ready for you to start building.
* **Feature Generation**: Accelerate your development workflow by generating new features with a single command. The `fastfeatures-feature` command creates a new feature with a predefined structure, including models, services, and routes, so you can focus on implementing the business logic.
* **Automatic Settings Generation**: Simplify your application's configuration with automatic settings generation. The `fastfeatures-settings` command generates a Pydantic `settings.py` file from your `.env` file, with support for nested settings, providing a type-safe and organized way to manage your application's configuration.
* **Automatic Route Discovery**: `fast-features` automatically discovers and includes `APIRouter` instances from your features, so you don't have to manually wire up your routes. This reduces boilerplate code and ensures that your routes are always up-to-date.
* **Automatic Model Discovery**: `fast-features` automatically discovers your `SQLModel` and `SQLAlchemy` models, making it easy to work with your database and ensuring that your models are always available when you need them.

## Installation

### Installation with pip

```bash
pip install fastfeatures
```

### Installation with Poetry

```bash
poetry add fastfeatures
```

## Getting Started

1. **Create a new project:**

   ```bash
   fastfeatures-scaffold
   ```

2. **Generate a new feature:**

   ```bash
   fastfeatures-feature
   ```

3. **Enable route discovery in `app/main.py`:**
It is already enabled by default if you used `fastfeatures-scaffold` command.

   ```python
   from fastfeatures.core.routes_discoverer import add_features_routes
   from app import features

   # ... (existing app setup)

   add_features_routes(app, features)
   ```

4. **Run your application:**

   ```bash
   uvicorn main:app --reload
   ```

## Usage

### Project Scaffolding

To create a new FastAPI project, use the `fastfeatures-scaffold` command. This command will prompt you for the project name and description.

```bash
fastfeatures-scaffold
```

This will create a new project scaffold in the current directory with the following structure:

```
<PROJECT_NAME>/
├── .env
├── main.py
└── app/
    ├── __init__.py
    ├── core/
    │   ├── __init__.py
    │   ├── settings.py
    │   └── lib/
    │       ├── __init__.py
    │       └── database.py
    ├── features/
    │   └── __init__.py
    └── main.py
```

### Feature Generation

To generate a new feature, use the `fastfeatures-feature` command. This command will prompt you for the feature name.

```bash
fastfeatures-feature
```

This will create a new feature directory inside `app/features` with the following structure:

```
app/features/<feature_name>/
├── __init__.py
├── models/
│   ├── __init__.py
│   └── <feature_name>.py
├── routes.py
└── services/
    ├── __init__.py
    └── <feature_name>_services.py
```

### Settings Generation

To generate a `settings.py` file from your `.env` file, use the `fastfeatures-settings` command.

```bash
fastfeatures-settings --env-file=.env --output-path=app/core/settings.py
```

This will generate a `settings.py` file with nested Pydantic models. For example:

```python
# THIS FILE IS AUTO-GENERATED...

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str
    DEV_MODE: bool = False

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )


settings = Settings()
```

### Route Discovery

To automatically discover and include all the `APIRouter` instances from your features, add the following to your `app/main.py`:

```python
from fastfeatures.core.routes_discoverer import add_features_routes
from app import features

# ... (existing app setup)

add_features_routes(app, features)
```

### Model Discovery

To automatically discover all your `SQLModel` and `SQLAlchemy` models, you can use the `get_sql_models` function. This is particularly useful for database migrations with Alembic.

```python
from fastfeatures.core.models_discoverer import get_sql_models
from app import features

# Discover all SQL models from the features module
sql_models = get_sql_models(features)
```

## The "Features" Concept

A "feature" is a self-contained unit of functionality that encapsulates a specific part of your application's domain. Each feature has its own models, services, and routes, promoting a clean separation of concerns and making your code easier to understand and maintain.

* **`models`**: This directory contains the data models for your feature, defined using `SQLModel` or `SQLAlchemy`.
* **`routes`**: This directory contains the API routes for your feature, defined using `FastAPI`'s `APIRouter`.
* **`services`**: This directory contains the business logic for your feature, which is used by the routes to interact with the models and perform actions.

By organizing your code into features, you can build complex applications in a more structured and maintainable way.

## Alembic Integration for Database Migrations

`fast-features` is designed to work seamlessly with Alembic for handling database migrations. By leveraging automatic model discovery, you can keep your database schema in sync with your models with minimal effort.

Here's a step-by-step guide to setting up and using Alembic in your `fast-features` project:

### 1. Set up Your Database

Before you can run migrations, you need a database. For this guide, we'll use a simple SQLite database, which is a single file on your filesystem.

**a) Configure the Database URL:**

In your `.env` file, make sure the `DATABASE_URL` is set correctly. For a SQLite database named `my_database.db` in your project's root directory, the URL should look like this:

```
DATABASE_URL="sqlite+aiosqlite:///my_database.db"
```

**b) Create the Database File:**

While SQLAlchemy will create the database file for you when the application runs, it's good practice to create it manually before running migrations. This ensures that the file exists with the correct permissions and ownership in your project directory.

Create an empty `my_database.db` file in your project's root directory with the `touch` command:

```bash
touch my_database.db
```

### 2. Initialize Alembic

With your database file in place, you can now initialize Alembic. Use the `async` template, as your application's database connection is asynchronous.

```bash
alembic init --template async migrations
```

This command creates a `migrations` directory with the necessary configuration files.

### 3. Configure `migrations/env.py` for Asynchronous Operations

Next, you need to configure Alembic to discover your application's models and use the correct asynchronous database connection. Open the `migrations/env.py` file and make the following changes:

**a) Import Modules and Discover Models:**

Add the following imports at the top of your `env.py` file:

```python
import asyncio
from sqlalchemy.ext.asyncio import create_async_engine
from sqlmodel import SQLModel
from app.core.settings import settings
from fastfeatures import get_sql_models
from app import features
```

Then, just after the line `config = context.config`, add the following code. This will discover all the SQL models from your features and set the database URL for Alembic.

```python
#### BEGIN OF CUSTOM CODE ####
# Discover all SQL models from the features module
get_sql_models(features)

# Set the database URL for Alembic
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)
#### END OF CUSTOM CODE ####
```

**b) Set the `target_metadata`:**

Find the line `target_metadata = None` and change it to:

```python
target_metadata = SQLModel.metadata
```

This tells Alembic to use the metadata from your `SQLModel` base class to detect changes in your models.

**c) Configure the Asynchronous Engine and Execution:**

Ensure your `run_migrations_online` function is defined as `async` and uses `create_async_engine` and `async with` for the connection. The `alembic init --template async` command should have set up a structure similar to this, but verify it matches:

```python
# In your env.py, locate the run_migrations_online function.
# It should look similar to this:
async def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""
    connectable = create_async_engine(
        config.get_main_option("sqlalchemy.url"),
        # Add any other engine options here if needed, e.g., poolclass=pool.NullPool
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()

# And in the main execution block at the end of env.py, ensure it's called with asyncio.run:
if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

### 4. Update Alembic Template for SQLModel

To ensure Alembic correctly recognizes `SQLModel` definitions during autogeneration, it's a good practice to add `import sqlmodel` to the migration script template.

Open `migrations/script.py.mako` and add the following to the import section:

```python
import sqlmodel
```

### 5. Generate Your First Migration

Now you're ready to generate a migration. If you haven't already, create a feature with a model:

```bash
fastfeatures-feature
```

Then, run the following command to have Alembic automatically generate a migration script based on your models:

```bash
alembic revision --autogenerate -m "Initial migration"
```

### 6. Apply the Migration

Finally, apply the migration to your database to create the tables:

```bash
alembic upgrade head
```

That's it! You have successfully set up Alembic to manage your database migrations in your `fast-features` project. For more information on Alembic, you can refer to the [official tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html).

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.