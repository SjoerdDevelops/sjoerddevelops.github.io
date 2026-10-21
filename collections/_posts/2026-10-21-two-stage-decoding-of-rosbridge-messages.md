**At work I was tasked to develop a Robot Fleet API using [FastAPI](https://fastapi.tiangolo.com/) that talks to robots through `rosbridge_server`. I wanted the whole chain to be strongly typed: from the raw websocket bytes to strongly typed domain entities and API endpoints. But typing everything has a price: validating a payload costs CPU on every message, which adds up for large and frequent streams such as camera images and point clouds. Furthermore, every payload type also needs a model, which adds development cost. Validation shouldn't be a cost that every message pays blindly, in performance or in maintenance.**

**This post describes how I resolved that tension with [msgspec](https://jcristharif.com/msgspec/): by decoding each rosbridge packet in two stages, and making the second stage optional.**

## Rosbridge Protocol

Every packet from `rosbridge_server` is a JSON object with the structure defined by the [rosbridge protocol](https://github.com/RobotWebTools/rosbridge_suite/blob/ros2/ROSBRIDGE_PROTOCOL.md). Like a letter, each packet has an envelope that says what kind of message it is and where it's going, and a payload inside it with the actual message:

- Envelope:
    - an `op` field that states what kind of message it is (`publish`, `service_response`, `action_feedback`, …),
    - a few metadata fields such as `topic`, `service`, `action` or `id`,
- Message:
    - a payload field (`msg`, `values` or `args`) that carries the actual ROS message.

Below is an example of a rosbridge packet:
```json
{"op": "publish", "topic": "/client_count", "msg": {"data": 3}}
```

## Decoding Rosbridge Packets
What the payload contains depends on the topic, service or action it belongs to. The `msg` above holds integer field called `data` but only because it was published on the `/client_count` topic. On another topic, the same field could contain different data, for example an image or vector. In this case, the type is determined by the `topic` metadata field. Importantly, this field cannot be accessed until the packet's envelope has been decoded.

Decoding a full packet therefore takes three steps: decode the envelope, use its metadata to determine the payload type, and finally decode the payload. That's getting complicated, so let's take one step at a time. We will start by decoding the envelope and routing the packet to the right handler. Then we'll ask whether the payload needs decoding at all.

### Decoding the Envelope
I chose [msgspec](https://jcristharif.com/msgspec/) for this application over [Pydantic](https://pydantic.dev/). Besides being faster than Pydantic (see [msgspec's benchmarks](https://jcristharif.com/msgspec/benchmarks.html)), it has two features that help decoding the envelope into strongly typed domain models:

- Tagged unions let the JSON decoder pick the right envelope schema directly from the `op` field.
- `msgspec.Raw` lets the payload field stay as undecoded bytes.

#### Step 1: Defining Envelope Structs
Each envelope model is defined as a generic struct tagged by its `op` value:

```python
class BaseStruct(msgspec.Struct, kw_only=True):
    """Shared config for all envelope structs."""


class Publish[T](BaseStruct, tag_field="op", tag="publish"):
    topic: str
    msg: T


class ServiceResponse[T](BaseStruct, tag_field="op", tag="service_response"):
    service: str
    id: str
    result: bool
    values: T | None = None

# ActionFeedback, ActionResult, ...
```

#### Step 2: Defining the Tagged Union
A tagged union type is defined:
```python
RosbridgeEnvelope = Publish[msgspec.Raw] | ServiceResponse[msgspec.Raw] # | ActionFeedback | ActionResult, ...
```

#### Step 3: Decoding the Envelope
A single decoder for rosbridge packets is instantiated to decode raw JSON packets into envelope models:

```python
decoder = msgspec.json.Decoder(RosbridgeEnvelope)
envelope = decoder.decode(frame)
```

`decoder.decode(frame)` returns a typed `Publish[msgspec.Raw]`, `ServiceResponse[msgspec.Raw]` or other decoded rosbridge packet. The envelope is fully validated, but the payload is still a zero-copy view of the original bytes. A packet with an unknown `op` or missing fields is rejected.

#### Step 4: Dispatching
Because each envelope has its own type, its corresponding handler can be determine by type instead of by comparing `op` strings. A simple dictionary that maps each envelope type to its handler is enough:

```python
type RosbridgeEnvelopeHandler = Callable[[RosbridgeEnvelope], None]

rosbridge_frame_handlers: dict[RosbridgeEnvelope, RosbridgeEnvelopeHandler] = {
    Publish: handle_publish,
    ServiceResponse: handle_service_response,
    # ActionFeedback: handle_action_feedback,
    # ActionResult: handle_action_result,
    # ...
}

handler = rosbridge_frame_handlers[type(envelope)]
```
This works because a decoded `Publish[msgspec.Raw]` is, at runtime, simply an instance of `Publish`. The type parameter only exists for the type checker.



### Decoding the ROS Message Payload
The payload can be further validated inside the handler, which now has the context it needs to do so. `msgspec` allows users to decode single fields of a `msgspec.Struct`. This allows us to define a domain model (`msgspec.Struct`) to decode the ROS message:

```python
envelope: Publish = {
    topic: str = "/client_count", 
    msg: msgspec.Raw = b'{"data":3}'
}

print(envelope.msg) # b'{"data":3}'

class ClientCountMsg(msgspec.Struct):
    data: int

payload = msgspec.json.decode(envelope, type=ClientCountMsg)
print(payload) # ClientCountMsg(data=3)
```
Handlers can, for example, look up the topic, service or action in a registry that binds a ROS name to a domain model (`msgspec.Struct`) defined for the specific robot and decodes the payload:
```python
ros_topic: RosTopic = topic_registry(robot_id, envelope.topic)  # RosTopic(name="/client_count", model=ClientCountMsg)
payload = msgspec.json.decode(envelope.msg, type=ros_topic.model)
```

Note that the same topic, service or action can, in theory, carry a different payload type on different robots. The registry can handle this by keying its entries on both the robot ID and the topic name.


## Answering the Real Question 
Which brings us to the real question: should we bother decoding the payload at all? Well... maybe? As always, it depends.

Decoding the payload pays off when the server actually does something with the data. If it validates the data, acts on it, reshapes it, or hides ROS types behind its own interface, it needs a typed payload. If it only forwards the data, decoding just costs performance, and on high-bandwidth streams such as camera images or point clouds that cost can be significant. There is also a development cost to consider: every message type needs a model, and those models must stay in sync with the ROS definitions on every robot.

The right choice therefore depends on the broader context. We pilot new robot concepts in hospital environments, where the robots' interfaces change regularly. Most of our messages are small, so decoding them costs almost nothing in performance. The real cost of decoding the payload is maintenance: every interface change means updating a model, and that time is better spent on the robot concepts themselves. 

I therefore decided to make the payload decoding opt-in. By default, a topic, service or action relays its payload as raw bytes. You can opt in to decoding when a model for the payload is available.

## The Result

Every packet that reaches the Robot Fleet API is now typed from the moment it arrives, but we only pay for the validation we actually need:

- **Every packet is validated at the edge.** Malformed packets and unknown `op` values are rejected before they reach any handler.
- **Routing is typed.** Handlers are selected by envelope type rather than by string comparison, so the type checker can verify each handler's signature, and each handler has direct access to the envelope's metadata.
- **Skipping the payload is cheap.** `msgspec.Raw` keeps a zero-copy view of the original bytes, so a payload that isn't decoded costs almost nothing.
- **Payload decoding is opt-in.** Each topic, service or action decides whether its payload is decoded or relayed as raw bytes, so validation is a per-endpoint choice rather than a cost every message pays.
- **Models only where they matter.** We write and maintain models only for payloads the server actually works with, so most interface changes on the robots require no changes to the API.

The result is a small amount of extra structure, a decoder, a dispatch table and a registry, in exchange for type safety where we need it and flexibility everywhere else.
