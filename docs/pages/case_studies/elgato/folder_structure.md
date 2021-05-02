## Folder Structure

The Elgato integration is an official component (=integration) of Hass and part of the Home Assistant Core repository.  Official components can be found in the folder core/homeassistant/components. The Elgato integration can be found [here](https://github.com/home-assistant/core/tree/dev/homeassistant/components/elgato). Let's have a look at the folder structure:

- translations
    - ca.json
    - cs.json
    ...
    - zh-Hant.json
- __init__.py
- config_flow.py
- const.py
- light.py
- manifest.json
- strings.json

So, what is the purpose of each of these files? Let's have a look:

- manifest.json: As discussed [here](https://developers.home-assistant.io/docs/creating_integration_manifest), this file specifies basic information about the integration. 
- config_flow.py: Contains a config_flow object which will allow the user to configure the integration through the UI
- const.py: An module with constants for this integration
- strings.json: Contains a set (dictionary actually) of strings that the integration uses (for its UI?) which can be localized.
- translations/*: The translations (localization) of strings.json in various languages.
_ __init__.py: Contains several methods to setup this integration in Hass
- light.py: Creates a class for the Elgato Entity (based on the LightEntity class)