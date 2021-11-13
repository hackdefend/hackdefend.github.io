---
title: Remote Full Packet Capture 
classes: wide
categories: 
  - Golang 
  - tools
tags:
  - SIEM
---

The requirement was to create a tool that can stream captured packets to a remote machine. There are many tools that monitor traffic and send aggregated logs, however the goal is the packet payload inclusion. Generally, network forensics requires looking into payload network streams for deep analysis or file extraction. In addition, there are troubleshooting situations where full packet capture is needed. Wireshark will not be a solution to capture on teh remote server and manually transfer the file.

<!-- more --> 

The tool is intended be to used mainly for capturing traffic from compromised machines or for troubleshooting remote servers. There might be ways to configure Wireshark to send traffic to a remote destination, but not scalable if multiple machines needed to send the traffic at the same time.  


### Design 

- Agent to send logs 
- Collector server to receive the packets from the agent
- BPF support
- Capture stops by condition based on number of packet or size.
- Multiple machine capture support
- Arkime full packet capture platform support at collector server


Golang programming language was selected for writing the agent and collector server, because it's a compiled language and has a robust packet processing libraries. Python has also strong packet processing packages, but will be a challenge to run a python script on a remote machine. 

Two main libraries used to achieve the goal are **GoPacket** as a packet processing library and **gRPC** for sending the serialized packets to a remote destination. 

## Implementation 

