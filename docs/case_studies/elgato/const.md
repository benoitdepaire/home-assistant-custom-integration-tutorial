## Const

This module defines a set of constants relevant to the elgato package.

```python
# Integration domain
DOMAIN = "elgato"

# Home Assistant data keys
DATA_ELGATO_CLIENT = "elgato_client"

# Attributes
ATTR_IDENTIFIERS = "identifiers"
ATTR_MANUFACTURER = "manufacturer"
ATTR_MODEL = "model"
ATTR_ON = "on"
ATTR_SOFTWARE_VERSION = "sw_version"
ATTR_TEMPERATURE = "temperature"

CONF_SERIAL_NUMBER = "serial_number"
```

First, the domain of the integration is set. This has to be unique across integrations.

Next, Hass keeps a dictionary (homeassistant.data) which can be used by integrations to store custom data. The DATA_ELGATO_CLIENT holds the key to be used for this integration.

I assume the ATTR_* constants are the attributes that stored in this dictionary?
