**Client Libraries**

The Prometheus client libraries provide an easy way to add instrumentation to your code in order to track and expose metrics for Prometheus. 
- track metrics in the Prometheus expected format
- expose metrics via /metrics path


1. Update the /root/main.py file to replace the Question 3: PLACEHOLDER section with a Counter metric called http_requests_total. Find more details below:
   (a) First, import the Counter object from the prometheus-client library
```py
from prometheus_client import Counter
```
   (b) Create a Counter object and name it REQUESTS
         (i) Metric name should be: http_requests_total
         (ii) Description should be: Total number of requests
```py
REQUESTS = Counter('http_requests_total',
                   'Total number of requests')
```

```py
from prometheus_client import Counter # Q3: Fix the import statement 
# from prometheus_client import Counter for Task 3
# from prometheus_client import Counter, start_http_server for Task 5
# from prometheus_client import Counter, start_http_server, Gauge for Task 13

from flask import Flask

"""
Question 3: Add Request Counter
"""
REQUESTS = Counter('http_requests_total',
                   'Total number of requests')

"""
Question 10: Create error counter (ERRORS) with 'code' label
"""

"""
Question 13: Create IN_PROGRESS gauge to track active requests
"""

def before_request():
    """
    Question 13: Increment the IN_PROGRESS gauge here.
    """

def after_request(response):
    """
    Question 13: Decrement the IN_PROGRESS gauge here.
    """
    return response

app = Flask(__name__)


@app.get("/products")
def get_products():
    """
    Question 4: Increment the REQUESTS metric here
    """
    return "product"


@app.post("/products")
def create_product():
    """
    Question 4: Increment the REQUESTS metric here
    """
    return "created product", 201


@app.get("/cart")
def get_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    return "cart"


@app.post("/cart")
def create_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    return "created cart", 201


@app.errorhandler(404)
def page_not_found(e):
    """
    Question 10: Increment the ERRORS counter here.
    """
    return "page not found", 404


if __name__ == '__main__':
    """
    Question 5: Start the Prometheus metrics server on port 8000.
    """
    app.run(debug=False, host="0.0.0.0", port='6000')
```

2. Update the `/root/main.py` file to replace the `Question 4: PLACEHOLDER[1-4]` sections with the required code blocks. The `http_requests_total` metric will track the total number of requests received by the application. This metric should be incremental in all `4` API endpoints defined in `/root/main.py`. The code that needs to be added is:
```py
REQUESTS.inc()
```

```py
from prometheus_client import Counter # Q3: Fix the import statement 
# from prometheus_client import Counter for Task 3
# from prometheus_client import Counter, start_http_server for Task 5
# from prometheus_client import Counter, start_http_server, Gauge for Task 13

from flask import Flask

"""
Question 3: Add Request Counter
"""
REQUESTS = Counter('http_requests_total',
                   'Total number of requests')

"""
Question 10: Create error counter (ERRORS) with 'code' label
"""

"""
Question 13: Create IN_PROGRESS gauge to track active requests
"""

def before_request():
    """
    Question 13: Increment the IN_PROGRESS gauge here.
    """

def after_request(response):
    """
    Question 13: Decrement the IN_PROGRESS gauge here.
    """
    return response

app = Flask(__name__)


@app.get("/products")
def get_products():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "product"


@app.post("/products")
def create_product():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "created product", 201


@app.get("/cart")
def get_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "cart"


@app.post("/cart")
def create_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "created cart", 201


@app.errorhandler(404)
def page_not_found(e):
    """
    Question 10: Increment the ERRORS counter here.
    """
    return "page not found", 404


if __name__ == '__main__':
    """
    Question 5: Start the Prometheus metrics server on port 8000.
    """
    app.run(debug=False, host="0.0.0.0", port='6000')
```

3. Update the `/root/main.py` file to replace the `Question 5: PLACEHOLDER` section with the required code block to expose the metrics on an endpoint using the `start_http_server` object from the prometheus_client library. Find more details below:
    (a) Import the `start_http_server` object from the `prometheus_client` library.
```py
from prometheus_client import Counter, start_http_server
```
    (b) Start the `http` server and make sure it listens on port `8000`.
