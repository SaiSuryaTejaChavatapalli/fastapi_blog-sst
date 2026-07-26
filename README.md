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
