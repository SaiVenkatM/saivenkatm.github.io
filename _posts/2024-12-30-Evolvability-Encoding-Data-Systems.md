---
layout: post
title: Evolvability - Building Resilient Systems
subtitle: Ensuring Seamless Data Exchange in an Ever-Evolving Digital World
#cover-img: /assets/img/tcover.jpg
#thumbnail-img: /assets/img/tcover3.jpg
#share-img: /assets/img/tcover2.jpg
tags: [Dataflow, Data Systems, Encoding, Protocol Buffers ]
author: Sai Venkat Malreddy
---

### Understanding Encoding, Evolvability, and Compatibility in Modern Systems

In the realm of data-intensive applications, change is a constant. As Heraclitus said, "Everything changes and nothing stands still." This truth reverberates in software systems, where features evolve, user requirements mature, and business landscapes shift. The ability to adapt gracefully to these changes is encapsulated in the concept of **evolvability** — designing systems that make change easy. Encoding, forward compatibility, and backward compatibility are essential tools in achieving this goal. 

This blog delves into encoding mechanisms, their significance, and protocols like Apache Thrift, Protocol Buffers, and Avro. We’ll explore their practical applications, real-world scenarios, and how they empower systems to evolve seamlessly.

---

### Encoding: The Language of Data Exchange

Encoding bridges the gap between in-memory data structures and their external representations, allowing systems to save data to files or transmit it over networks. This process, known as serialization or marshalling, transforms complex data structures into self-contained byte sequences, while decoding (deserialization or unmarshalling) performs the reverse.

#### Why Encoding Matters
In modern distributed systems, data exchange between heterogeneous components is inevitable. Encoding ensures:
1. **Interoperability:** Different systems and programming languages can understand the shared data.
2. **Efficiency:** Encoded data is compact, enabling faster transmission and reduced storage requirements.
3. **Versioning:** Smooth schema evolution without disrupting older systems or data.

Real-World Scenario:
Imagine a payment processing system where a mobile app communicates with backend servers to validate transactions. The app’s older version might transmit payment details in a simpler format, while newer server versions might handle more complex details. Encoding ensures seamless communication across versions.

---

### Evolvability and Compatibility: The Twin Pillars

#### Forward Compatibility
Forward compatibility enables older systems to read data written by newer systems. This ensures uninterrupted functionality even as the system evolves.

#### Backward Compatibility
Backward compatibility ensures that newer systems can read data written by older systems, accommodating legacy applications.

These forms of compatibility are crucial for systems undergoing rolling updates, where old and new versions coexist. For example, in a social media platform, user profile updates might include new fields like “pronouns” while retaining compatibility with older API versions used by third-party applications.

---

### Encoding Formats: JSON, XML, and Beyond

#### JSON and XML
These human-readable formats are widely used due to their simplicity and ubiquity. However, they have limitations like verbosity, ambiguities in numeric representations, and inefficiency for large-scale data.

Real-World Example:
Twitter’s API provides tweet IDs as both JSON numbers and strings to mitigate issues with JavaScript’s inability to handle 64-bit integers accurately.

#### Binary Encoding Formats
Binary formats like **MessagePack**, **BSON**, and others address JSON/XML inefficiencies. However, protocols like **Apache Thrift**, **Protocol Buffers (Protobuf)**, and **Avro** elevate encoding by introducing schemas for compactness, clarity, and evolution support.

---

### Protocols for Schema-Based Encoding

#### Apache Thrift
- Developed at Facebook, Thrift uses schemas to define data structures.
- Supports multiple languages, making it ideal for polyglot environments.
- CompactProtocol and BinaryProtocol offer varying levels of compactness and parsing efficiency.




Each field in Thrift’s BinaryProtocol is encoded with a type descriptor, a tag number, and the field value. This enables compact encoding and robust schema evolution.

#### Protocol Buffers (Protobuf)
- Created by Google, Protobuf focuses on simplicity and performance.
- Tag numbers replace field names in binary encoding, reducing size.
- Ideal for high-performance systems like chat applications or IoT devices.