```py
start_http_server(8000)
```
- Solution:
```py
# from prometheus_client import Counter for Task 3
from prometheus_client import Counter, start_http_server # for Task 5
# from prometheus_client import Counter, start_http_server, Gauge for Task 13

from flask import Flask

"""
Question 3: Add Request Counter
"""
REQUESTS = Counter('http_requests_total',
                   'Total number of requests')

"""
Question 10: Create error counter (ERRORS) with 'code' label
"""

"""
Question 13: Create IN_PROGRESS gauge to track active requests
"""

def before_request():
    """
    Question 13: Increment the IN_PROGRESS gauge here.
    """

def after_request(response):
    """
    Question 13: Decrement the IN_PROGRESS gauge here.
    """
    return response

app = Flask(__name__)


@app.get("/products")
def get_products():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "product"


@app.post("/products")
def create_product():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "created product", 201


@app.get("/cart")
def get_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "cart"


@app.post("/cart")
def create_cart():
    """
    Question 4: Increment the REQUESTS metric here
    """
    REQUESTS.inc()
    return "created cart", 201


@app.errorhandler(404)
def page_not_found(e):
    """
    Question 10: Increment the ERRORS counter here.
    """
    return "page not found", 404


if __name__ == '__main__':
    """
    Question 5: Start the Prometheus metrics server on port 8000.
    """
    start_http_server(8000)
    app.run(debug=False, host="0.0.0.0", port='6000')
```

4. Currently, the `http_requests_total` metric does not provide the capability to track requests per `route/endpoint`. Add functionality to track the number of requests to each route and each `http` method using labels. Please find more details below:
     (a) Add two labels path and method to the REQUESTS counter. It should look like as below:
```py
REQUESTS = Counter('http_requests_total',
                   'Total number of requests', labelnames=['path', 'method'])
```
     (b) Add `.labels()` to each `REQUESTS.inc()` to pass in path and method. For example, `REQUESTS.labels('products', 'get').inc()` where `products` are a `path` and `get` is a `method`.
The `updated section` in the file will look like as below:
```py
@app.get("/products")
def get_products():
    REQUESTS.labels('products', 'get').inc()
    return "product"


@app.post("/products")
def create_product():
    REQUESTS.labels('products', 'post').inc()
    return "created product", 201


@app.get("/cart")
def get_cart():
    REQUESTS.labels('cart', 'get').inc()
    return "cart"


@app.post("/cart")
def create_cart():
    REQUESTS.labels('cart', 'post').inc()
    return "created cart", 201
```
     (c) Run the `flask.sh` command to restart the app.
Note:
The `flask.sh` script is located at: `/usr/local/bin/flask.sh`.
Run the command `flask.sh` in the terminal to restart the app.

- Solution:
```py
# from prometheus_client import Counter for Task 3
from prometheus_client import Counter, start_http_server # for Task 5
# from prometheus_client import Counter, start_http_server, Gauge for Task 13

from flask import Flask

"""
Question 3: Add Request Counter
"""
REQUESTS = Counter('http_requests_total',
                   'Total number of requests', labelnames=['path', 'method'])   # updated

"""
Question 10: Create error counter (ERRORS) with 'code' label
"""

"""
Question 13: Create IN_PROGRESS gauge to track active requests
"""

def before_request():
    """
    Question 13: Increment the IN_PROGRESS gauge here.
    """

def after_request(response):
    """
    Question 13: Decrement the IN_PROGRESS gauge here.
    """
    return response

app = Flask(__name__)


@app.get("/products")
def get_products():
    REQUESTS.labels('products', 'get').inc()
    return "product"


@app.post("/products")
def create_product():
    REQUESTS.labels('products', 'post').inc()
    return "created product", 201


@app.get("/cart")
def get_cart():
    REQUESTS.labels('cart', 'get').inc()
    return "cart"


@app.post("/cart")
def create_cart():
    REQUESTS.labels('cart', 'post').inc()
    return "created cart", 201


@app.errorhandler(404)
def page_not_found(e):
    """
    Question 10: Increment the ERRORS counter here.
    """
    return "page not found", 404


if __name__ == '__main__':
    """
    Question 5: Start the Prometheus metrics server on port 8000.
    """
    start_http_server(8000)
    app.run(debug=False, host="0.0.0.0", port='6000')
```

5. Configure a new job named api to scrape our Flask application and restart the Prometheus.

- Solution:
```yml
# /etc/prometheus/prometheus.yml
...
scrape_configs:
  - job_name: 'api'
    static_configs:
      - targets: ['localhost:8000']
```

