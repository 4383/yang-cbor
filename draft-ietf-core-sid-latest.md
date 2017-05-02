﻿---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-sid-01
title: YANG Schema Item iDentifier (SID)
area: Applications and Real-Time Area (art)
wg: Internet Engineering Task Force
kw: CBOR
cat: std
pi:
  strict: 'yes'
  toc: 'yes'
  tocdepth: '4'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  subcompact: 'no'
author:
- role: editor
  ins: M. V. Veillette
  name: Michel Veillette
  org: Trilliant Networks Inc.
  street: 610 Rue du Luxembourg
  code: J2J 2V2
  city: Granby
  region: Quebec
  country: Canada
  phone: "+14503750556"
  email: michel.veillette@trilliantinc.com
- role: editor
  ins: A. P. Pelov
  name: Alexander Pelov
  org: Acklio
  street: 2bis rue de la Chataigneraie
  code: '35510'
  city: Cesson-Sevigne
  region: Bretagne
  country: France
  email: a@ackl.io
- ins: R. T. Turner
  name: Randy Turner
  org: Landis+Gyr
  street:
  - 30000 Mill Creek Ave
  - Suite 100
  code: '30022'
  city: Alpharetta
  region: GA
  country: US
  phone: "++16782581292"
  email: randy.turner@landisgyr.com
  uri: http://www.landisgyr.com/
- ins: A. M.  Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 2bis rue de la châtaigneraie
  code: '35510'
  city: Cesson-Sévigné
  region: Bretagne
  country: France
  email: ana@ackl.io
- ins: A.  S. Somaraju
  name: Abhinav Somaraju
  org: Tridonic GmbH & Co KG
  street: Farbergasse 15
  code: '6850'
  city: Dornbirn
  region: Vorarlberg
  country: Austria
  phone: "+43664808926169"
  email: abhinav.somaraju@tridonic.com
normative:
  RFC7950:
  RFC7951:
  RFC2119:
  RFC7049:
informative:
  RFC5226:
  RFC6241:
  RFC7223:
  RFC7224:
  RFC7277:
  RFC7317:
  RFC8040:
  I-D.ietf-core-comi: comi

--- abstract

YANG Schema Item iDentifiers (SID) are globally unique 64-bit numeric identifiers used to identify all items used in YANG.  This document defines the semantics, the registration, and assignment processes of SIDs.  To enable the implementation of these processes, this document also defines a file format used to persist and publish assigned SIDs.

--- middle

# Introduction

Some of the items defined in YANG {{RFC7950}} require the use of a unique identifier.  In both NETCONF {{RFC6241}} and RESTCONF, {{RFC8040}} these identifiers are implemented using names.  To allow the implementation of data models defined in YANG in constrained devices and constrained networks, a more compact method to identify YANG items is required. This compact identifier, called SID, is encoded using a 64-bit unsigned integer. The following items are identified using SIDs:

* identities

* data nodes

* RPCs and associated input(s) and output(s)

* actions and associated input(s) and output(s)

* notifications and associated information

* YANG modules, submodules and features

To minimize their size, SIDs are often represented as a difference between the current SID and a reference SID. Such difference is called "delta" (shorthand for "delta-encoded SID").  Conversion from SIDs to deltas (and back to SIDs) is a stateless process. Each protocol implementing deltas must unambiguously define the reference SID for each YANG item.

SIDs are globally unique numbers, a registration system is used in order to guarantee their uniqueness. SIDs are registered in blocks called "SID ranges". {{sid-range-registry}} provide more details about the registration process of SID range(s).

Assignment of SIDs to YANG items can be automated, the recommended process to assign SIDs is as follows:

* A tool extracts the different items defined for a specific YANG module.

* The list of items is ordered in alphabetical order by type and label. Valid types and label formats are described within the 'ietf-sid-file' YANG module defined in {{sid-file-format}}.

* SIDs are assigned sequentially from the entry point up to the size of the registered SID range. This approach is recommended to minimize the serialization overhead, especially when delta encoding is implemented.

* If the number of items exceeds the SID range(s) allocated to a YANG module, an extra range is added for subsequent assignments.

