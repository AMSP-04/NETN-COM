# NETN-COM
The purpose of the NATO Education and Training Network (NETN) Communication Network (COM) Module is to provide a standard way to exchange data related to Communication Networks and Physical Networks. The main objective is to provide a generic model that represents the status of connections in a communication network and links in a physical network. The Communication Networks can be set up to use radios, ethernet, satellite communication or laser-based links, transmitting from point to point or routing through the network.
        
This module is a specification of how to represent Communication Networks and related connectivity data to be shared among participants in a federated distributed simulation. The specification is based on IEEE 1516 High Level Architecture (HLA) Object Model Template (OMT) and is primarily intended to support interoperability in a federated simulation (federation) based on HLA. An HLA-based Federation Object Model (FOM) is used to specify types of data and how it is encoded on the network. The NETN-COM FOM module is available as an XML file for use in HLA-based federations.

## Overview
The NETN-COM module distinguishes between three layers of networks.

1. **Application Layer:** This is the topmost layer and corresponds to OSI layers 5-7. This layer implemented by simulations internally and has no specific representation in the NETN-COM object model except the optional `CommunicationNetwork` object. Applications use communication networks to exchange data via connections. 
2. **Connection Layer:** This layer corresponds to OSI layers 3-4 and describes the connections associated with communication networks. `CommunicationNode` objects are associated with simulated entities and connected to form an information-sharing space.
3. **Link Layer:** This layer corresponds to OSI layers 1-2 and is used to define the link quality parameters between nodes to form a physical network.

The following definition of terms are used in the NETN-FOM module:

* **Entity:** A entity is either a simulated physical or an aggregated entity.
* **Communication network:** A group of communication nodes exchanging messages using a uniquely named logical network which is independent of the physical network implementation. A communication network is the foundation of a shared information space.
* **Communication node:** The representation of the interface of a simulated entity to logical communication networks. A communication node describes the relationship between an entity to all communication networks it uses and the network devices used to establish a connection to other communication nodes.
* **Connection:** A connection describes the relationship of at least two communication nodes in a communication network.
* **Network device:** The technical device (e.g. radio or ethernet) connecting an entity to a physical network to participate in one or more communication networks.
* **Physical network:** The physical medium used to implement the connection between nodes.
* **Link:** The physical connection between two nodes of a physical network. A link describes the relationship between a transmitting and a receiving network device.

<img src="./images/Concepts.png" width="100%"/>
Figure. Concept Relationships


By separating the representation of the physical- and communication network layers, different simulations can be used to model the system on different levels, e.g. radio signal propagation simulations for the link layer and an ad hoc network routing simulation on the connection layer.

The model does not require all levels and networks to be represented in the federation. Which objects are needed depends on the federation design and allocation of modelling responsibilities. E.g. at the application layer, only the connection information needs to be published.

<img src="./images/objectclasses.png" width="85%"/>

Figure. Object Classes

The application layer should only use the `CommunicationNode` and `Connection` objects to determine if data can be sent or received. If no `Connection` object exists or if the `Receivers` attribute is empty, the sender can drop the data entirely. A receiver should drop any incoming messages if not included in the `Receivers` attribute of the corresponding `Connection` object. If the `IncommingConnections` attribute of a `CommunicationNode` is calculated for a receiver, it can be used to determine if incoming data should be dropped or not.

## Object Classes 

### CommunicationNetwork

The `CommunicationNetwork` class specifies details about the type of service used by a logical communication network. This corresponds to the CommunicationNet-Elements of MSDL Units and Equipments. 
Instances of this object class should be considered optional. No federate should rely on this data to work with communication networks.
    
|Attribute|Semantics|
|---|---|
|NetworkType|Optional. The communication network type of use.|
|ServiceType|Optional. The type of service used on the communication network.|
|UniqueName|Required. Unique communication network name. Unique in the context of communication networks.|

### CommunicationNode

A `CommunicationNode` is the representation of the interface of a simulated entity to logical communication networks. The location of the `CommunicationNode` is derived from the referenced entity or specified explicitly (if the referenced entity is not registered in the federation). 