gRPC uses [Proto3 (Protocol Buffers)](https://developers.google.com/protocol-buffers/docs/proto3) language to describe the services that will be implemented server and client sides. Writing Proto3 services file will enable developers to auto-generate clients for any language supported by gRPC without writing the code. 

![gRPC](/imgs/grpc-proto_concept.png)
[img source](https://developer.token.io/tailrd_rest_api_doc/content/0-token_fundamentals/proto_buffers.htm)

The tool is fairly simple, it has only 3 messages and on one services consisting of two functions. Details of how to write proto3 can be found in the [language guide](https://developers.google.com/protocol-buffers/docs/proto3).    

```
syntax = "proto3";
package service;
option go_package = "service/;service";

message Packet{
    bytes Data = 1; 
    bytes Seralizedcapturreinfo = 2; 
}

message EndpointInfo{
    string Hostname = 1;
    string IPaddress = 2;
    string Interface = 3;
}

message Empty {
    string okay = 1;
}

service RemoteCaputre {
    rpc Capture (stream Packet) returns (Empty) {}
    rpc GetReady(EndpointInfo) returns (Empty)  {}

}
```

Using above proto3 code, a client can be generated containing golang code required to send and receive data between client and server. Google Protobuf compiler generated 470 lines of code that has all code necessary to exchange messages between client and server, the code must be imported by the client and server code being written around the generated Protbuf code. Applications with few functions will have thousands of lines automatically generated with zero effort. The same can be generated for any language. 


GoPacket can capture traffic directly from physical network interface. Interestingly almost all Golang libraries documentation contains useful examples, which helped me understanding how to use the tools more than any other language. GoPacket library has [examples folder](https://github.com/google/gopacket/tree/master/examples) in the official github repository.

For more clean code snippet, check the examples folder. Remote capture tools code snippets will be used in the post rather than showing snippets that are not directly related to the post. 


**Windows NPF interface naming**

Linux has simple ifconfig command that shows the interface names such as eth0, but Windows has more difficult ways to know what is the name of the interface. In order to capture packets from windows, you need to know the name of the interface which is not what it's shown in the output of ipconfig or getmac. [Golang and Windows Network Interfaces](https://haydz.github.io/2020/07/06/Go-Windows-NIC.html) post discuss the naming in more details. 

Luckily there is pcap.FindAllDevs() function in GoPacket to find all the physical interfaces. Once the names are fetched, it's the best to leave the user decides what interface to capture by mapping the NPF naming with with the friendly names such as wireless or ethernet. 

```golang
if *listNICsOption {
  count := 0
  devices, err := pcap.FindAllDevs()
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println("Devices found:")
  if OS == "Windows" {
    fmt.Println("")
    for _, device := range devices {
      count++
      fmt.Printf("(%d)- %s:%s\n", count, device.Description, device.Name)
    }
    os.Exit(0)
  } else {
    for _, device := range devices {
      count++
      fmt.Printf("(%d)- %s\n", count, device.Name)
    }
    os.Exit(0)
  }
}
```

Output of above code will show the interfaces in numbers so the user can select the correct number, same approach is in tcpdump tool. 

![NPF Naming](/imgs/NPF_names.png)


**Packet Capture**

The code to capture is a more complex than below one because of the feature needed such as verbose logging, size and packet tracking, error handling ..etc, but for a show case, it's as simple as below one. 

```golang
handle, err = pcap.OpenLive(deviceName, snapshotLen, promiscuous, timeout)
if err != nil {
  fmt.Printf("Error opening device %s: %v", deviceName, err)
  os.Exit(1)
  }

  defer handle.Close()
  packetSource := gopacket.NewPacketSource(handle, handle.LinkType())

  if err != nil {
    panic("No interface found")
  }

  packets := packetSource.Packets()

  for {

    select {
    case packet := <-packets:
      if packet.NetworkLayer() == nil || packet.TransportLayer() == nil {
        continue
      }

      data := packet.Data()

      byteArray, err := json.Marshal(packet.Metadata())
      if err != nil {
        fmt.Println(err)
      }
      pkt := service.Packet{
        Data:                  data,
        Seralizedcapturreinfo: byteArray,
      }

      sendchan <- pkt
    }

  }

```

Json marshal is used to serialize packet data and Metadata into bytes for the gRPC transfer. Golang channels passes data around the code. Using channels in the code is enough for Golang endless love. In the same code, network handling and packet manipulation can be achieved by modifying layers and protocol events. Since the tool is created for packet with payload (raw packets), no manipulation was done. 


**Sending packets over gRPC**


```golang
conn, err := grpc.Dial(*serverIP+":9000", grpc.WithInsecure(), grpc.WithBlock())
if err != nil {
  log.Fatalf("can not connect with server %v", err)
}
defer conn.Close()

ctx, cancel := context.WithTimeout(context.Background(), time.Minute)
defer cancel()

// create gRPC client
client := service.NewRemoteCaputreClient(conn)

hostname, _ := os.Hostname()
IP, err := GetIpByInterface(deviceName)
if err != nil {
  fmt.Println(err.Error())
  os.Exit(1)
}

e := service.EndpointInfo{IPaddress: IP, Hostname: hostname, Interface: deviceName}
_, err = client.GetReady(ctx, &e)
if err != nil {
  fmt.Println(err)
}

ServerStream, err := client.Capture(context.Background())
if err != nil {
  log.Fatalf("open stream error %v", err)
}

go func() {
  for {
    select {
    case pkt := <-sendchan:
      err = ServerStream.Send(&pkt)
      if err == io.EOF {
        fmt.Printf("\nReceived EOF: %v\n", err)

      } else if err != nil {
        log.Fatalf("can not send %v", err)
      }
    }
  }
}()
```


**BPF filters**

The agent can be converted into a systemd service or windows service for constant transmission, caution needed to not over load the network when capturing multiple servers and sending the traffic to one destination. BPF should be used to narrow down the scope of the capture.

It's possible to run the tools with no filters to capture everything, BPF filters can be compiled and used with GoPacket to avoid capturing unnecessary traffic. I rely on Alexa top 1000 website rank + organization internal and business related domains in one huge BPF. 

```golang 
if *whitelisting || *captureFilter != "" {
  buildFilter()
  if err := handle.SetBPFFilter(whitelistFilter); err != nil {
    log.Fatal(err)
  }

} else {
  fmt.Println(*captureFilter)
  whitelistFilter = fmt.Sprintf("not host %s", *serverIP)
  if err := handle.SetBPFFilter(whitelistFilter); err != nil {
    log.Fatal(err)
  }

}
```


**BPF construction**

It may be needed to resolve domains to IP Addresses if supplied whitelist is not IP Addresses, The list is feteched from server during the initialization of the agent. 

```golang
func buildFilter() {
  count := 0
  resp, err := http.Get(fmt.Sprintf("http://%s:8080/exceptions.list", *serverIP))
  if err != nil {
    log.Fatalln(err)
  }

  // don't capture yourself 
  whitelistFilter = fmt.Sprintf("not host %s ", *serverIP)

  scanner := bufio.NewScanner(resp.Body)
  defer resp.Body.Close()

  for scanner.Scan() {
    line := scanner.Text()

    if valid.IsDNSName(line) {
      whitelistFilter += fmt.Sprintf("and not host %s ", line)
  }

  whitelistFilter += *captureFilter
}

```


**Collector server implementation**

Writing the received packets to disk will not be useful if not supported by a searching platform. The first thought was to use Security Onion and schedule a cron job that will tcpreplay the packets every N minutes. Fortunately I found an interesting option in Arkime (formerly Moloch) to monitor a folders for pcaps and automatically ingest them to Arkime.

![gRPC](/imgs/streaming_packets.png)  


**gRPC server**

gRPC server listens on TCP Port 9000, file server part is used to start a server for public folder where it hosts couple of txt files that acts like settings api. Agents will read the files to know what BPF filters to apply from the public folder. 

```golang
lis, err := net.Listen("tcp", "0.0.0.0:9000")
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}

// Serve the exception list over http
go func() {

  fs := http.FileServer(http.Dir("./public"))
  http.Handle("/", fs)

  log.Println("Listening on :8080...")
  err := http.ListenAndServe(":8080", nil)
  if err != nil {
    log.Fatal(err)
  }
}()

grpcserver := grpc.NewServer()
service.RegisterRemoteCaputreServer(grpcserver, &Server{})
fmt.Println("Server started. ")
if err := grpcserver.Serve(lis); err != nil {
  log.Fatalf("failed to serve: %v", err)

}
```


The received packets will be written to disk using a special GoPacket writer. 

```golang
go func() {
  for {
    pkt, err := srv.Recv()
    if err != nil {
      StreamEnd <- true
      break
    }
    metadata := gopacket.PacketMetadata{}
    err = json.Unmarshal(pkt.Seralizedcapturreinfo, &metadata)
    if err != nil {
      fmt.Printf("Error unmarshal the packet %s \n", err)
      continue
    }
    err = w.WritePacket(metadata.CaptureInfo, pkt.Data)
    if err != nil {
      fmt.Println(err)
    }
    endpoints[endpoint].Packetcount++
    fmt.Printf("Received...\nPacketCount: %d ", endpoints[endpoint].Packetcount)
  }
}()
```
<br>
Packets should appear in Arkime platform for packet analysis or download. 