SIDs are assigned permanently, items introduced by a new revision of a YANG module are added to the list of SIDs already assigned.  This process can also be automated using the same method described above except that the assignment restart from the highest SID already assigned plus one.

To avoid duplicate assignment of SIDs, the registration of the SIDs assigned to YANG module(s) is recommended.  {{module-registry}} provide more details about the registration process of YANG modules and associated SIDs. To enable the implementation of this registry, {{sid-file-format}} defines a standard file format used to store and publish SIDs.

# Terminology and Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in {{RFC2119}}.

The following terms are defined in {{RFC7950}}:

* action

* feature

* module

* notification

* RPC

* schema node

* schema tree

* submodule

This specification also makes use of the following terminology:

* delta : Difference between the current SID and a reference SID. A reference SID is defined for each context for which deltas are used.

* item:  A schema node, an identity, a module, a submodule or a feature defined using the YANG modeling language.

* path: A path is a string that identifies a schema node within the schema tree. A path consists of the list of schema node identifier(s) separated by slashes ("/"). Schema node identifier(s) are always listed from the top-level schema node up to the targeted schema node. (e.g. "/system-state/clock/current-datetime")

* YANG Schema Item iDentifier (SID): Unsigned integer used to identify different YANG items.

# ".sid" file lifecycle  {#sid-lifecycle}

YANG is a language designed to model data sent between clients and servers using one of the compatible protocol (e.g. NETCONF {{RFC6241}}, RESCONF {{RFC8040}} and CoMI {{-comi}}). A YANG module defines hierarchies of data, including configuration, state data, RPCs, actions and notifications.

YANG modules are not necessary created in the context of constrained applications. YANG modules can be implemented using NETCONF {{RFC6241}} or RESTCONF {{RFC8040}} without the need to assign SIDs.

As needed, authors of YANG modules can assign SIDs to their YANG modules. This process starts by the registration of a SID range. Once a SID range is registered, the owner of this range assigns sub-ranges to each YANG module in order to generate the associated “.sid” files. Generation of “.sid” files SHOULD be performed using an automated tool.

Registration of the .sid file associated to a YANG module is optional but recommended to promote interoperability between devices and to avoid duplicate allocation of SIDs to a single YANG module.

The following activity diagram summarizes the creation of a YANG module and its associated .sid file.

~~~~
       +---------------+
  O    | Creation of a |
 -|- ->| YANG module   |
 / \   +---------------+
               |
               V
        /-------------\
       / Standardized  \ yes
       \ YANG module ? /-------------+
        \-------------/              |
               | no                  |
               V                     V
        /-------------\      +---------------+
       / Constrained   \ yes | SID range     |
   +-->\ application ? /---->| registration  |
   |    \-------------/      +---------------+
   |           | no                  |
   |           V                     V
   |   +---------------+     +---------------+
   +---| YANG module   |     | SID sub-range |
       | update        |     | assignment    |
       +---------------+     +---------------+
                                     |
                                     V
                             +---------------+
                             | .sid file     |
                             | generation    |
                             +---------------+
                                     |
                                     V
                              /-------------\      +---------------+
                             /  Publicly     \ yes | YANG module   |
                             \  available ?  /---->| registration  |
                              \-------------/      +---------------+
                                     | no                  |
                                     +---------------------+
                                     |
                                   [DONE]
~~~~
{: align="left"}

Each time a YANG module or one of its imported module(s) or included sub-module(s) is updated, the ".sid" file MAY need to be updated. This update SHOULD also be performed using an automated tool.

If a new revision requires more SIDs than initially allocated, a new SID range MUST be added to the assignment ranges as defined in the ".sid" file header. These extra SIDs are used for subsequent assignements.

The following activity diagram summarizes the update of a YANG module and its associated .sid file.

