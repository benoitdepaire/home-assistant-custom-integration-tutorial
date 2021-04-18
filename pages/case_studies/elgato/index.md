## Elgato integration

The Elgato integration is an integration developed for the [Elgato Key Light](https://www.elgato.com/en/gaming/key-light). This key light allows you to control both the brightness and color temperature over Wifi, using the Elgato software tool. 

Frank Nijhof has created both a [3rd party Python library](https://github.com/frenck/python-elgato) to control the Elgato keylight and a [Hass integration](https://github.com/home-assistant/core/tree/dev/homeassistant/components/elgato) for it as well. 

This distinction between a separate Python library, available in PyPi, to directly control the physical device and a separate integration within Hass, is a key concept of the Hass architecture. 