Each potential connection is described in terms of requested connections (the equivalent to the information provided by MSDL). `Connection` objects represent the availability of a connection and its receivers. It is up to a communication simulation to create `Connection` objects for the requested connections based on the network devices of the `CommunicationNode`, the physical network and the link quality between network devices. Depending on the federation design and agreements, the explicit representation of the physical layer objects, `PhysicalNetwork` and `LinkStates`, is optional.

A requested connection is used to describe the characteristics of a possible connection to a communication network. For the types of connections where data is intended to be transmitted, a connection identifier can be provided that can be used to create a `Connection` object instance. 

Unless a network device is explicitly specified, all suitable devices associated with the communication network of the requested connection shall be used. 

Depending on the type of the requested connection the use of the associated network device parameters differs:

* **Broadcast Transceiver**: A combination of Broadcast Transmitter and Receiver
* **Broadcast Transmitter**:
  * If TX is specified for a network device, connections are established to all receiving entities reachable depending on the physical network description (ranges, max hops, etc.). Data is sent to all (directly linked) receivers simultaneously.
  * A Connection instance should be created.
  * The DestinationEntityArray is ignored.
* **Broadcast Receiver**:
  * If RX is specified for a network device, the connections listen to all incoming messages. 
  * The DestinationEntityArray is ignored.
* **Peer-To-Peer**:
  * If TX is specified for a network device connections are established to all reachable specified destination entities.
  * If RX is specified for a network device, the connections listen to all incoming messages which are sent to the entity.
  * It is assumed that the communication is bidirectional and should use the same route in both directions. The connection must be defined at the receiver as well.
  * If DestinationEntityArray is empty messages will be sent to all reachable participants of the communication network. It is strongly recommended to define a limited set of destinations wherever possible.
  * It is assumed that messages are transmitted sequentially to all receivers defined as a destination. (Be aware: this does not include intercepting devices)
  * Any max. hop count / TTL parameter of the physical network is ignored.
* **Unidirectional**: Like Peer-To-Peer but no connection back to the source with the same route is expected. 
* **Multicast**:
  * The connection logic is similar to unidirectional connections.
  * Data is sent to all receivers simultaneously.
  * The DestinationEntityArray is ignored.
* **Intercepting connections**: 
  * These are special connections not related to specific destination entities but intercept all types of connections that are reachable by the corresponding device. In the case of broadcasts, this corresponds to a broadcast receiver, otherwise, this means the connection is routed through the corresponding device.
  * The DestinationEntityArray is ignored.

Before any data can be sent using a connection, it must have been requested, and the corresponding `Connection` object instance must exist in the federation.

|Attribute|Semantics|
|---|---|
|EntityId|Required. Reference by UUID to an object instance of one of the following classes: <br/> <br/>* NETN-MRM Aggregate Entity <br/>* NETN-Physical extensions of RPR Physical Entities. <br/> <br/>If the referenced entity exists in the federation, the location of the node is derived from the location of the entity. <br/>If the referenced entity does not exist in the federation, the location of the node is defined by the Location attribute.|
|IncomingConnections|Optional. Description of all incoming connections to the receiving entity.|
|Location|Optional. Specifies the location of the `CommunicationNode` in case the entity referenced by EntityId is not registered in the federation. If the entity referenced by EntityId exists in the federation, the location of the communication node is derived from that entity and the value of the Location attribute shall be ignored.|
|NetworkDevices|Required. Available network devices define the association of a communication network (connection layer) with a physical network (link layer). Each network device can be associated with several communication networks but only one physical network. Each network device also describes the transmitter and receiver capabilities.|
|RequestedConnections|Required. Possible (requested) connections for the communication node.|

### Connection

A connection object describes the communication capability of each entity to all other entities for a communication network. The `Connection` object is created for each requested connection if there is at least one reachable receiver. The federation agreement defines which system is responsible for creating and maintaining the connection instance. The connection describes the connectivity but not the routes through a physical network defined by links between nodes. 
    
|Attribute|Semantics|
|---|---|
|CommunicationNetworkName|Required. A reference to the communication network this connection belongs to.|
|Receivers|Required. Characteristics of the connections to individual receiving entities.|
|SenderEntityId|Required. A reference to the entity sending data using this connection.|

