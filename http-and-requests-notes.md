# Notes: Python `http` module & the `requests` library

---

## 1. Python `http` — HTTP modules (standard library)

Source: https://docs.python.org/3/library/http.html

`http` is a **package**, not a single module. It bundles several sub-modules for working with HTTP, plus two useful enums.

### 1.1 Sub-modules under `http`
| Module | Purpose |
|---|---|
| `http.client` | Low-level HTTP protocol client (for high-level URL fetching, use `urllib.request` instead). |
| `http.server` | Basic HTTP server classes, built on `socketserver`. |
| `http.cookies` | Utilities for HTTP state management (cookies). |
| `http.cookiejar` | Cookie persistence across requests. |

### 1.2 `http.HTTPStatus` (added in 3.5)
- An `enum.IntEnum` subclass listing all IANA-registered HTTP status codes, their names, and reason phrases.
- Behaves like an int, so it can be compared directly to a status-code number.
- Each member exposes `.value` (the number), `.phrase` (short reason text), and `.description` (longer explanation).

Example usage pattern:
```python
from http import HTTPStatus

HTTPStatus.OK          # HTTPStatus.OK
HTTPStatus.OK == 200   # True
HTTPStatus.OK.value    # 200
HTTPStatus.OK.phrase   # 'OK'
```

**Status code groups worth remembering:**
- `1xx` – Informational (e.g. `CONTINUE`, `SWITCHING_PROTOCOLS`, `EARLY_HINTS`)
- `2xx` – Success (`OK`, `CREATED`, `ACCEPTED`, `NO_CONTENT`, `PARTIAL_CONTENT`, ...)
- `3xx` – Redirection (`MOVED_PERMANENTLY`, `FOUND`, `SEE_OTHER`, `NOT_MODIFIED`, `TEMPORARY_REDIRECT`, `PERMANENT_REDIRECT`, ...)
- `4xx` – Client error (`BAD_REQUEST`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `METHOD_NOT_ALLOWED`, `TOO_MANY_REQUESTS`, `IM_A_TEAPOT` (418, joke status), ...)
- `5xx` – Server error (`INTERNAL_SERVER_ERROR`, `BAD_GATEWAY`, `SERVICE_UNAVAILABLE`, `GATEWAY_TIMEOUT`, ...)

Note: as of Python 3.13 the constant names follow RFC 9110 naming, but old names (e.g. `REQUEST_ENTITY_TOO_LARGE`) still work for backward compatibility.

These same constants are also exposed on `http.client` (e.g. `http.client.OK`) for backward compatibility.

### 1.3 Status "category" properties (added in 3.12)
Each `HTTPStatus` member has boolean helper properties so you don't need manual range checks:

| Property | True when |
|---|---|
| `is_informational` | 100–199 |
| `is_success` | 200–299 |
| `is_redirection` | 300–399 |
| `is_client_error` | 400–499 |
| `is_server_error` | 500–599 |

```python
HTTPStatus.OK.is_success       # True
HTTPStatus.OK.is_client_error  # False
```

### 1.4 `http.HTTPMethod` (added in 3.11)
- An `enum.StrEnum` subclass listing the standard HTTP methods with descriptions.
- Compares equal to the plain string method name.

```python
from http import HTTPMethod

HTTPMethod.GET               # <HTTPMethod.GET>
HTTPMethod.GET == 'GET'      # True
HTTPMethod.GET.value         # 'GET'
HTTPMethod.GET.description   # 'Retrieve the target.'
```

**Available methods:** `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS`, `TRACE`, `PATCH`.

### 1.5 Quick takeaway
`http` itself has no request-sending functionality — it's the shared vocabulary (status codes + methods) used by `http.client`, `http.server`, and libraries like `requests`. Use it when you need readable, type-safe references to status codes/methods instead of magic numbers/strings.

---

## 2. `requests` — "HTTP for Humans" (third-party library)

Source: https://requests.readthedocs.io/en/latest/ (and its Quickstart page)

`requests` is the de-facto standard Python library for making HTTP calls — much simpler than the low-level `http.client` / `urllib`. Current documented release: **v2.34.2**. Supports **Python 3.10+** and PyPy.

### 2.1 Headline features
- Connection pooling & keep-alive handled automatically (via `urllib3`)
- Automatic content decoding (gzip/deflate, and Brotli if the `brotli`/`brotlicffi` package is installed)
- Sessions with cookie persistence
- Automatic SSL/TLS verification
- Multipart file uploads, streaming uploads/downloads
- Built-in Basic/Digest auth, proxy support, `.netrc` support

### 2.2 Making requests
Each HTTP verb has its own top-level function:
```python
import requests

r = requests.get(url)
r = requests.post(url, data={'key': 'value'})
r = requests.put(url, data={'key': 'value'})
r = requests.delete(url)
r = requests.head(url)
r = requests.options(url)
```
Each call returns a **`Response`** object (`r` in the examples).

### 2.3 Query parameters
Pass a dict via `params=` instead of building a query string by hand:
```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get(url, params=payload)
```
- Keys with a `None` value are dropped from the URL.
- A list value produces repeated keys, e.g. `key2=value2&key2=value3`.

