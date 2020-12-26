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

#### State

The State Class represents the state of an entity in the State Machine.

##### __init__

```python
def __init__(
        self,
        entity_id: str,
        state: str,
        attributes: Optional[Mapping] = None,
        last_changed: Optional[datetime.datetime] = None,
        last_updated: Optional[datetime.datetime] = None,
        context: Optional[Context] = None,
        validate_entity_id: Optional[bool] = True,
    ) -> None:
```

The constructor shows us that a state consists of 7 attributes, of which two are mandatory - the entity_id and the state:
- entity_id: this one is mandatory and is represented by a string in the format <domain>.<object_id>
- state: A state represented as a string of maximum 255 characters long
- attributes: (optional) Extra information on the state and entity, represented as an object of type Mapping (typically a dictionary)
- last_changed: (optional) last time the state was changed, not the attributes
- last_updated: (optional) last time this object was updated
- context: (optional) Context in which it was created (= Context object)
- domain: Domain of the entity_id
- object_id: Object_id of the entity

#### StateMachine

The StateMachine class defines following methods:
- __init__: constructor
- entity_ids or async_entity_ids: Returns a list of entity ids that are being tracked.
- async_entity_ids_count: Count the entity ids that are being tracked
- all or async_all: Create a list of all states matching the filter
- get: Retrieve state of entity_id or None if not found
- is_state: Test if entity exists and is in specified state.
- remove or async_remove: Remove the state from an entity. This results in the entity no longer being tracked by the StateMachine.
- set or async_set: Set the state of an entity, add entity if it does not exist
- async_reserve: Reserve a state in the state machine for an entity being added
- async_available: Check to see if an entity_id is available to be used

Let's focus on some methods more in detail.
##### __init__

```python
def __init__(self, bus: EventBus, loop: asyncio.events.AbstractEventLoop) -> None:
    """Initialize state machine."""
    self._states: Dict[str, State] = {}
    self._reservations: Set[str] = set()
    self._bus = bus
    self._loop = loop
```

The constructor creates a StateMachine object consisting of four attributes:
- _states: This is a dictionary mapping entity_ids to a specific state
- _reservations: This is a set of entity_ids for which a state is going to be set (?). 
- _bus: This represents a specific EventBus object which fires events and registers listeners
- _loop: This is an event loop

##### async_set

This method sets the state for an entity. If the entity hasn't been registered yet, it will be by setting the state.
```python
@callback
def async_set(
    self,
    entity_id: str,
    new_state: str,
    attributes: Optional[Dict] = None,
    force_update: bool = False,
    context: Optional[Context] = None,
) -> None:
```

When called, 3 arguments are mandatory and two optiional. You have to pass at least the entity_id and the new state. The attributes is an optional argument to specify attributes of this state. Force_update is a flag which, when set to TRUE, indicates that one has to change the last_changed date even if the new state is the same as the old state.

```python
entity_id = entity_id.lower()
new_state = str(new_state)
attributes = attributes or {}
old_state = self._states.get(entity_id)
```

First, entity_id, new_state and attributes are preprocessed. Additioanlly, old_state is retrieved from the StateMachine object based on the entity_id.

```python 
if old_state is None:
    same_state = False
    same_attr = False
    last_changed = None
else:
    same_state = old_state.state == new_state and not force_update
    same_attr = old_state.attributes == MappingProxyType(attributes)
    last_changed = old_state.last_changed if same_state else None
```

Next, two flags are determined - same_state and same_attr. The former is true when the new state didn't changed compared to the old state, unless force_update is set to TRUE. The flag same_attr is true when the old attributes are the same as the new attributes. Finally, last_changed, representing the timestamp when the state changed for the last time, is unaltered when the same_state flag is TRUE. Otherwise, it is changed to None.

```python
if same_state and same_attr:
    return

if context is None:
    context = Context()

now = dt_util.utcnow()
```

