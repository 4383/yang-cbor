---
stand_alone: true
ipr: trust200902
docname: draft-veillette-core-yang-library-02
title: Constrained YANG Module Library
area: Applications and Real-Time Area (art)
wg: Internet Engineering Task Force
kw: YANG
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
normative:
  RFC2119:
  RFC7950:
  I-D.ietf-core-yang-cbor: core-yang-cbor
  I-D.ietf-core-comi: comi
  I-D.ietf-netmod-yang-tree-diagrams: tree-diagrams
informative:
  RFC7895:
  I-D.dsdt-nmda-guidelines: nmda-guidelines
  I-D.ietf-netconf-rfc7895bis: netconf-rfc7895bis
  I-D.ietf-netmod-revised-datastores: revised-datastores
  I-D.ietf-netconf-nmda-restconf: nmda-restconf

--- abstract

This document describes a YANG library that provides information about all the YANG modules used by a constrained network management server (e.g., a CoAP Management Interface (CoMI) server). Simple caching mechanisms are provided to allow clients to minimize retrieval of this information.

--- middle

# Introduction

WARNING:
The CoMI protocol {{-comi}} and this contribution need to be reviewed to verify their compatibility with the "Network Management Datastore Architecture" (NMDA) introduced netmod. See {{-nmda-guidelines}}, {{-netconf-rfc7895bis}}, {{-revised-datastores}} and {{-nmda-restconf}} for more details.

The YANG library specified in this document is available to clients of a given server to discover the YANG modules supported by this constrained network management server. A CoMI server provides a link to this library in the /mod.uri resource. The following YANG module information is provided to client applications to fully utilize the YANG data modeling language:

* module list: The list of YANG modules implemented by a server, each module is identified by its assigned YANG Schema Item iDentifier (SID) and revision.

* submodule list: The list of YANG submodules included by each module, each submodule is identified by its assigned SID and revision. 
   
* feature list: The list of features supported by the server, each feature is identified by its assigned SID.

* deviation list: The list of YANG modules used for deviation statements associated with each YANG module, each module is identified by its assigned SID and revision.


## Major differences between ietf-constrained-yang-library and ietf-yang-library

YANG module 'ietf-constrained-yang-library' targets the same functionality and shares the same approach as YANG module ietf-yang-library. The following changes with respect to ietf-yang-library are specified to make ietf-constrained-yang-library compatible with SID {{-core-yang-cbor}} used by CoMI {{-comi}} and to improve its applicability to constrained devices and networks. 

* YANG module 'ietf-constrained-yang-library' extends the caching mechanism supported by 'ietf-yang-library' to multiple servers of the same type. This is accomplished by replacing the 'module-set-id' by a hash of the library content. 

* Modules, sub-modules, deviations and features are identified using a numerical value (SID) instead of a string (yang-identifier).

* The "namespace" leaf, not required for SIDs, but mandatory in 'ietf-yang-library' is not included in 'ietf-constrained-yang-library'.

* Schemas can be located using the already available module or sub-module identifier (SID) and revision. For this reason, support of module and sub-module schema URIs have been removed.

o To minimize their size, each revision date is encoded in binary.

# Terminology and Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in {{RFC2119}}.

The following terms are defined in {{RFC7950}}:

* module

* submodule

* feature

* deviation

The following terms are defined in {{-core-yang-cbor}}:

* YANG Schema Item iDentifier (SID)

The following terms are defined in {{-comi}}:

* client

* server
   
The following terms are used within this document:

* library: a collection of YANG modules used by a server.

# Overview

The "ietf-constrained-yang-library" module provides information about the YANG library used by a given server.  This module is defined using YANG version 1 as defined by {{RFC7950}}, but it supports the description of YANG modules written in any revision of YANG.

## Tree diagram

The tree diagram of YANG module ietf-constrained-yang-library is provided below. This graphical representation of a YANG module is defined in {{-tree-diagrams}}.

