# Netzwerk Diagramm

```plantuml
@startuml
!define RECTANGLE class height 80

package "Host" {
  [Host] 
}

package "VM1" {
  [VM1]
}

package "VM2" {
  [VM2]
}

package "VM3" {
  [VM3]
}

[Host] --> [VM1] : Connection
[Host] --> [VM2] : Connection
[Host] --> [VM3] : Connection

[VM1] -- [VM2] : Own Network
[VM2] -- [VM3] : Own Network
[VM3] -- [VM1] : Own Network
@enduml

```