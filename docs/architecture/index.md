## Home Assistant Architecture

A first step is to have a good grasp of the Home Assistant architecture. 

First it is important to clear up some terminology as this has changed over time and therefore can be confusing, particularly when consulting older material on the internet.

### This is not Home Assistant
As Belgians, we are all familiar with the famous painting of Margritte, called "Ceci n'est pas une pipe" (translated: "This is not a pipe") showing the picture of a pipe. This is how it sometimes feels like when studying the documentation on Home Assistant. Whenever people talk about Home Assistant, the first challenge is to discover what they really are talking about as the name covers many concepts. Furthermore, it doesn't help that terminology changed over time, so what once was the proper name for a given concept might no longer be the case. Fortunately, the Internet updates itself immediately to prevent confusion. Not.

The current date is 23 December 2020 and the [glossary](https://www.home-assistant.io/docs/glossary/) in the documntation provides following definitions:
- **HomeAssistant Core**: This is a Python program. It can be run on various operating systems and is the basis for Home Assistant. When people are talking about Home Assistant Core they usually refer to a standalone installation method that can be installed using a Virtual Environment or Docker. Home Assistant Core does not use the Home Assistant Supervisor.
- **Add-on**: Add-ons provide additional, standalone, applications that can run beside Home Assistant. Most of these, add-on provided, applications can be integrated into Home Assistant using integrations. Examples of add-ons are: an MQTT broker, database service or a file server.
- **Home Assistant Supervisor**: The Home Assistant Supervisor is a program that manages a Home Assistant installation, taking care of installing and updating Home Assistant, add-ons, itself and, if used, updating the Home Assistant Operating System.
- **Home Assistant Supervised** (Previously known as Hass.IO): Home Assistant is a full UI managed home automation ecosystem that runs Home Assistant, the Home Assistant Supervisor and add-ons. It comes pre-installed on Home Assistant OS, but can be installed on any Linux system. It leverages Docker, which is managed by the Home Assistant Supervisor. 
- **Home Assistant Operating System** (also referred to as HassOS): The Home Assistant Operating System, is an embedded, minimalistic, operating system designed to run the Home Assistant ecosystem on single board computers (like the Raspberry Pi) or Virtual Machines. The Home Assistant Supervisor can keep it up to date, removing the need for you to manage an operating system.
- **HomeAssistant** or **Hass**: A catch all term when there is no need for further distinction.

