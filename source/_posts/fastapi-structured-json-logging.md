---
title: Structured JSON Logging using FastAPI
date: 2023-06-24 15:25:26
image: /images/2023-fastapi-structured-json-logging/2023-fastapi-structured-json-logging.png
keywords: Python, FastAPI, Uvicorn, Logging
tags:
  - Python
  - FastAPI
  - Logging
---

Let's start with a simple FastAPI example:

```python
# src/main.py

import uvicorn
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def hello():
    return "Hello World"


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

When you run this app, you'll see logs in text format:

```plaintext
INFO:     Started server process [9529]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

If you make a request to this app, you'll see similar text logs:

```plaintext
INFO:     127.0.0.1:50824 - "GET / HTTP/1.1" 200 OK
```

What if you want logs in a JSON format?

## JSON formatting

Let's add a logger module:

```python
# src/logger.py

import json
import logging
from logging import Formatter

class JsonFormatter(Formatter):
    def __init__(self):
        super(JsonFormatter, self).__init__()

    def format(self, record):
        json_record = {}
        json_record["message"] = record.getMessage()
        return json.dumps(json_record)

logger = logging.root
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.handlers = [handler]
logger.setLevel(logging.DEBUG)
```

Here we're subclassing `logging.Formatter` and overriding the `format` method. This method takes in the log record, using which we can construct our JSON log record. We then add our formatter to the Root logger, remove other handlers and import this new logger module into our main application:

```diff
  # src/main.py

  import uvicorn
  from fastapi import FastAPI
+ from src.logger import logger

  app = FastAPI()


  @app.get("/")
  async def hello():
+     logger.info("Hello")
      return "Hello World"


  if __name__ == "__main__":
-    uvicorn.run(app, host="0.0.0.0", port=8000)
+    uvicorn.run(app, host="0.0.0.0", port=8000, log_config=None)
```

Here's how the logs look now:

```json
{"message": "Using selector: KqueueSelector"}
{"message": "Started server process [9981]"}
{"message": "Waiting for application startup."}
{"message": "Application startup complete."}
{"message": "Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)"}
```

```json
{"message": "Hello"}
{"message": "127.0.0.1:50896 - \"GET / HTTP/1.1\" 200"}
```

Not exactly structured logging, but some progress.

## Structured logging

Let's add a FastAPI middleware to log the requests. You can change the structure of the JSON log record in this method:

```python
# src/log_middleware.py

from starlette.middleware.base import BaseHTTPMiddleware
from src.logger import logger


class LogMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        logger.info(
            "Incoming request",
            extra={
                "req": { "method": request.method, "url": str(request.url) },
                "res": { "status_code": response.status_code, },
            },
        )
        return response
```

Update the `logger.py` module to look for these new fields and add them to the final log output. Also, disable the `uvicorn.access` logger now that we've the above middleware.

```diff
  # src/logger.py

  import json
  import logging
  from logging import Formatter

  class JsonFormatter(Formatter):
      def __init__(self):
          super(JsonFormatter, self).__init__()

      def format(self, record):
          json_record = {}
          json_record["message"] = record.getMessage()
+         if "req" in record.__dict__:
+             json_record["req"] = record.__dict__["req"]
+         if "res" in record.__dict__:
+             json_record["res"] = record.__dict__["res"]
+         if record.levelno == logging.ERROR and record.exc_info:
+             json_record["err"] = self.formatException(record.exc_info)
          return json.dumps(json_record)

  logger = logging.root
  handler = logging.StreamHandler()
  handler.setFormatter(JsonFormatter())
  logger.handlers = [handler]
  logger.setLevel(logging.DEBUG)

+ logging.getLogger("uvicorn.access").disabled = True
```

Finally, add the middleware to the application.

```diff
  # src/main.py

  import uvicorn
  from fastapi import FastAPI
  from src.logger import logger
+ from src.log_middleware import LogMiddleware

  app = FastAPI()
+ app.add_middleware(LogMiddleware)

  @app.get("/")
  async def hello():
      logger.info("Hello")
      return "Hello World"


  if __name__ == "__main__":
      uvicorn.run(app, host="0.0.0.0", port=8000, log_config=None)
```

You can now see structured JSON logs for the requests:

```json
{"message": "Hello"}
{"message": "Incoming request", "req": {"method": "GET", "url": "http://0.0.0.0:8000/"}, "res": {"status_code": 200}}
```

You can find the full codebase in this [repo](https://github.com/sheshbabu/fastapi-structured-json-logging-demo).
