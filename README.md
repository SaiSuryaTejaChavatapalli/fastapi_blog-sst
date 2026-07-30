DEV start: uv run fastapi dev main.py
//-------------------------------------
initialise alembic:
uv run alembic init -t async alembic

in alembic.ini -> sqlalchemy.url =
//--------------------------------------
import models # noqa: F401
from config import settings
from database import Base

config.set_main_option("sqlalchemy.url", settings.database_url)

target_metadata = Base.metadata
//----------------------------------------------------------------------

uv run alembic revision --autogenerate -m "Initial schema"

uv run alembic upgrade head

2nd column:
uv run alembic revision --autogenerate -m "Add likes to post"

//------------------------------------

ROLLBACK:

alembic downgrade -1

//------------------------------------

uv run alembic current

uv run alemic history

# AWS S3 and Boto3 - Moving File Uploads to the Cloud

uv add boto3

AWS S3 -> Create bucket -> unique name -> keep the defaults

For Block Public Access settings for this bucket section:

1. Block public access to buckets and objects granted through new access control lists (ACLs) - check
2. Block public access to buckets and objects granted through any access control lists (ACLs) - check
3. Remaining keep uncheck
   create-bucket

//--------------------------

IAM -> policies -> Create Policy -> name -> create
IAM - IAM users -> create user -> Permissions -> Attach policies directly -> search fastapi-blog-s3-policy -> next
-> Security credentials- create access key -> Application running outside AWS -> copy access key and secret access key to local machine as it is visible only once -> keep it in .env file
//-------------

# Testing the API

`uv add --dev pytest`

### Moto: library that mocks AWS services

` uv add --dev "moto[s3]"`
-> HTTPX coming in intial setup, which will give async client
-> [s3] means it will download files related to s3

### Create Test directory

` tests/__init__.py`
` conftest.py` - It's a special file pytest recognises automatically
` tests/test_posts.py` - prefix test\_ is need to pytest to pickup

### Fast API Test Client for Sync Code

```
from fastapi import FastAPI
from fastapi.testclient import TestClient

demo_app = FastAPI()

@demo_app.get("/")
def demo_home():
    return {"message" :" Hello"}

client= TestClient(demo_app)

def test_homepage():
    response = client.get("/")
    assert response.status_code == 200

```

### Run Test command

` uv run pytest tests/test_demo.py -v`

### Running single test in a file

` uv run pytest tests/test_posts.py::test_get_posts_empty -v`

### Print output, if we put any print statements in tests

` uv run pytest tests/ -s`
