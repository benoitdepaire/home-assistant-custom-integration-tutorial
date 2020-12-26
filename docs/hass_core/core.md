## core.md

This module contains the core components of Hass

### Important imports
- asyncio
- voluptuous
- Various parts of homeasssistent.util

### Methods
This module defines the following methods:
- split_entity_id: Split a state entity_id into domain, object_id.
- valid_entity_id: Test if an entity ID is a valid format.
- valid_state: Test if a state is valid.
- callback: Annotation to mark method as safe to call from within the event loop.
- is_callback: Check if function is safe to be called in the event loop.
- _async_create_timer: Create a timer that will start on HOMEASSISTANT_START.


### Classes

This module defines the following classes:
- HassJobType: Represents a job type (Coroutinefunction, Callback or Executor)
- HassJob : Represents a job to be run later
- CoreState: Represents the current state of Hass
- HomeAssistant: Root object of Hass
- Context: The context that triggered something
- EventOrigin: Represents the origin of an event
- Event: Representation of an event within the bus
- EventBus: Allows the firing of and listening for events
- State: Object to represent the state within the state machine
- StateMachine: Helper class that tracks the state of different entities
- Service: Representation of a callable service
- ServiceCall: Representation of a call to a service
- ServiceRegistry: Offers the services over the eventbus
- Config: Configuration settings for HomeAssistant

#### Event

An Event Object represents an event within the bus and has following constructor:

```python
def __init__(
        self,
        event_type: str,
        data: Optional[Dict[str, Any]] = None,
        origin: EventOrigin = EventOrigin.local,
        time_fired: Optional[datetime.datetime] = None,
        context: Optional[Context] = None,
    ) -> None:
```

This shows that an event consists of 5 attributes - an event_type, data, origin, time_fired and context. Only event_type and origin are mandatory. 

The following event types are imported from homeassistant.const:
- EVENT_CORE_CONFIG_UPDATE
- EVENT_CALL_SERVICE
- EVENT_HOMEASSISTANT_CLOSE
- EVENT_HOMEASSISTANT_FINAL_WRITE
- EVENT_HOMEASSISTANT_START
- EVENT_HOMEASSISTANT_STARTED
- EVENT_HOMEASSISTANT_STOP
- EVENT_SERVICE_REGISTERED
- EVENT_SERVICE_REMOVED
- EVENT_STATE_CHANGED
- EVENT_TIME_CHANGED
- EVENT_TIMER_OUT_OF_SYNC
and also
- MATCH_ALL (which is a placeholder for '*')

Furthermore, this class has some useful methods:
- as_dict: this represents the event as a dictionary


#### EventOrigin

An enumeration of possible event origins. Currently, this is limited to EventOrigin.local and EventOrigin.remote.

#### EventBus

##### __init__
```python
def __init__(self, hass: HomeAssistant) -> None:
        """Initialize a new event bus."""
        self._listeners: Dict[str, List[HassJob]] = {}
        self._hass = hass
```

The constructor shows us that the event bus is a relatively simple object. Besides a reference to a HomeAssistant object, it consists of a dictionary where the key-values are Event types (represented as strings) and the values are a list of HassJob objects. These are the listeners, i.e. they are the jobs that need to be executed when the events of a specific type are fired.

##### async_fire

```python
@callback
    def async_fire(
        self,
        event_type: str,
        event_data: Optional[Dict] = None,
        origin: EventOrigin = EventOrigin.local,
        context: Optional[Context] = None,
        time_fired: Optional[datetime.datetime] = None,
    ) -> None:
        """Fire an event.
        This method must be run in the event loop.
        """
```
This method receives 5 arguments which match the attributes of an instance of the Event Class. So, we pass it an event (in a rather remarkable way) which is then fired. 

```python
listeners = self._listeners.get(event_type, [])

# EVENT_HOMEASSISTANT_CLOSE should go only to his listeners
match_all_listeners = self._listeners.get(MATCH_ALL)
if match_all_listeners is not None and event_type != EVENT_HOMEASSISTANT_CLOSE:
    listeners = match_all_listeners + listeners
```

So, first we get the list of all listeners registered with the given event_type (if there is no key corresponding to event_type, the get method will return an empty list!). Additionally, all listeners (i.e. hassjobs linked to an event type) that should react to any event type are added if it is not an empty list, nor the original event type which was fired was of the type "EVENT_HOMEASSISTANT_CLOSE". This piece of code ends with a list of HassJobs (listeners), stored in listener, which need to be executed as the event_type they were listening to is being fired.

```python
event = Event(event_type, event_data, origin, time_fired, context)

if event_type != EVENT_TIME_CHANGED:
    _LOGGER.debug("Bus:Handling %s", event)
```

Next, an event is created based on the arguments and if this is communicated in the logger at the debug level, unless the event is of the type 'EVENT_TIME_CHANGED'

```python 
if not listeners:
    return

for job in listeners:
    self._hass.async_add_hass_job(job, event)
```

Finally, if there are no listeners, we simply stop (return). If there are HassJobs listening, we call the `async_add_hass_job` method on the HomeAssistant object linked to this EventBus, passing it both the HassJob and the Event which triggered it.

##### async_listen
```python
@callback
def async_listen(self, event_type: str, listener: Callable) -> CALLBACK_TYPE:
    """Listen for all events or events of a specific type.
    To listen to all events specify the constant ``MATCH_ALL``
    as event_type.
    This method must be run in the event loop.
    """
    return self._async_listen_job(event_type, HassJob(listener))
```

This method registers a listener with a specific event_type in this event bus. The listener can be any object that is 'callable', which implies that it can be executed within the event loop. The method casts the listener object to an HassJob and then calls the internal method `_async_listen_job`. Let's have a quick look at this method:

```python
@callback
def _async_listen_job(self, event_type: str, hassjob: HassJob) -> CALLBACK_TYPE:
    self._listeners.setdefault(event_type, []).append(hassjob)

    def remove_listener() -> None:
        """Remove the listener."""
        self._async_remove_listener(event_type, hassjob)

    return remove_listener
```

This method first tries to retrieve the list of listeners corresponding to the event_type and adding the new HassJob to this list. The `setdefault` method returns the value corresponding to the key. In case this key doesn't exist, it creates this new entry in the dictionary using the second argument as default value and returns this value.

Next, a function is created which, when called, will remove the listener again from the event bus. This method is returned.