### 2.4 Reading the response
| Attribute/Method | Gives you |
|---|---|
| `r.text` | Decoded string content (encoding auto-guessed from headers; override with `r.encoding = '...'`) |
| `r.content` | Raw `bytes` (gzip/deflate/br already decoded) |
| `r.json()` | Parsed JSON → Python object; raises `requests.exceptions.JSONDecodeError` if the body isn't valid JSON |
| `r.raw` | Raw socket-level stream — only populated if you pass `stream=True` |
| `r.status_code` | Integer status code, comparable to `requests.codes.ok` etc. |
| `r.headers` | Case-insensitive dict-like object of response headers |
| `r.cookies` | Cookies set by the server (a `RequestsCookieJar`) |
| `r.history` | List of `Response` objects from any redirects followed, oldest first |
| `r.url` | Final URL after redirects |

Important: a successful `r.json()` call does **not** mean the request succeeded — some servers return JSON bodies alongside error statuses. Always check `r.status_code` or call `r.raise_for_status()`.

### 2.5 Streaming large downloads
```python
r = requests.get(url, stream=True)
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size=128):
        fd.write(chunk)
```
`iter_content()` is preferred over reading `r.raw` directly because it also handles gzip/deflate decoding for you.

### 2.6 Custom headers
```python
headers = {'user-agent': 'my-app/0.0.1'}
r = requests.get(url, headers=headers)
```
Caveats:
- `Authorization` headers can be overridden by `.netrc` credentials, which in turn are overridden by the `auth=` parameter.
- `Authorization` headers are stripped if a redirect sends the request to a different host.
- `Content-Length` is recalculated automatically when possible.
- Header values must be strings/bytestrings.

### 2.7 Sending data in POST/PUT
- **Form-encoded:** pass a dict (or list of tuples, for repeated keys) to `data=`.
- **Raw string body:** pass a plain string to `data=` — sent as-is (no `Content-Type` set automatically).
- **JSON body:** pass a dict to `json=` — this auto-serializes and sets `Content-Type: application/json`. Note `json=` is ignored if `data=` or `files=` is also given.
- **Multipart file upload:** pass a dict to `files=`, e.g. `files={'file': open('report.xls', 'rb')}`. You can also supply `(filename, fileobj, content_type, headers)` tuples for finer control. Always open files in **binary mode**.

### 2.8 Status codes & errors
```python
r.status_code == requests.codes.ok      # built-in lookup table
r.raise_for_status()                    # raises HTTPError on 4xx/5xx, else returns None
```

**Exception hierarchy** (all inherit from `requests.exceptions.RequestException`):
- `ConnectionError` — network-level problem (DNS failure, refused connection, etc.)
- `HTTPError` — raised by `raise_for_status()` for a bad status code
- `Timeout` — request exceeded the `timeout=` value
- `TooManyRedirects` — redirect chain exceeded the max allowed

### 2.9 Timeouts
```python
requests.get(url, timeout=5)
```
- Best practice: **always set a timeout** in production code, or a hung server can freeze your program indefinitely.
- `timeout` is not a total-download time limit — it measures the gap between bytes received on the socket, not overall duration.
- With no `timeout=`, requests will wait forever.

### 2.10 Redirects
- Followed automatically for all verbs except `HEAD`.
- Disable with `allow_redirects=False` (GET/OPTIONS/POST/PUT/PATCH/DELETE).
- Enable for `HEAD` with `allow_redirects=True`.
- Inspect the chain via `r.history`.

### 2.11 Cookies
```python
r.cookies['name']                                   # read cookie from response
r = requests.get(url, cookies={'cookies_are': 'working'})  # send cookies
jar = requests.cookies.RequestsCookieJar()          # advanced: scoped by domain/path
jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
```

### 2.12 Documentation structure (for reference)
- **User Guide:** Installation → Quickstart → Advanced Usage → Authentication
- **Community Guide:** recommended add-ons (Certifi, CacheControl, Requests-Toolbelt, Requests-OAuthlib, Betamax), FAQ, support, release process
- **API Reference:** full class/method docs — Main Interface, Exceptions, Sessions, lower-level classes, Auth, Encodings, Cookies, Status-code lookup

### 2.13 How it relates to the `http` module notes above
`requests` sits **on top of** the concepts `http` defines: it sends the methods listed in `http.HTTPMethod`, and `r.status_code` corresponds to values from `http.HTTPStatus`. `requests` handles the actual socket work, connection pooling, redirects, and encoding that you'd otherwise have to manage manually with `http.client`.

---

## 3. Cheat-sheet summary

| Task | `http` (stdlib) | `requests` |
|---|---|---|
| Named status codes | `HTTPStatus.NOT_FOUND` | Same enum works; also `requests.codes.not_found` |
| Named HTTP methods | `HTTPMethod.POST` | `requests.post(...)` |
| Make an actual HTTP call | Not provided (use `http.client` or `urllib.request`) | `requests.get/post/put/delete/head/options(...)` |
| Check status category | `.is_success`, `.is_client_error`, etc. | Compare `r.status_code` or call `r.raise_for_status()` |
