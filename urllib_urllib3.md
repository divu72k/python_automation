# Python `urllib` & `urllib3` — Quick Notes

## 1. `urllib`

Built into Python — **no installation required**.

### Main modules

* `urllib.request` → HTTP requests
* `urllib.parse` → Parse/build URLs
* `urllib.error` → Handle URL errors

### GET Request

```python
from urllib.request import urlopen

response = urlopen("https://example.com")

print(response.status)
print(response.read().decode())
```

### Headers

```python
from urllib.request import Request, urlopen

req = Request(
    "https://example.com",
    headers={"User-Agent": "Mozilla/5.0"}
)

response = urlopen(req)
```

### Query Parameters

```python
from urllib.parse import urlencode

params = {"q": "python", "page": 1}
query = urlencode(params)

url = "https://example.com/search?" + query
```

### Parse URL

```python
from urllib.parse import urlparse

result = urlparse("https://example.com/products?id=10")

print(result.scheme)  # https
print(result.netloc)  # example.com
print(result.path)    # /products
print(result.query)   # id=10
```

### Error Handling

```python
from urllib.error import HTTPError, URLError

try:
    response = urlopen("https://example.com")
except HTTPError as e:
    print(e.code)
except URLError as e:
    print(e.reason)
```

---

## 2. `urllib3`

Third-party HTTP client — install with:

```bash
pip install urllib3
```

### Basic GET

```python
import urllib3

http = urllib3.PoolManager()

response = http.request(
    "GET",
    "https://example.com"
)

print(response.status)
print(response.data.decode())
```

### Query Parameters

```python
response = http.request(
    "GET",
    "https://example.com/search",
    fields={"q": "python", "page": 1}
)
```

### Headers

```python
response = http.request(
    "GET",
    "https://example.com",
    headers={"User-Agent": "MyApp/1.0"}
)
```

### POST

```python
response = http.request(
    "POST",
    "https://example.com/login",
    fields={"username": "john", "password": "1234"}
)
```

### Timeout

```python
response = http.request(
    "GET",
    "https://example.com",
    timeout=5.0
)
```

### Retries

```python
http = urllib3.PoolManager(retries=3)
```

---

## 3. `urllib` vs `urllib3`

| Feature            | `urllib` | `urllib3` |
| ------------------ | -------- | --------- |
| Built into Python  | ✅        | ❌         |
| Install required   | ❌        | ✅         |
| HTTP requests      | ✅        | ✅         |
| URL parsing        | ✅        | Limited   |
| Query parameters   | ✅        | ✅         |
| Connection pooling | Basic    | ✅         |
| Retries            | Basic    | ✅         |
| Timeouts           | ✅        | ✅         |

### Remember

**`urllib`** → Standard library + URL handling

**`urllib3`** → Feature-rich HTTP client + connection pooling/retries

> Note: `urllib3` is also commonly used underneath other Python HTTP libraries.