~~~~
       +---------------+
  O    | Update of the |
 -|- ->| YANG module   |
 / \   | or include(s) |
       | or import(s)  |
       +---------------+
               |
               V
        /-------------\      +----------------+
       /  More SIDs    \ yes | Extra sub-range|
       \  required ?   /---->| assignment     |
        \-------------/      +----------------+
               | no                  |
               +---------------------+
               |
               V
       +---------------+
       | .sid file     |
       | update based  |
       | on previous   |
       | .sid file     |
       +---------------+
               |
               V
        /-------------\      +---------------+
       /  Publicly     \ yes | YANG module   |
       \  available ?  /---->| registration  |
        \-------------/      +---------------+
               | no                  |
               +---------------------+
               |
             [DONE]
~~~~
{: align="left"}

# ".sid" file format  {#sid-file-format}

".sid" files are used to persist and publish SIDs assigned to the different YANG items of a specific YANG module. The following YANG module defined the structure of this file, encoding is performed using the rules defined in {{RFC7951}}.

~~~~
<CODE BEGINS> file "ietf-sid-file@2015-12-16.yang"
module ietf-sid-file {
  namespace "urn:ietf:params:xml:ns:yang:ietf-sid-file";
  prefix sid;

  organization
    "IETF Core Working Group";

  contact
    "Alexander Pelov
     <mailto:a@ackl.io>

     Abhinav Somaraju
     <mailto:abhinav.somaraju@tridonic.com>

     Laurent Toutain
     <Laurent.Toutain@telecom-bretagne.eu>

     Randy Turner
     <mailto:Randy.Turner@landisgyr.com>

     Michel Veillette
     <mailto:michel.veillette@trilliantinc.com>";

  description
    "This module define the structure of the .sid files.
     .sid files contains the identifiers (SIDs) assigned
     to the different items defined in a YANG module.
     SIDs are used to encode a data model defined in YANG
     using CBOR.";

  revision 2015-12-16 {
    description
      "Initial revision.";
    reference
      "RFC XXXX";
      // RFC Ed.: replace XXXX with RFC number assigned to
      // draft-ietf-core-yang-cbor and remove this note
  }

  typedef yang-identifier {
    type string {
      length "1..max";
      pattern '[a-zA-Z_][a-zA-Z0-9\-_.]*';
      pattern '.|..|[^xX].*|.[^mM].*|..[^lL].*';
    }
    description
      "A YANG identifier string as defined by the 'identifier'
       rule in Section 12 of RFC 6020.";
  }

  typedef revision-identifier {
    type string {
      pattern '\d{4}-\d{2}-\d{2}';
    }
    description
      "Represents a date in YYYY-MM-DD format.";
  }

  leaf module-name {
    type yang-identifier;
    description
      "Name of the module associated with this .sid file.";
  }

  leaf module-revision {
    type revision-identifier;
    description
      "Revision of the module associated with this .sid file.
       This leaf is not present if no revision statement is
       defined in the YANG module.";
  }

  list assigment-ranges {
    key "entry-point";
    description
      "Range(s) of SIDs available for assignment to the
       different items defined by the associated module.";

    leaf entry-point {
      type uint32;
      mandatory true;
      description
        "Lowest SID available for assignment.";
    }

    leaf size {
      type uint16;
      mandatory true;
      description
        "Number of SIDs available for assignment.";
    }
  }

  list items {
    key "type label";
    description
      "List of items defined by the associated YANG module.";

    leaf type {
      type string {
        pattern 'Module|Submodule|feature|' +
                'identity$|node$|notification$|rpc$|action$';
      }
      mandatory true;
      description
        "Item type assigned, this field can be set to:
          - 'Module'
          - 'Submodule'
          - 'feature'
          - 'identity'
          - 'node'
          - 'notification'
          - 'rpc'
          - 'action'";
    }

    leaf label {
      type string;
      mandatory true;
      description
        "Label associated to this item, can be set to:
          - a module name
          - a submodule name
          - a feature name
          - a base identity encoded as
            '/<base identity name>'
          - an identity encoded as
            '/<base identity name>/<identity name>'
          - a data node path";
    }

    leaf sid {
      type uint32;
      mandatory true;
      description "Identifier assigned to this YANG item.";
    }
  }
}
<CODE ENDS>
~~~~
{: align="left"}

# Security Considerations

The security considerations of {{RFC7049}} and {{RFC7950}} apply.

