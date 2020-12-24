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