~~~~
module: ietf-constrained-yang-library
   +--ro modules-state
      +--ro hash      binary
      +--ro module* [sid revision]
         +--ro sid                 comi:sid
         +--ro revision            revision
         +--ro feature*            comi:sid
         +--ro deviation* [sid revision]
         |  +--ro sid         comi:sid
         |  +--ro revision    revision
         +--ro conformance-type    enumeration
         +--ro submodule* [sid revision]
            +--ro sid         comi:sid
            +--ro revision    revision

notifications:
   +---n yang-library-change
      +--ro hash    -> /modules-state/hash
~~~~
{: align="left"}

## Description

### modules-state

This mandatory container specifies the module set identifier and the list of modules supported by the server.

###  modules-state/hash

This mandatory leaf contains the hash of the library content. The value of this leaf MUST change whenever the set of modules and submodules in the library changes. This leaf allows a client to fetch the module list once, cache it, and only re-fetch it if the value of this leaf has been changed.

If the value of this leaf changes, the server also generates a 'yang-library-change' notification.

###  modules-state/module

This mandatory list contains one entry for each YANG module supported by the server.  There MUST be an entry in this list for each revision of each YANG module that is used by the server. It is possible for multiple revisions of the same module to be imported, in addition to an entry for the revision that is implemented by the server.

# YANG Module "ietf-constrained-yang-library"

RFC Ed.: update the date below with the date of RFC publication
and remove this note.
   
~~~~
<CODE BEGINS> file "ietf-constrained-yang-library@2018-01-20.yang"
module ietf-constrained-yang-library {
  namespace
    "urn:ietf:params:xml:ns:yang:ietf-constrained-yang-library";
  prefix "lib";

  import ietf-comi {
    prefix comi;
  }
  
  organization
    "IETF CORE (Constrained RESTful Environments) Working Group";

  contact
    "WG Web:   <http://datatracker.ietf.org/wg/core/>
    
     WG List:  <mailto:core@ietf.org>

     WG Chair: Carsten Bormann 
               <mailto:cabo@tzi.org>

     WG Chair: Jaime Jimenez 
               <mailto:jaime.jimenez@ericsson.com>

     Editor:   Michel Veillette
               <mailto:michel.veillette@trilliantinc.com>";

  description
    "This module contains the list of YANG modules and submodules
    implemented by a server.

     Copyright (c) 2016 IETF Trust and the persons identified as
     authors of the code.  All rights reserved.

