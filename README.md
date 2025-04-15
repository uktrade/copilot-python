# DBT Copilot Python

![](https://codebuild.eu-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiTG41bUNVdHN1b3NRS0hTYUlmMktLbnNNQzEyTlpMRDBlYlZiV1ZjNnl4b3dyMXl0R3VIUEVIbGVnYVJWbHd0OVZndVhURFpnckp5dWx0R0llMVpHUktzPSIsIml2UGFyYW1ldGVyU3BlYyI6ImthS3RRRUtOYkljSUVVUHMiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=main)

A set of utility functions for running Django & Flask apps in AWS ECS via AWS Copilot.

## Using `dbt-copilot-python`

### Supported Python versions

3.9, 3.10, 3.11, 3.12 and 3.13.

### Installation

```shell
pip install dbt-copilot-python
```

### Usage

#### `ALLOWED_HOSTS` setting

Add the ECS container IP to `ALLOWED_HOSTS` setting so that the Application Load Balancer (ALB) healthcheck will succeed:

```python
from dbt_copilot_python.network import setup_allowed_hosts

ALLOWED_HOSTS = [...]

ALLOWED_HOSTS = setup_allowed_hosts(ALLOWED_HOSTS)
```

#### Celery health check

Add the health check in your application's Celery config file...

```python
from dbt_copilot_python.celery_health_check import healthcheck

celery_app = Celery("application_name")
...

celery_app = healthcheck.setup(celery_app)
```

Add the health check to the Celery worker service in `docker-compose.yml`...

```yaml
healthcheck:
  test: [ "CMD-SHELL", "python -m dbt_copilot_python.celery_health_check.healthcheck" ]
  interval: 10s
  timeout: 5s
  retries: 2
  start_period: 5s
```

In your `*-deploy` codebase, add the health check to the Celery worker service in `copilot/celery-worker/manifest.yml`...

```yaml
healthcheck:
  command: [ "CMD-SHELL", "launcher bash -c 'python -m dbt_copilot_python.celery_health_check.healthcheck'" ]
  interval: 10s
  timeout: 5s
  retries: 2
  start_period: 10s
```

#### `DATABASES` setting

To configure the `DATABASES` setting from an RDS JSON object stored in AWS Secrets Manager, there are two options.

1. Configure the `DATABASES` setting to use a database URL (recommended):

    Note: This is dependent on the [`dj-database-url`](https://pypi.org/project/dj-database-url/) package which can be installed via `pip install dj-database-url`.

    ```python
    import dj_database_url

    from dbt_copilot_python.database import database_url_from_env
   
    DATABASES = {
        "default": dj_database_url.config(
            default=database_url_from_env("DATABASE_ENV_VAR_KEY")
        )
    }
    ```

2. Configure the `DATABASES` setting to use a dictionary containing the settings:

    ```python
    from dbt-copilot-python.database import database_from_env

    DATABASES = database_from_env("DATABASE_ENV_VAR_KEY")
    ```

## Contributing to `dbt-copilot-python`

### Requirements

- [Poetry](https://python-poetry.org/); `pip install poetry`

### Install dependencies & pre-commit hooks

```shell
poetry install && poetry run pre-commit install
```

### Run the tests

```shell
poetry run pytest
```

### Publishing

To publish the Python package `dbt-copilot-python`, you will need an API token.

1. Acquire API token from [Passman](https://passman.ci.uktrade.digital/secret/cc82a3f7-ddfa-4312-ab56-1ff8528dadc8/).
   - Request access from the SRE team.
   - _Note: You will need access to the `platform` group in Passman._
2. Run `poetry config pypi-token.pypi <token>` to add the token to your Poetry configuration.

Update the version, as the same version cannot be published to PyPi.

```shell
poetry version patch
```

More options for the `version` command can be found in the [Poetry documentation](https://python-poetry.org/docs/cli/#version). For example, for a minor version bump: `poetry version minor`.

Build the Python package.

```shell
poetry build
```

Publish the Python package.

_Note: Make sure your Pull Request (PR) is approved and contains the version upgrade in `pyproject.toml` before publishing the package._

```shell
poetry publish
```

Check the [PyPi Release history](https://pypi.org/project/dbt-copilot-python/#history) to make sure the package has been updated.

For an optional manual check, install the package locally and test everything works as expected.
