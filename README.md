# SSH MicroService

Use gRPC to manage remote SSH clients

[![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/kainonly/ssh-microservice?style=flat-square)](https://github.com/kainonly/ssh-microservice)
[![Github Actions](https://img.shields.io/github/workflow/status/kainonly/ssh-microservice/release?style=flat-square)](https://github.com/kainonly/ssh-microservice/actions)
[![Image Size](https://img.shields.io/docker/image-size/kainonly/ssh-microservice?style=flat-square)](https://hub.docker.com/r/kainonly/ssh-microservice)
[![Docker Pulls](https://img.shields.io/docker/pulls/kainonly/ssh-microservice.svg?style=flat-square)](https://hub.docker.com/r/kainonly/ssh-microservice)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/kainonly/ssh-microservice/master/LICENSE)

## Setup

Example using docker compose

```yaml
version: "3.8"
services: 
  ssh:
    image: kainonly/ssh-microservice
    restart: always
    volumes:
      - ./ssh:/app/config
    ports:
      - 6000:6000
```

## Configuration

For configuration, please refer to `config/config.example.yml` and create `config/config.yml`

- **debug** `string` Turn on debugging, that is `net/http/pprof`, and visit the address `http://localhost: 6060/debug/pprof`
- **listen** `string` Microservice listening address

## Service

The service is based on gRPC to view `api/api.proto`

```protobuf
syntax = "proto3";
package ssh;
import "google/protobuf/empty.proto";

service API {
  rpc Testing (Option) returns (google.protobuf.Empty) {}
  rpc Put (IOption) returns (google.protobuf.Empty) {}
  rpc Exec (Bash) returns (Output) {}
  rpc Delete (ID) returns (google.protobuf.Empty) {}
  rpc Get (ID) returns (Data) {}
  rpc All (google.protobuf.Empty) returns (IDs) {}
  rpc Lists (IDs) returns (DataLists) {}
  rpc Tunnels (TunnelsOption) returns (google.protobuf.Empty) {}
  rpc FreePort (google.protobuf.Empty) returns (Port) {}
}

message Option {
  string host = 1;
  uint32 port = 2;
  string username = 3;
  string password = 4;
  string private_key = 5;
  string passphrase = 6;
}

message IOption {
  string id = 1;
  Option option = 2;
}

message Bash {
  string id = 1;
  string bash = 2;
}

message Output {
  bytes data = 1;
}

message ID {
  string id = 1;
}

message Data {
  string id = 1;
  string host = 2;
  uint32 port = 3;
  string username = 4;
  string connected = 5;
  repeated Tunnel tunnels = 6;
}

message Tunnel {
  string src_ip = 1;
  uint32 src_port = 2;
  string dst_ip = 3;
  uint32 dst_port = 4;
}

message IDs {
  repeated string ids = 1;
}

message DataLists {
  repeated Data data = 1;
}

message TunnelsOption {
  string id = 1;
  repeated Tunnel tunnels = 2;
}

message Port {
  uint32 data = 1;
}
```

#### rpc Testing (Option) returns (google.protobuf.Empty) {}

Test for ssh client connection

- **Option**
  - **host** `string`
  - **port** `uint32`
  - **username** `string`
  - **password** `string` password (default empty)
  - **private_key** `string` private key (Base64)
  - **passphrase** `string` key passphrase (Base64)

```golang
client := pb.NewRouterClient(conn)
response, err := client.Testing(
  context.Background(),
  &pb.Option{
    Host:       debug.Host,
    Port:       debug.Port,
    Username:   debug.Username,
    Password:   debug.Password,
    PrivateKey: debug.PrivateKey,
    Passphrase: debug.Passphrase,
  },
)
```

#### rpc Put (IOption) returns (google.protobuf.Empty) {}

Update the ssh client configuration to the service

- **IOption**
  - **id** `string` ID
  - **option** `Option`
    - **host** `string`
    - **port** `uint32`
    - **username** `string`
    - **password** `string` password (default empty)
    - **private_key** `string` private key (Base64)
    - **passphrase** `string` key passphrase (Base64)

```golang
client := pb.NewRouterClient(conn)
response, err := client.Put(
  context.Background(),
  &pb.IOption{
    Id: "debug",
    Option: &pb.Option{
      Host:       debug.Host,
      Port:       debug.Port,
      Username:   debug.Username,
      Password:   debug.Password,
      PrivateKey: debug.PrivateKey,
      Passphrase: debug.Passphrase,
    },
  },
)
```

#### rpc Exec (Bash) returns (Output) {}

Send commands to the server via ssh

- **Bash**
  - **id** `string` ID
  - **bash** `string` shell command
- **Output**
  - **data** command output result

```golang
client := pb.NewRouterClient(conn)
response, err := client.Exec(
    context.Background(),
    &pb.ExecParameter{
        Identity: "test",
        Bash:     "uptime",
    },
)
```

#### rpc Delete (ID) returns (google.protobuf.Empty) {}

Remove an ssh client from the service

- **ID**
  - **id** `string` ID

```golang
client := pb.NewRouterClient(conn)
response, err := client.Delete(
  context.Background(),
  &pb.ID{
    Id: "debug",
  },
)
```

#### rpc Get (ID) returns (Data) {}

Get the details of an ssh client from the service

- **ID**
  - **id** `string` ID
- **Data**
  - **id** `string` ID
  - **host** `string`
  - **port** `uint32`
  - **username** `string`
  - **connected** ssh connected client version
  - **tunnels** `[]Tunnel` ssh tunnels
    - **src_ip** `string` origin ip
    - **src_port** `uint32` origin port
    - **dst_ip** `string` target ip
    - **dst_port** `uint32` target port

```golang
client := pb.NewRouterClient(conn)
response, err := client.Get(
  context.Background(),
  &pb.ID{
    Id: "debug",
  },
)
```

#### rpc All (google.protobuf.Empty) returns (IDs) {}

Get all ssh client IDs from the service

- **IDs**
  - **ids** `[]string` IDs

```golang
client := pb.NewRouterClient(conn)
response, err := client.All(
  context.Background(),
  &empty.Empty{},
)
```

#### rpc Lists (IDs) returns (DataLists) {}

Get the specified list ssh client details from the service

- **IDs**
  - **ids** `[]string` IDs
- **DataLists** 
  - **data** `[]Data`
    - **id** `string` ID
    - **host** `string`
    - **port** `uint32`
    - **username** `string`
    - **connected** ssh connected client version
    - **tunnels** `[]Tunnel` ssh tunnels
      - **src_ip** `string` origin ip
      - **src_port** `uint32` origin port
      - **dst_ip** `string` target ip
      - **dst_port** `uint32` target port

```golang
client := pb.NewRouterClient(conn)
response, err := client.Lists(
  context.Background(),
  &pb.IDs{
    Ids: []string{"debug", "debug-next"},
  },
)
```

#### rpc Tunnels (TunnelsOption) returns (google.protobuf.Empty) {}

Set up a tunnel for the ssh client

- **TunnelsOption**
  - **id** `string` ID
  - **tunnels** `[]Tunnel` ssh tunnels
    - **src_ip** `string` origin ip
    - **src_port** `uint32` origin port
    - **dst_ip** `string` target ip
    - **dst_port** `uint32` target port

```golang
client := pb.NewRouterClient(conn)
response, err := client.Tunnels(
  context.Background(),
  &pb.TunnelsOption{
    Id: "debug-1",
    Tunnels: []*pb.Tunnel{
      {
        SrcIp:   "127.0.0.1",
        SrcPort: 9200,
        DstIp:   "127.0.0.1",
        DstPort: 9200,
      },
      {
        SrcIp:   "127.0.0.1",
        SrcPort: 5601,
        DstIp:   "127.0.0.1",
        DstPort: 5601,
      },
    },
  },
)
```

#### rpc FreePort (google.protobuf.Empty) returns (Port) {}

Get available ports on the host

- **Port**
  - **data** `uint32` port

```golang
client := pb.NewRouterClient(conn)
response, err := client.FreePort(
  context.Background(),
  &empty.Empty{},
)
```