     Redistribution and use in source and binary forms, with or
     without modification, is permitted pursuant to, and subject
     to the license terms contained in, the Simplified BSD License
     set forth in Section 4.c of the IETF Trust's Legal Provisions
     Relating to IETF Documents
     (http://trustee.ietf.org/license-info).

     This version of this YANG module is part of RFC XXXX; see
     the RFC itself for full legal notices.";

  // RFC Ed.: replace XXXX with actual RFC number and remove
  // this note.

  // RFC Ed.: update the date below with the date of the RFC
  // publication and remove this note.

  revision 2018-01-20 {
    description
      "Initial revision.";
    reference
      "RFC XXXX: Constrained YANG Module Library.";
  }

  /*
   * Typedefs
   */

  typedef revision {
    type binary {
      length "4";
    }
    description
      "Revision date encoded as a binary string as follow:
      - First byte = Century
      - Second byte = Year (0 to 99)
      - Third byte = Month (1 = January to 12 = December)
      - Forth byte = Day (1 to 31)";
  }

  /*
   * Groupings
   */

  grouping identification-info {
    description
      "YANG modules and submodules identification information.";

    leaf sid {
      type comi:sid;
      mandatory true;
      description
        "SID assigned to this module or submodule.";
    }
    
    leaf revision {
      type revision;
      description
        "Revision date assigned to this module or submodule.
        A zero-length binary string is used if no revision
        statement is present in the YANG module or submodule.";
    }
  }
  
  identity module-set {
      description
        "Base identity from which shared module-set identifiers
        are derived.";
    }
  
  /*
   * Operational state data nodes
   */

  container modules-state {
    config false;
    description
      "Contains information about the different data models
      implemented by the server.";
    
    leaf hash {
      type binary {
        length "8..32";
      }
      mandatory true;
      description
        "A server-generated hash of the contents of the library.
        The server MUST change the value of this leaf each time
        the content of the library has changed. The hash function
        and size are not specified, but shall be collision
        resistant.";
    }

    list module {
      key "sid revision";
      description
        "Each entry represents one revision of one module
         currently supported by the server.";

      uses identification-info;
      
      leaf-list feature {
        type comi:sid;
        description
          "List of YANG features from this module that are
          supported by the server, regardless whether
          they are defined in the module or in any
          included submodules.";
      }
      
      list deviation {
        key "sid revision";
        description
          "List of YANG deviation modules used by this server
          to modify the conformance of the module associated
          with this entry.  Note that the same module can be
          used for deviations for multiple modules, so the same
          entry MAY appear within multiple 'module' entries.

          Deviation modules MUST also be present in the 'module'
          list, with the same sid and revision values and the
          'conformance-type' set to 'implement'.";
          
        uses identification-info;
      }
      
      leaf conformance-type {
        type enumeration {
          enum implement {
            value 0;
            description
              "Indicates that the server implements one or more
              protocol-accessible objects defined in the YANG
              module identified in this entry.  This includes
              deviation statements defined in the module.

              For YANG version 1.1 modules, there is at most one
              module entry with conformance type 'implement' for
              a particular module, since YANG 1.1 requires that
              at most one revision of a module is implemented.

              For YANG version 1 modules, there SHOULD NOT be more
              than one module entry for a particular module.";
          }
          enum import {
            value 1;
            description
              "Indicates that the server imports reusable
              definitions from the specified revision of the
              module, but does not implement any protocol
              accessible objects from this revision.

              Multiple module entries for the same module MAY
              exist. This can occur if multiple modules import
              the same module, but specify different revision-dates
              in the import statements.";
          }
        }
        mandatory true;
        description
          "Indicates the type of conformance the server is claiming
          for the YANG module identified by this entry.";
      }
      
      list submodule {
        key "sid revision";
        description
          "Each entry represents one submodule within the
           parent module.";
        uses identification-info;
      }
    }
  }

  /*
   * Notifications
   */

  notification yang-library-change {
    description
      "Generated when the set of modules and submodules supported
      by the server has changed.";
      
    leaf hash {
      type leafref {
        path "/lib:modules-state/lib:hash";
      }
      mandatory true;
      description
        "New hash value.";
    }
  }
}
<CODE ENDS>
~~~~
{: align="left"}

# IANA Considerations

## YANG Module Registry

This document registers one YANG module in the YANG Module Names registry {{RFC7950}}.

name:         ietf-constrained-yang-library

namespace:    urn:ietf:params:xml:ns:yang:ietf-constrained-yang-library

prefix:       lib

reference:    RFC XXXX

// RFC Ed.: replace XXXX with RFC number and remove this note

# Security Considerations

This YANG module is designed to be accessed via the CoMI protocol {{-comi}}.  Some of the readable data nodes in this YANG module may be considered sensitive or vulnerable in some network environments.  It is thus important to control read access to these data nodes.

Specifically, the 'module' list may help an attacker to identify the server capabilities and server implementations with known bugs. Server vulnerabilities may be specific to particular modules, module revisions, module features, or even module deviations.  This information is included in each module entry.  For example, if a particular operation on a particular data node is known to cause a server to crash or significantly degrade device performance, then the module list information will help an attacker identify server implementations with such a defect, in order to launch a denial of service attack on the device.

# Acknowledgments

The YANG module defined by this memo have been derived from an already existing YANG module, ietf-yang-library {{RFC7895}}, we will like to thanks to the authors of this YANG module. A special thank also to Andy Bierman for his initial recommendations for the creation of this YANG module.

--- back