This document defines a new type of identifier used to encode data models defined in YANG {{RFC7950}}. As such, this identifier does not contribute to any new security issues in addition of those identified for the specific protocols or contexts for which it is used.

# IANA Considerations  {#IANA}

## "SID mega-range" registry  {#sid-range-registry}

The name of this registry is "SID mega-range". This registry is used to delegate the management of block of SIDs for third party's (e.g. SDO, registrar).

Each entry in this registry must include:

* The entry point (first entry) of the registered SID range.

* The size of the registered SID range.

* The contact information of the requesting organization including:

  * Organization name

  * Primary contact name, email address, and phone number

  * Secondary contact name, email address, and phone number

The initial entry in this registry reserved for IANA:

| Entry Point | Size    | Organization name     |
|-------------+---------+-----------------------|
| 0           | 1000000 | IANA                  |
{: align="left"}

The IANA policies for future additions to this registry are "Hierarchical Allocation, Expert Review" {{RFC5226}}. Prior to a first allocation, the requesting organization must demonstrate a functional registry infrastructure. On subsequent allocation request(s), the organization must demonstrate the exhaustion of the prior range. These conditions need to be asserted by the assigned expert(s).

### IANA SID Mega-Range Registry

The first million SIDs assigned to IANA and sub-divided as follow:

* The range of 0 to 999 is reserved for future extensions.  The IANA policy for this range is "IETF review" {{RFC5226}}.

* The range of 1000 to 59,999 is reserved for YANG modules defined in RFCs.  The IANA policy for future additions to this sub-registry is "RFC required" {{RFC5226}}. Allocation within this range requires publishing of the associated ".yang" and ".sid" files in the YANG module registry. 

* The range of 60,000 to 99,999 is reserved for experimental YANG modules.  Use of this range MUST NOT be used in operational deployments since these SIDs are not globally unique which limit their interoperability. The IANA policy for this range is "Experimental use" {{RFC5226}}.

* The range of 100,000 to 999,999 is reserved for standardized YANG modules.  The IANA policy for future additions to this sub-registry is "Specification Required" {{RFC5226}}. Allocation within this range requires publishing of the associated ".yang" and ".sid" files in the YANG module registry.

| Entry Point   | Size          | IANA policy                       |
|---------------+---------------+-----------------------------------|
| 0             | 1,000         | IETF review                       |
| 1,000         | 59,000        | RFC required                      |
| 60,000        | 40,000        | Experimental use                  |
| 100,000       | 1,000,000,000 | Specification Required            |
{: align="left"}

The size of SID range assigned to a YANG module should be between 50% and 60% of the current number of YANG items. This headroom allows assignment within the same range of new YANG items introduced by the subsequent versions. A larger SID range size may be requested by the authors if this recommendation is considered insufficient. It is important to note that an extra range can be allocated an existing YANG module if the initial ranges(s) are eventually exhausted.

###  IANA "RFC SID range assignment" sub-registries

The name of this sub-registry is "RFC SID range assignment". This sub-registry corresponds to the SID entry point 1000, size 59000. Each entry in this sub-registry must include the SID range entry point, the SID range size, the YANG module name, the RFC number.
  
Initial entries in this registry are as follows:

| Entry Point | Size | Module name     | Reference              |
|-------------+------+-----------------+------------------------|
| 1000        | 100  |                 | Reserved for {{-comi}} |
| 1100        | 400  | iana-if-type    | {{RFC7224}}            |
| 1500        | 100  | ietf-interfaces | {{RFC7223}}            |
| 1600        | 100  | ietf-ip         | {{RFC7277}}            |
| 1700        | 100  | ietf-system     | {{RFC7317}}            |
{: align="left"}

##  "YANG module assignment" registry {#module-registry}

The name of this registry is "YANG module assignment". This registry is used to track which YANG modules have been assigned and the specific YANG items assignment. Each entry in this sub-registry must include:

* The YANG module name

* The associated ".yang" file(s)

* The associated ".sid" file


The validity of the ".yang" and ".sid" files added to this registry MUST be verified.

* The syntax of the registered ".yang" and ".sid" files must be valid.

* Each YANG item defined by the registered ".yang" file must have a SID assigned in the ".sid" file.