If both flags are true, nothing really changed and we can stop. No new state has to be set! If not, we continue by initializing the context (if none was given as argument) and setting the current timestamp.

```python 
state = State(
    entity_id,
    new_state,
    attributes,
    last_changed,
    now,
    context,
    old_state is None,
)
self._states[entity_id] = state
```

Next, a new state is created and the state for this entity_id is overwritten.

```python
self._bus.async_fire(
    EVENT_STATE_CHANGED,
    {"entity_id": entity_id, "old_state": old_state, "new_state": state},
    EventOrigin.local,
    context,
    time_fired=now,
)
```

Finally, a new event of the type EVENT_STATE_CHANGED is fired on the event bus. The data of the event consists of a dictionary, which holds the entity_id, the old_state and the new_state.

##### async_remove

This method removes (unregisters) an entity from the StateMachine
```python
@callback
def async_remove(self, entity_id: str, context: Optional[Context] = None) -> bool:
    """Remove the state of an entity.
    Returns boolean to indicate if an entity was removed.
    This method must be run in the event loop.
    """
```

The entity_id is the only mandatory argument. The method returns a boolean value whether removal was succesful. 

```python
entity_id = entity_id.lower()
old_state = self._states.pop(entity_id, None)

if entity_id in self._reservations:
    self._reservations.remove(entity_id)

if old_state is None:
    return False
```

Next, thestate of this entity is removed from the StateMachine and stored in old_state. If this entity_id was not registered, old_sate stores the value None. If the entity_id also exists in the reservation list, it will be removed from here also. If the old_state is None, this implies that this entity_id was not registered and therefore the method returns with the value False.

```python
self._bus.async_fire(
    EVENT_STATE_CHANGED,
    {"entity_id": entity_id, "old_state": old_state, "new_state": None},
    EventOrigin.local,
    context=context,
)
return True
```

A new event is fired on the event bus. This new event is of the type EVENT_STATE_CHANGED and its payload consists of the entity_id, the old state and None as the new state. 

#### HomeAssistant

This is the main component. 

It contains the following set of methods:
- async_run: Home Assistant main entry point
- async_start: Finalize startup from inside the event loop
- add_job or async_add_job: Add job to the executor pool
- async_add_hass_job: Add a HassJob from within the event loop
- async_create_task: Create a task from within the eventloop
- async_add_executor_job: Add an executor job from within the event loop
- async_track_tasks: Track tasks so you can wait for all tasks to be done
- async_stop_track_tasks: Stop track tasks so you can't wait for all tasks to be done
- async_run_hass_job: Run a HassJob from within the event loop
- async_run_job: Run a job from within the event loop
- block_till_done or async_block_till_done: Block until all pending work is done
- _await_and_log_pending: Await and log tasks that take a long time
- stop or async_stop: Stop Home Assistant and shuts down all threads

Let's have a look at some of these methods.

##### __init__

```python
def __init__(self) -> None:
    """Initialize new Home Assistant object."""
    self.loop = asyncio.get_running_loop()
    self._pending_tasks: list = []
    self._track_task = True
    self.bus = EventBus(self)
    self.services = ServiceRegistry(self)
    self.states = StateMachine(self.bus, self.loop)
    self.config = Config(self)
    self.components = loader.Components(self)
    self.helpers = loader.Helpers(self)
    # This is a dictionary that any component can store any data on.
    self.data: dict = {}
    self.state: CoreState = CoreState.not_running
    self.exit_code: int = 0
    # If not None, use to signal end-of-loop
    self._stopped: Optional[asyncio.Event] = None
    # Timeout handler for Core/Helper namespace
    self.timeout: TimeoutManager = TimeoutManager()
```