### PhysicalNetwork

The `PhysicalNetwork` object class represents type-specific parameters/constraints for a physical network. Physical networks are simulated by `LinkState` objects and do not require that an instance of PhysicalNetwork exist in the federation.
    
|Attribute|Semantics|
|---|---|
|Description|Required. Characteristics of the physical network.|
|UniqueName|Required. Unique physical network name. Uniqueness in the context of physical networks.|
### LinkStates

A `LinkStates` object describes the presence and quality of a set of direct links between nodes on a physical level. Links are defined between a transmitting and receiving device. 

Bidirectional links assume identical link quality in both directions. Otherwise, two unidirectional links should be described. 

Each `LinkStates` instance is associated with one physical network. To support load balancing in link quality simulation, multiple `LinkStates` instances can be associated with the same physical network as long as they do not contain duplicated link-state information.

A link should not be contained in more than one `LinkStates` object. If a link is not contained in any instance, it is not present, and it means it could not be established.

The `LinkStates` provides information that can be used to calculate connections.
    
|Attribute|Semantics|
|---|---|
|Links|Required. Status of a set of physical network links.|
|PhysicalNetworkName|Required. Reference to a physical network by its unique name.|
### DisruptionEffect

The `DisruptionEffect` object class is used to represent the disruption of connections between communication nodes in an affected area. Communication models can subscribe to disruption-effect objects and use the information to simulate the effect of disruption on communication nodes, connections and link states.
    
|Attribute|Semantics|
|---|---|
|AffectedArea|Optional. The area affected by the disruption. If not provided the default is a global disruption and all connections are affected.|
|AffectedNetworks|Optional. Names of all affected communication networks. If not provided all networks in the specified area are affected.|
|Effect|Required. Level of disruption. 100% equals No connectivity and 0% no disruption effect. The level of disruption can vary over time.|

## Datatypes 

|Name|Semantics|
|---|---|
|BitsPerSecond|Data transfer rate.|
|CommunicationNetworkArray|References to a set of communication networks.|
|CommunicationNetworkTypeEnum|The type of communication network.|
|CommunicationServiceTypeEnum|The type of service used on a communication network. Service types as defined in MSDL. DATTRF: Data transfer FAX: Facsimile IFF: Identify Friend or Foe IMAGE: Image MCI: Multilateral Interoperability Programme (MIP) Common Interface Service MHS: Message Handling Service TDL: Tactical Data Link VIDSVC: Video Service VOCSVC: Voice Service NOS: Not Otherwise Specified|
|ConnectionReceiverArray|Connection characteristics for a number of receivers.|
|ConnectionReceiverStruct|Characteristics of a connection to a receiver.|
|ConnectionTypeEnum|The type of connection.|
|IncomingConnectionArray|A set of incoming connections.|
|IncomingConnectionStruct|Characteristics of a specific incoming connection.|
|LinkStatusArray|The status of physical network links.|
|LinkStatusStruct|Status of a physical network link.|
|NetworkDeviceArray|A set of network devices.|
|NetworkDeviceEmptyCharactersticsStruct|Empty placeholder for future network device characteristics.|
|NetworkDeviceGenericTransmitterCharacteristicsStruct|Generic description of network device transmitter.|
|NetworkDeviceReceiverCharacteristicsVariant|Characteristics of a network device receiver.|
|NetworkDeviceStruct|Describes the type and capability of a network device. Each device must have at least a transmitter (TX) or receiver (RX). Transceivers should define both.|
|NetworkDeviceTransmitterCharacteristicsVariant|Characteristics of network device transmitter.|
|PhysicalGenericNetworkStruct|Characteristics of a generic physical network.|
|PhysicalNetworkDescriptionVariant|Characteristics of physical networks depend on their type.|
|PhysicalNetworkTypeEnum|The type of a physical network.|
|PhysicalUndefinedNetworkStruct|(Empty) Characteristics of an undefined physical network.|
|RequestedConnection|Characterisation of a requested connection.|
|RequestedConnectionArray|Characterisation of a set of requested connections.|