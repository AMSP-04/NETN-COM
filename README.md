# NETN-COM
The purpose of the NATO Education and Training Network (NETN) Communication Network (COM) Module is to provide a standard way to exchange data related to Communication Networks and Physical Networks. The main objective is to provide a generic model that represents the status of connections in a communication network and links in a physical network. The Communication Networks can be set up to use radios, ethernet, satellite communication or laser-based links, transmitting from point to point or routing through the network.
        

This module is a specification of how to represent Communication Networks and related connectivity data to be shared among participants in a federated distributed simulation. The specification is based on IEEE 1516 High Level Architecture (HLA) Object Model Template (OMT) and is primarily intended to support interoperability in a federated simulation (federation) based on HLA. An HLA-based Federation Object Model (FOM) is used to specify types of data and how it is encoded on the network. The NETN-COM FOM module is available as an XML file for use in HLA-based federations.



## License

Copyright (C) 2020 NATO/OTAN. This work is licensed under a [Creative Commons Attribution-NoDerivatives 4.0 International License](LICENCE.md).

The work includes the NETN-COM.xml FOM Module and documentation.

The licence gives you the right to use and redistribute the NETN FOM Module (XML file and Documentation) in its entirety without modification. You are also allowed to develop new FOM Modules (in separate XML files and separate documentation) that build on or extend the NETN module by referencing and including necessary scaffolding classes. You are NOT allowed to modify this FOM Module or its documentation without prior permission from the NATO Modelling and Simulation Group.

## Versions, updates and extensions

All updates and versioning of this work are coordinated by the NATO Modelling and Simulation Coordination Office (MSCO), managed by the NATO Modelling and Simulation Group (NMSG) and performed as NATO Science and Technology Organization (STO) technical activities in support of the NMSG Modelling and Simulation Standards Subgroup (MS3).

Feedback on the use of this work, suggestions for improvements and identified issues are welcome and can be provided using GitHub issue tracking. To engage in the development and update of this FOM Module please contact your national NMSG representative.

Version numbering of this FOM Module and associated documentation is based on the following principles:

* A new official version number is in effect only when a new release is made in the Master branch.
* An update of the major version number is made if the change constitutes a major restructuring, merging, addition or redefinition of semantics that breaks backward compatibility or covers entirely new concepts.
* An update of the minor version number is made if the change constitutes minor additions to existing concepts and editorial changes that do not break backward compatibility but may require updates of software to use new concepts.
* Patches are released to fix minor issues that do not break backward compatibility.

|Version|
|---|
|1.0.0 - Initial version developed by MSG-164 and MSG-163 for NETN-FOM v3.0|
|1.1.0 - Version developed by MSG-191 for NATO FOM v4.0|

[Changelog](changelog.md)

## Documentation

[Full Documentation](NETN-COM.md)