This constructor shows the many attributs of a HomeAssistant object:
- loop: a reference to an asyncio event loop.
- bus: a reference to an event bus (to fire events and register listeners)
- services: a reference to a service registry (???)
- states: a reference to a statemachine (to keep track of the states of registered entities)
- config: the config object (??)
- components: A components object (??)
- helpers: a reference to a helpers object (??)
- _pending_tasks: a list of pending tasks (?)
- _track_task: a flag by default set to True (?)
- data: A dictionary where any component can store any doata on. 
- state: the current state of Home Assistant. Initialized as not_running

##### async_run

This is Home Assistant main entry point. This method will start Hass and will be blocked until stopped.

```python
if self.state != CoreState.not_running:
    raise RuntimeError("Home Assistant is already running")

# _async_stop will set this instead of stopping the loop
self._stopped = asyncio.Event()

await self.async_start()
```

First it is checked if Hass isn't running already. Next, the _stopped attribute is initialized to an Asyncio event. Finally, the coroutine async_start is called. This is an awaitable method and thus can be executed asynchronously. 

```python
await self._stopped.wait()
return self.exit_code
```

At the end of this method, the method will wait until the Event stored in _stopped is set to True. Only then will this method complete. (and probably shut down Home Assistant?)

##### async_start

```python
async def async_start(self) -> None:
"""Finalize startup from inside the event loop.
This method is a coroutine.
"""
...
self.state = CoreState.starting
self.bus.async_fire(EVENT_CORE_CONFIG_UPDATE)
self.bus.async_fire(EVENT_HOMEASSISTANT_START)
```

This method changes the state of Hass to 'starting' adn fires the first two events: EVENT_CORE_CONFIG_UPDATE and EVENT_HOMEASSISTANT_START.

A bit later, it seems as if everything is repeated again.

```python
# Allow automations to set up the start triggers before changing state
await asyncio.sleep(0)
...
self.state = CoreState.running
self.bus.async_fire(EVENT_CORE_CONFIG_UPDATE)
self.bus.async_fire(EVENT_HOMEASSISTANT_STARTED)
_async_create_timer(self)
```

It appears that one forces this method briefly to sleep such that other automations can be set up and triggered by the same two events that are fired again on the event bus. 

##### async_run_hass_job

```python
@callback
def async_run_hass_job(
    self, hassjob: HassJob, *args: Any
) -> Optional[asyncio.Future]:
    """Run a HassJob from within the event loop.
    This method must be run in the event loop.
    hassjob: HassJob
    args: parameters for method to call.
    """
    if hassjob.job_type == HassJobType.Callback:
        hassjob.target(*args)
        return None

    return self.async_add_hass_job(hassjob, *args)
```

This method takes a HassJob and a set of arguments for the HassJobs and runs it directly if the Hassjob is of the type "Callback". If the HassJob is of any other type (Coroutinefunction or Executor), it is passed on to async_add_hass_job, which puts the HassJob in the event loop for execution as soon as possible.
##### async_add_hass_job

```python
@callback
def async_add_hass_job(
    self, hassjob: HassJob, *args: Any
) -> Optional[asyncio.Future]:
    """Add a HassJob from within the event loop.
    This method must be run in the event loop.
    hassjob: HassJob to call.
    args: parameters for method to call.
    """
```

This method takes a HassJob object and a set of arguments to pass on to the HassJob object and adds it to the event loop for execution.

```python
if hassjob.job_type == HassJobType.Coroutinefunction:
    task = self.loop.create_task(hassjob.target(*args))
elif hassjob.job_type == HassJobType.Callback:
    self.loop.call_soon(hassjob.target, *args)
    return None
else:
    task = self.loop.run_in_executor(  # type: ignore
        None, hassjob.target, *args
    )
```

The method evaluates the type of the HassJob. Depending on the type, a different method is required to put it into the event loop. If the HassJob is a coroutine, it is added as a Task. If the HassJob is a Callback function, it is added using the call_soon method. If the HassJob is of the type executor, it is added directly to the event loop using the run_in_executor method.

In all cases, except for the callback, a asyncio.Future object is saved as the task, which is ultimately returned by the async_add_hass_job method.