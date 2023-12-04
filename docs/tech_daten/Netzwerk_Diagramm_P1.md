# Netzwerk Diagramm

```plantuml
@startuml
class Host {
}

class VM1Win10 {
  IP: 192.168.56.104
      10.0.2.15
}

class VM2WinDC {
  IP: 192.168.56.102
      10.0.2.15
}

class VM3LinuxLogger {
  IP: 192.168.56.105
      10.0.2.15
}

class InternalNetwork {
}

class NatNetwork {
Gateway: 10.0.2.2
}

Host -- NatNetwork : NAT Connection
NatNetwork -- VM1Win10 : IP: 10.0.2.15
NatNetwork -- VM2WinDC : IP: 10.0.2.15
NatNetwork -- VM3LinuxLogger : IP: 10.0.2.15

VM1Win10 -- InternalNetwork : IP: 192.168.56.104
VM2WinDC -- InternalNetwork : IP: 192.168.56.102
VM3LinuxLogger -- InternalNetwork : IP: 192.168.56.105

@enduml
```