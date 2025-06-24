#### 1. **Initialized Django Backend**
- Used `django-admin startproject` to scaffold the Django project.
- Ensured `manage.py` and `settings.py` were correctly configured.
#### 2. **Created a Django App**
- Ran `python manage.py startapp tracker` to create the main app.
- Registered `'tracker'` in `INSTALLED_APPS` within `settings.py`.
#### 3. **Defined Models**
- Inside `tracker/models.py`, created two core models:
    - `Habit`: to represent a habit with fields like `name`, `created_at`.
    - `HabitLog`: to track daily completion status, linked via a foreign key to `Habit`.
#### 4. **Configured Database via Environment**
- Created a `.env` file to store DB credentials like:
    ```
    POSTGRES_DB=habitdb
    POSTGRES_USER=habituser
    POSTGRES_PASSWORD=postgres
    ```
- These were injected into `docker-compose.yml` via the `environment` section.
#### 5. **Set Up Docker Compose**

- Created a `docker-compose.yml` file with two services:
    - `db`: using `postgres:15`, with volume persistence and port mapping.
    - `web`: for Django, mounting the project directory, and running migrations/server.
- Used a Dockerfile for the `web` service to install dependencies and run `gunicorn` or `python manage.py runserver`.
#### 6. **Ran Initial Migrations**
- Used `docker compose exec web python manage.py makemigrations`
- Then applied them using `docker compose exec web python manage.py migrate`
#### 7. **Verified PostgreSQL Connection**
- Logged into the database container using:
    ```
    docker compose exec db psql -U habituser
    ```
- Ensured the tables were created in `habitdb`.