6. Update the `/root/main.py` file to replace the `Question 10: PLACEHOLDER[1-2]` sections with the code blocks to create a metric to track the number of `404` errors. Use the steps below to complete this task.
   (a) Create a new metric called `http_errors_total` with the description Total number of errors.
   (b) Add a label called `code` to this metric.
   (c) Variable name should be `ERRORS`.
```py
ERRORS = Counter('http_errors_total',
                 'Total number of errors', labelnames=['code'])
```
   (d) In the function called `page_not_found` increment, the `ERRORS` metric and pass in the code label value as `404`.
```py
@app.errorhandler(404)
def page_not_found(e):
    ERRORS.labels('404').inc()
    return "page not found", 404
```
     (e) Run the `flask.sh` command to restart the app.
Note:
The `flask.sh` script is located at: `/usr/local/bin/flask.sh`.
Run the command `flask.sh` in the terminal to restart the app.

- Solution:
```py
from prometheus_client import Counter, start_http_server
from flask import Flask

REQUESTS = Counter('http_requests_total', 'Total number of requests', labelnames=['path', 'method'])
ERRORS = Counter('http_errors_total', 'Total number of errors', labelnames=['code'])

def before_request():

def after_request(response):
    return response

app = Flask(__name__)


@app.get("/products")
def get_products():
    REQUESTS.labels('products','get').inc()
    return "product"

@app.post("/products")
def create_product():
    REQUESTS.labels('products','post').inc()
    return "created product", 201

@app.get("/cart")
def get_cart():
    REQUESTS.labels('cart','get').inc()
    return "cart"

@app.post("/cart")
def create_cart():
    REQUESTS.labels('cart','post').inc()
    return "created cart", 201

@app.errorhandler(404)
def page_not_found(e):
    ERRORS.labels('404').inc()
    return "page not found", 404

if __name__ == '__main__':
    start_http_server(8000)
    app.run(debug=False, host="0.0.0.0", port='6000')
```

7. Update the `/root/main.py` file to replace the `Question 13: PLACEHOLDER[1-3]` sections with the code blocks to create a new Gauge metric to track the number of active requests being processed. Find below more details:
(a) Import the `Gauge` object from `prometheus_client` library.
```py
from prometheus_client import Counter, start_http_server, Gauge
```
(b) Create a new metric called inprogress_requests with a description Total number of requests in progress and store it in a variable called `IN_PROGRESS`
(c) To calculate the `in_progress` request count in the app, before every request we’ll `increment` the metric, then send a response for a request, and finally, we’ll `decrement` the metric.
(d) Both actions can be done in the `before_request` and `after_request` functions.
(e) Call the `.inc()` method in the `before_request` function and call the `.dec()` method in the `after_request` function.
```py
IN_PROGRESS = Gauge('inprogress_requests',
                    'Total number of requests in progress')

def before_request():
    IN_PROGRESS.inc()

def after_request(response):
    IN_PROGRESS.dec()
    return response
```
     (f) Run the flask.sh command to restart the app.
Note:
The flask.sh script is located at: /usr/local/bin/flask.sh.
Run the command flask.sh in the terminal to restart the app.

- Solution:
```py
from prometheus_client import Counter, start_http_server, Gauge
from flask import Flask

REQUESTS = Counter('http_requests_total', 'Total number of requests', labelnames=['path', 'method'])
ERRORS = Counter('http_errors_total', 'Total number of errors', labelnames=['code'])
IN_PROGRESS = Gauge('inprogress_requests', 'Total number of requests in progress')
def before_request():
    IN_PROGRESS.inc()

def after_request(response):
    IN_PROGRESS.dec()
    return response

app = Flask(__name__)

@app.get("/products")
def get_products():
    REQUESTS.labels('products','get').inc()
    return "product"

@app.post("/products")
def create_product():
    REQUESTS.labels('products','post').inc()
    return "created product", 201

@app.get("/cart")
def get_cart():
    REQUESTS.labels('cart','get').inc()
    return "cart"

@app.post("/cart")
def create_cart():
    REQUESTS.labels('cart','post').inc()
    return "created cart", 201

@app.errorhandler(404)
def page_not_found(e):
    ERRORS.labels('404').inc()
    return "page not found", 404

if __name__ == '__main__':
    start_http_server(8000)
    app.run(debug=False, host="0.0.0.0", port='6000')
```