Protobuf uses compact field tags to identify fields, ensuring minimal overhead. The value is efficiently stored without unnecessary metadata.

#### Avro
- A Hadoop subproject, Avro integrates seamlessly with big data ecosystems.
- It omits tag numbers, relying on schemas for compactness.
- Avro’s dynamic schema resolution simplifies database exports and archival storage.



Avro relies on schema consistency between writer and reader. This enables compact encoding by omitting field identifiers, relying entirely on predefined order.

---

### Schema Evolution: Navigating Change
Schema evolution allows systems to:
1. **Add fields**: New fields with default values ensure backward compatibility.
2. **Remove fields**: Optional fields can be safely deprecated.
3. **Change data types**: Limited by compatibility requirements.

#### Handling Unknown Fields
Encoding protocols ensure that unknown fields in schemas are preserved during updates. This prevents data loss when older applications interact with newer datasets.

---




### Real-World Scenario: API Evolution in Microservices
Consider a user management microservice that handles authentication across multiple applications. Initially, the user profile only contained basic fields like name and email. As the system evolved, new requirements demanded additional fields for two-factor authentication and social media integration.

To manage this evolution:
- Services use Protocol Buffers for API contracts
- Legacy clients continue to function with basic user profiles
- Newer clients access enhanced authentication features
- Rolling updates maintain system stability

#### 1. Create the protocol buffer definitions:

**v1:**

```protobuf
syntax = "proto3";

package userservice;

service UserService {
    rpc GetUser (GetUserRequest) returns (User) {}
}

message GetUserRequest {
    string user_id = 1;
}

message User {
    string id = 1;
    string email = 2;
    string name = 3;
}
```

**Evolution to v2 with new fields:**

```protobuf
syntax = "proto3";

package userservice;

service UserService {
    rpc GetUser (GetUserRequest) returns (User) {}
    rpc UpdateUserPreferences (UpdatePreferencesRequest) returns (User) {}
}

message GetUserRequest {
    string user_id = 1;
}

message UpdatePreferencesRequest {
    string user_id = 1;
    UserPreferences preferences = 2;
}

message User {
    string id = 1;
    string email = 2;
    string name = 3;
    UserPreferences preferences = 4;  // New field
    repeated string roles = 5;        // New field
}

message UserPreferences {
    bool dark_mode = 1;
    string language = 2;
    repeated string notifications = 3;
}
```

#### 2. Server implementation:

```python
from concurrent import futures
import grpc
import user_service_pb2
import user_service_pb2_grpc

class UserServicer(user_service_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        # V2 server can handle both old and new clients
        user = user_service_pb2.User(
            id=request.user_id,
            email="user@example.com",
            name="Test User",
            preferences=user_service_pb2.UserPreferences(
                dark_mode=True,
                language="en"
            ),
            roles=["user"]
        )
        return user

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_service_pb2_grpc.add_UserServiceServicer_to_server(
        UserServicer(), server
    )
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

#### 3. V1 Client example:

```python
import grpc
import user_service_pb2
import user_service_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = user_service_pb2_grpc.UserServiceStub(channel)
        response = stub.GetUser(user_service_pb2.GetUserRequest(user_id="123"))
        # V1 client only uses id, email, and name fields
        print(f"User: {response.name} ({response.email})")

if __name__ == '__main__':
    run()
```

### This demonstrates:
- **Backward compatibility**: V1 clients work with V2 server
- **Forward compatibility**: V2 server can handle V1 requests
- **Schema evolution**: New fields added without breaking existing clients
- **Protocol Buffers handling unknown fields automatically**

### To run:
1. Generate Python code from protos
2. Start server
3. Run either V1 or V2 client


---

### Conclusion
Encoding, evolvability, and compatibility form the bedrock of resilient, scalable systems. Protocols like Thrift, Protobuf, and Avro empower developers to manage complexity, support diverse languages, and evolve schemas without disrupting operations. By adopting these practices, organizations can future-proof their systems and ensure seamless data exchange in an ever-changing digital landscape.

#### References

Kleppmann, Martin. Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. O'Reilly Media, 2017.

