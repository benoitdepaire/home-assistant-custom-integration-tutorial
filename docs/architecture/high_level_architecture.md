## General Architecture

To get a an idea of the general architectural design behind Hass , have a look at [this](https://developers.home-assistant.io/docs/architecture_index) and [this page](https://developers.home-assistant.io/docs/architecture_components) in the developer documentation (dd. 23 December 2020). Read this page first and then come back for the key take aways.

Key take aways:
- At a high level, it is important to distinguish three layers in our architecture:
    - The IoT Layer. This refers typically to the actual devices or services you want to monitor and control in a smart or automated way. 
    - The Actor Layer. This refers to actors that actively try to change the state of the smart IoT Layer by issuing commands. These actors can either be users, pre-configured services/scripts (Home Automation) or AI models (Smart Home).
    - The Central Processing Layer (Home Control). This layer keeps track of all events that are being fired by the other layers, keeps track of and communicates the current state of the entire system and orchestrates the interaction between both the Actor and IoT Layer. 
- Home Assistant Core not only implements the Central Processing Layer (Home Control) but also parts of the IoT Layer and the Actor Layer.
- As for the Home Control (CPLayer), Home Assistant implements four main parts:
    - An Event Bus
    - A State Machine
    - A Timer
    - A Service Registry
- To understand how Hass implements (part of) the IoT Layer, we first must clarify some terminology (can be found in the [glossary](https://www.home-assistant.io/docs/glossary/)):
    - Device: This refers to the physical entity which we want to integrate into our smart environment. This could be a sensor, an actuator (or even a web service?). The 'design' and 'implementation' of the Device is the responsibility of third-parties.
    - Integration (a.k.a. Component): An integration or component is the abstract equivalent of a Device.
    - Domain:
    - Platform: Platforms make the connection to a specific software or hardware platform. For example, the pushbullet platform works with the service pushbullet.com to send notifications.
    - Entity: An entity is the representation of a function of a single device, unit, or web service. There may be multiple entities for a single , unit, or web service, or there may be only one. This is very important to keep in mind: **a single entity (which can represent a specific device???) can consist of multiple entities (which exists within Home Assistant). An overview of different entities can be found [here](https://developers.home-assistant.io/docs/core/entity). Some common types are: 
        - [Light Entity](https://developers.home-assistant.io/docs/core/entity/light) which controls the brightness, hue and saturation color value, white value, color temperature and effects of a light source.
        - [Sensor Entity](https://developers.home-assistant.io/docs/core/entity/sensor) which is a read-only entity that provides some information. Information has a value and optionally, a unit of measurement.
        - [Switch Entity](https://developers.home-assistant.io/docs/core/entity/switch) which turns on or off something, for example a relay. It is important to realize that Home Assistant should be able to control the state of the Switch Entity.