## __init.py__

Let's have a closer look at the [module](https://github.com/home-assistant/core/blob/dev/homeassistant/components/elgato/__init__.py) which turns this into a Python package.

### Imports
This module imports from following sources:
- The Elgato Python package 
- The Home Assistant Core package
    - The light component with Hass
    - The config_entries module
    - The core module
    - The const module
    - The exception module
    - Some helper modules
- The const module within this component's package

### Methods
The module contains three top level methods:
- async_setup
- async_setup_entry
- async_unload_entry

#### async_setup
This method sets up the Elgato component.

```python 
async def async_setup(hass: HomeAssistant, config: ConfigType) -> bool:
```
This method gets two arguments - a HomeAssistant instance and a ConfigType instance - and returns a boolean (probably to indicate if setup succeeded). Do note that this method is async, i.e. during it's execution it can get in a waiting status where it has to wait for some results and where it will allow other code (from other modules, methods) to be executed, until it receives the results it was waiting for and when it takes back control (at some point) to continue executing its own code. 

In other words, this setup method will not block Hass.

This method doesn't do anything. It simply returns True.

#### async_setup_entry
This method sets up the Elgato component from a Config Entry instance.

```python 
async def async_setup_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
```

This method has almost the same signature as the async_setup method, except that its second argument is a ConfigEntry instance instead of a ConfigType entry. 

_This raises the question what is the difference between both?_ Looking at the import statements, we can already see that ConfigEntry comes from the config_entries module of Hass, whereas ConfigType comes from the helper module typing. Opening this helper module shows us that `ConfigType` is simply a dictionary where the keys are strings and the values can be any type of value.

Let's have a look at the code

```python
    session = async_get_clientsession(hass)
```

First, a session is created by calling `async_get_clientsession`, a method in the aiohttp_client helper module.

```python
    elgato = Elgato(
        entry.data[CONF_HOST],
        port=entry.data[CONF_PORT],
        session=session,
    )
```

Next, an Elgato object is created. This is an instance of the third-party package  that allows direct communication with the Elgato light. To initialize this object, it receives two values from the ConfigEntry object - the host and the port of the Elgato light. Both were stored in the data dictionary of the ConfigEntry object. Finally, it also gets a session object (which presumably allows async communication??)

```python
    # Ensure we can connect to it
    try:
        await elgato.info()
    except ElgatoConnectionError as exception:
        raise ConfigEntryNotReady from exception

```

Next, we try to connect to the Elgato keylight and throw an exception if this fails

```python
    hass.data.setdefault(DOMAIN, {})
    hass.data[DOMAIN][entry.entry_id] = {DATA_ELGATO_CLIENT: elgato}


```

Next, the data object of the Hass object is manipulated. (need to look in to this!!!)

```python
    hass.async_create_task(
        hass.config_entries.async_forward_entry_setup(entry, LIGHT_DOMAIN)
    )

    return True
```

Finally, it seems that an async task is being created within the Hass object. (Requires further investigation!)