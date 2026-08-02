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

### Docker build

`docker build -t fastapi-app .  `

### Docker Run

` docker run -p 8080:8080 --env-file .env fastapi-app`

## Google Cloud

`gcloud --version `

` gcloud auth login` - prompts

`  gcloud config set project PROJECT_ID`

` gcloud services enable run.googleapis.com` - For running our containers

` gcloud services enable cloudbuild.googleapis.com` - For building docker images in the cloud

` gcloud services enable artifactregistry.googleapis.com` - For storing our docker images

` gcloud services list --enabled`

-> Create Artifact registry

`gcloud artifacts repositories create fastapi-repo --repository-format=docker --location=asia-south1`

## Build and Push to Artifact Registry

`gcloud builds submit --tag asia-south1-docker.pkg.dev/YOUR_PROJECT_ID/fastapi-repo/fastapi-app`

## Deploy to Cloud Run

`gcloud run deploy fastapi-service --image asia-south1-docker.pkg.dev/fastapi-blog-sst/fastapi-repo/fastapi-app --region asia-south1 --allow-unauthenticated`

## Create secret key

` python3 -c "import secrets; print(secrets.token_hex(32))"`

### In Google console

-> Go to cloud run -> go to your service
-> Add env evariables,
change frontend url - https://fastapi-service-736178705031.asia-south1.run.app
change mail port - 587
Click Deploy

-> Add domain name from your desired service

### Verify domain ownership in Google search console

-> Add your domain there (not url prefix)
-> Verify ownership using TXT record, you need to add in your DNS provider (namecheap)-> Add new TXT record - @ - value (given by google)

-> Once verified, we can create domain mapping using comamnd below

` gcloud beta run domain-mappings describe --domain=myawesomeapp.com --region=us-east4`

-> It will give some A records and AAA records (4 records each)
-> Add it in your DNS provider as host @ (namecheap)
-> After one hour or so, u will be able to see in new domain

### After Code changes,

`gcloud builds submit --tag asia-south1-docker.pkg.dev/YOUR_PROJECT_ID/fastapi-repo/fastapi-app`

`gcloud run deploy fastapi-service --image asia-south1-docker.pkg.dev/YOUR_PROJECT_ID/fastapi-repo/fastapi-app --region asia-south1 --allow-unauthenticated`

-> G Cloud Run rooling new version with zero downtime, if health cehcks fails it route traffics to old version

-> If you change DB schema:

-> alembic upgrade head, if you use two varibales, you can run this comamnd, otherwise change url and alembic upgrade head