* Each SID is assigned to a single YANG item, duplicate assignment is not allowed.

* The SID range(s) defined in the ".sid" file must be unique, must not conflict with any other ranges defined in already registered ".sid" files. 

* The ownership of the SID range(s) should be verify.

The IANA policy for future additions to this registry is "First Come First Served" as described in {{RFC5226}}.

# Acknowledgments

The authors would like to thank Carsten Bormann for his help during the development of this document and his useful comments during the review process.

--- back

# ".sid" file example  {#sid-file-example}

The following .sid file (ietf-system@2014-08-06.sid) have been generated using the following yang modules:

* ietf-system@2014-08-06.yang

* ietf-yang-types@2013-07-15.yang

* ietf-inet-types@2013-07-15.yang

* ietf-netconf-acm@2012-02-22.yang

* iana-crypt-hash@2014-04-04.yang

~~~~
{
 "assignment-ranges": [
  {
   "entry-point": 1700,
   "size": 100
  }
 ],
 "module-name": "ietf-system",
 "module-revision": "2014-08-06",
 "items": [
  {
   "type": "Module",
   "label": "ietf-system",
   "sid": 1700
  },
  {
   "type": "feature",
   "label": "authentication",
   "sid": 1701
  },
  {
   "type": "feature",
   "label": "dns-udp-tcp-port",
   "sid": 1702
  },
  {
   "type": "feature",
   "label": "local-users",
   "sid": 1703
  },
  {
   "type": "feature",
   "label": "ntp",
   "sid": 1704
  },
  {
   "type": "feature",
   "label": "ntp-udp-port",
   "sid": 1705
  },
  {
   "type": "feature",
   "label": "radius",
   "sid": 1706
  },
  {
   "type": "feature",
   "label": "radius-authentication",
   "sid": 1707
  },
  {
   "type": "feature",
   "label": "timezone-name",
   "sid": 1708
  },
  {
   "type": "identity",
   "label": "/authentication-method",
   "sid": 1709
  },
  {
   "type": "identity",
   "label": "/authentication-method/local-users",
   "sid": 1710
  },
  {
   "type": "identity",
   "label": "/authentication-method/radius",
   "sid": 1711
  },
  {
   "type": "identity",
   "label": "/radius-authentication-type",
   "sid": 1712
  },
  {
   "type": "identity",
   "label": "/radius-authentication-type/radius-chap",
   "sid": 1713
  },
  {
   "type": "identity",
   "label": "/radius-authentication-type/radius-pap",
   "sid": 1714
  },
  {
   "type": "node",
   "label": "/system",
   "sid": 1715
  },
  {
   "type": "node",
   "label": "/system-state",
   "sid": 1716
  },
  {
   "type": "node",
   "label": "/system-state/clock",
   "sid": 1717
  },
  {
   "type": "node",
   "label": "/system-state/clock/boot-datetime",
   "sid": 1718
  },
  {
   "type": "node",
   "label": "/system-state/clock/current-datetime",
   "sid": 1719
  },
  {
   "type": "node",
   "label": "/system-state/platform",
   "sid": 1720
  },
  {
   "type": "node",
   "label": "/system-state/platform/machine",
   "sid": 1721
  },
  {
   "type": "node",
   "label": "/system-state/platform/os-name",
   "sid": 1722
  },
  {
   "type": "node",
   "label": "/system-state/platform/os-release",
   "sid": 1723
  },
  {
   "type": "node",
   "label": "/system-state/platform/os-version",
   "sid": 1724
  },
  {
   "type": "node",
   "label": "/system/authentication",
   "sid": 1725
  },
  {
   "type": "node",
   "label": "/system/authentication/user",
   "sid": 1726
  },
  {
   "type": "node",
   "label": "/system/authentication/user-authentication-order",
   "sid": 1727
  },
  {
   "type": "node",
   "label": "/system/authentication/user/authorized-key",
   "sid": 1728
  },
  {
   "type": "node",
   "label": "/system/authentication/user/authorized-key/algorithm",
   "sid": 1729
  },
  {
   "type": "node",
   "label": "/system/authentication/user/authorized-key/key-data",
   "sid": 1730
  },
  {
   "type": "node",
   "label": "/system/authentication/user/authorized-key/name",
   "sid": 1731
  },
  {
   "type": "node",
   "label": "/system/authentication/user/name",
   "sid": 1732
  },
  {
   "type": "node",
   "label": "/system/authentication/user/password",
   "sid": 1733
  },
  {
   "type": "node",
   "label": "/system/clock",
   "sid": 1734
  },
  {
   "type": "node",
   "label": "/system/clock/timezone-name",
   "sid": 1735
  },
  {
   "type": "node",
   "label": "/system/clock/timezone-utc-offset",
   "sid": 1736
  },
  {
   "type": "node",
   "label": "/system/contact",
   "sid": 1737
  },
  {
   "type": "node",
   "label": "/system/dns-resolver",
   "sid": 1738
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/options",
   "sid": 1739
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/options/attempts",
   "sid": 1740
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/options/timeout",
   "sid": 1741
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/search",
   "sid": 1742
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/server",
   "sid": 1743
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/server/name",
   "sid": 1744
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/server/udp-and-tcp",
   "sid": 1745
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/server/udp-and-tcp/address",
   "sid": 1746
  },
  {
   "type": "node",
   "label": "/system/dns-resolver/server/udp-and-tcp/port",
   "sid": 1747
  },
  {
   "type": "node",
   "label": "/system/hostname",
   "sid": 1748
  },
  {
   "type": "node",
   "label": "/system/location",
   "sid": 1749
  },
  {
   "type": "node",
   "label": "/system/ntp",
   "sid": 1750
  },
  {
   "type": "node",
   "label": "/system/ntp/enabled",
   "sid": 1751
  },
  {
   "type": "node",
   "label": "/system/ntp/server",
   "sid": 1752
  },
  {
   "type": "node",
   "label": "/system/ntp/server/association-type",
   "sid": 1753
  },
  {
   "type": "node",
   "label": "/system/ntp/server/iburst",
   "sid": 1754
  },
  {
   "type": "node",
   "label": "/system/ntp/server/name",
   "sid": 1755
  },
  {
   "type": "node",
   "label": "/system/ntp/server/prefer",
   "sid": 1756
  },
  {
   "type": "node",
   "label": "/system/ntp/server/udp",
   "sid": 1757
  },
  {
   "type": "node",
   "label": "/system/ntp/server/udp/address",
   "sid": 1758
  },
  {
   "type": "node",
   "label": "/system/ntp/server/udp/port",
   "sid": 1759
  },
  {
   "type": "node",
   "label": "/system/radius",
   "sid": 1760
  },
  {
   "type": "node",
   "label": "/system/radius/options",
   "sid": 1761
  },
  {
   "type": "node",
   "label": "/system/radius/options/attempts",
   "sid": 1762
  },
  {
   "type": "node",
   "label": "/system/radius/options/timeout",
   "sid": 1763
  },
  {
   "type": "node",
   "label": "/system/radius/server",
   "sid": 1764
  },
  {
   "type": "node",
   "label": "/system/radius/server/authentication-type",
   "sid": 1765
  },
  {
   "type": "node",
   "label": "/system/radius/server/name",
   "sid": 1766
  },
  {
   "type": "node",
   "label": "/system/radius/server/udp",
   "sid": 1767
  },
  {
   "type": "node",
   "label": "/system/radius/server/udp/address",
   "sid": 1768
  },
  {
   "type": "node",
   "label": "/system/radius/server/udp/authentication-port",
   "sid": 1769
  },
  {
   "type": "node",
   "label": "/system/radius/server/udp/shared-secret",
   "sid": 1770
  },
  {
   "type": "rpc",
   "label": "/set-current-datetime",
   "sid": 1771
  },
  {
   "type": "rpc",
   "label": "/set-current-datetime/input/current-datetime",
   "sid": 1772
  },
  {
   "type": "rpc",
   "label": "/system-restart",
   "sid": 1773
  },
  {
   "type": "rpc",
   "label": "/system-shutdown",
   "sid": 1774
  }
 ]
}
~~~~

--- back
