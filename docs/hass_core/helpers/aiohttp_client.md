## aiohttp_client

Aiohttp_client is a helper module of Hass which relies on the packages [asyncio](https://docs.python.org/3/library/asyncio.html) and [aiohttp](https://pypi.org/project/aiohttp/). The former provides built-in asynchronuous code execution since Python 3.  The latter is an async http client/server framework. 

### (some) Imports

- asyncio
- aiohttp
- Event and callback from homeassistant.core
- bind_hass from homeassistant.loader

### Methods

#### async_get_clientsession

Decorated with `callback` and `bind_hass`

```python
@callback
@bind_hass
def async_get_clientsession(
    hass: HomeAssistantType, verify_ssl: bool = True
) -> aiohttp.ClientSession:
    """Return default aiohttp ClientSession.
    This method must be run in the event loop.
    """
```

#### async_create_clientsession

Decorated with `callback`and `bind_hass`

```python
def async_create_clientsession(
    hass: HomeAssistantType,
    verify_ssl: bool = True,
    auto_cleanup: bool = True,
    **kwargs: Any,
) -> aiohttp.ClientSession:
    """Create a new ClientSession with kwargs, i.e. for cookies.
    If auto_cleanup is False, you need to call detach() after the session
    returned is no longer used. Default is True, the session will be
    automatically detached on homeassistant_stop.
    This method must be run in the event loop.
    """
```



#### async_aiohttp_proxy_web

Decorated with `bind_hass`

```python
async def async_aiohttp_proxy_web(
    hass: HomeAssistantType,
    request: web.BaseRequest,
    web_coro: Awaitable[aiohttp.ClientResponse],
    buffer_size: int = 102400,
    timeout: int = 10,
) -> Optional[web.StreamResponse]:
    """Stream websession request to aiohttp web response."""
```

#### async_aiohttp_proxy_stream

Decorated with `bind_hass`

```python
async def async_aiohttp_proxy_stream(
    hass: HomeAssistantType,
    request: web.BaseRequest,
    stream: aiohttp.StreamReader,
    content_type: Optional[str],
    buffer_size: int = 102400,
    timeout: int = 10,
) -> web.StreamResponse:
    """Stream a stream to aiohttp web response."""
```

#### _async_register_clientsession_shutdown

Decorated with `callback`

```python
@callback
def _async_register_clientsession_shutdown(
    hass: HomeAssistantType, clientsession: aiohttp.ClientSession
) -> None:
    """Register ClientSession close on Home Assistant shutdown.
    This method must be run in the event loop.
    """
```

#### _async_get_connector

Decorated with `callback`

```python
@callback
def _async_get_connector(
    hass: HomeAssistantType, verify_ssl: bool = True
) -> aiohttp.BaseConnector:
    """Return the connector pool for aiohttp.
    This method must be run in the event loop.
    """
```