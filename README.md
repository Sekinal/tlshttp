# tlshttp

Modern Python wrapper for [tls-client](https://github.com/bogdanfinn/tls-client) with httpx-like API and browser TLS fingerprinting.

## Features

- **httpx-like API** - Familiar `Client` and `AsyncClient` interface
- **Browser TLS Fingerprinting** - Bypass anti-bot detection with Chrome, Firefox, Safari profiles
- **Auto-download Binaries** - Automatically downloads the correct binary for your platform
- **Async Support** - Full async/await support with anyio
- **Cookie Management** - Proper cookie handling with working `clear()` method
- **Case-insensitive Headers** - HTTP-spec compliant header handling
- **Memory Safe** - Proper cleanup with context managers and finalizers

## Installation

```bash
pip install tlshttp
```

Or with uv:

```bash
uv add tlshttp
```

## Quick Start

### Synchronous Usage

```python
import tlshttp

# Using context manager (recommended)
with tlshttp.Client(profile="chrome_120") as client:
    response = client.get("https://example.com")
    print(response.json())

# POST with JSON
with tlshttp.Client() as client:
    response = client.post(
        "https://api.example.com/data",
        json={"key": "value"}
    )
    print(response.status_code)
```

### Asynchronous Usage

```python
import asyncio
import tlshttp

async def main():
    async with tlshttp.AsyncClient(profile="chrome_120") as client:
        response = await client.get("https://example.com")
        print(response.json())

        # Concurrent requests
        tasks = [
            client.get("https://example.com/1"),
            client.get("https://example.com/2"),
            client.get("https://example.com/3"),
        ]
        responses = await asyncio.gather(*tasks)

asyncio.run(main())
```

## Browser Profiles

```python
import tlshttp

# Use specific browser profile
client = tlshttp.Client(profile="chrome_120")
client = tlshttp.Client(profile="firefox_120")
client = tlshttp.Client(profile="safari_16_0")

# Use profile constants
client = tlshttp.Client(profile=tlshttp.Chrome.LATEST)
client = tlshttp.Client(profile=tlshttp.Firefox.DEFAULT)
```

Available profiles:
- **Chrome**: 103-133 (including PSK variants)
- **Firefox**: 102-133
- **Safari**: 15.6.1, 16.0, iOS 15.5-18.5
- **Opera**: 89-91
- **Android OkHttp**: 7-13

## Configuration

```python
import tlshttp

client = tlshttp.Client(
    profile="chrome_120",           # Browser TLS fingerprint
    timeout=30.0,                   # Request timeout in seconds
    follow_redirects=True,          # Follow HTTP redirects
    proxy="http://user:pass@host:port",  # Proxy URL
    verify=True,                    # Verify SSL certificates
    http2=True,                     # Use HTTP/2
    headers={"User-Agent": "..."},  # Default headers
    cookies={"session": "..."},     # Default cookies
    base_url="https://api.example.com",  # Base URL for relative paths
)
```

## Cookies

```python
import tlshttp

with tlshttp.Client() as client:
    # Cookies are automatically managed
    response = client.get("https://example.com/login")

    # Access cookies
    print(client.cookies.to_dict())

    # Set cookies manually
    client.cookies.set("session", "abc123", domain="example.com")

    # Clear cookies (this actually works unlike other wrappers!)
    client.cookies.clear()
```

## Headers

```python
import tlshttp

with tlshttp.Client() as client:
    response = client.get("https://example.com")

    # Case-insensitive header access
    print(response.headers["Content-Type"])
    print(response.headers["content-type"])  # Same result
    print(response.headers.get("X-Custom", "default"))
```

## Error Handling

```python
import tlshttp

with tlshttp.Client() as client:
    try:
        response = client.get("https://example.com")
        response.raise_for_status()  # Raises HTTPStatusError for 4xx/5xx
    except tlshttp.TimeoutError:
        print("Request timed out")
    except tlshttp.ConnectError:
        print("Connection failed")
    except tlshttp.HTTPStatusError as e:
        print(f"HTTP error: {e.response.status_code}")
    except tlshttp.RequestError as e:
        print(f"Request error: {e}")
```

## Comparison with Other Libraries

| Feature | tlshttp | tls-client | curl_cffi | requests |
|---------|---------|------------|-----------|----------|
| TLS Fingerprinting | Yes | Yes | Yes | No |
| Async Support | Yes | No | Yes | No |
| httpx-like API | Yes | No | Yes | ~ |
| Auto Binary Download | Yes | No | Yes | N/A |
| Cookie clear() works | Yes | No | Yes | Yes |
| Memory Leak Free | Yes | No | Yes | Yes |
| HTTP/2 | Yes | Yes | Yes | No |
| HTTP/3 | Yes | Yes | Yes | No |

## Requirements

- Python 3.10+
- Supported platforms: Linux (x64, ARM64, Alpine), macOS (x64, ARM64), Windows (x86, x64)

## License

MIT
