---
stand_alone: true
ipr: trust200902
docname: draft-petrov-t2trg-youpi-latest
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'
title: YANG Object Universal Parsing Interface
abbrev: YOUPI
rg: t2trg Research Group
author:
- role: editor
  ins: I. Petrov
  name: Ivaylo Petrov
  org: Acklio
  street: 1137A avenue des Champs Blancs
  code: '35510'
  city: Cesson-Sevigne
  region: Bretagne
  country: France
  email: ivaylo@ackl.io
normative:
  RFC2119:
  RFC8174:
  RFC7950:
informative:

--- abstract

YANG Object Universal Parsing Interface (YOUPI) specification describes generic
way to encode and decode binary data based on a YANG model for use of
constrainted devices. YOUPI is a generic mechanism designed for great
flexibility, so that it can be adapted for any of the constainted devices.

--- middle

# Introduction {#introduction}

A huge number of very constraint IoT devices are expected to be coming to the
market. They are very constraint in terms of the MTU (sometimes as small as 10b
per message). As they are expected to be running for many years without the
need for external energy, energy efficiency which is directly linked to the
size of the payloads that need to be sent, is also very important. For those
devices JSON and even CBOR formats might be too wasteful in terms of payload
size. The reality of the ecosystem is that currently a great number of
applications use proprietary binary formats for exchanging information. A
significant problem exists if those systems are to be interacting in a purely
M2M fashion. While there are a number of possibilities to resolve those issues,
due to the constraints it is mandatory to have a way to extract and encode
information from/to the binary payload and be able to annotate it with semantic
metadata.

While binary formats can be rather complicated to parse and sometimes even
context dependent (some entity needs to keep context in order to parse a
message), for most cases a simple description format could be sufficient.

A good solution should not be bounded to the output format. It should be a data
modeling language like YANG {{RFC7950}} that simply describes the structure of
the obtained data and that allows different serialization formats afterwards.

## Terminology {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# YOUPI

YOUPI provides a number of yang extentions as defined in {{yang-extensions}}.
Thanks to that additional information in the YANG definitions, it is possible
to decode binary data and then transform it to a different easier to parse
format like JSON, XML or CBOR. Additionally it defines extensions that allow meta
information to be added so that JSON-LD is generated. This draft is not
describing how the data is formatted as JSON or other format. For information
how this could be done, please refer to RESTCONF, NETCONF or CORECONF.

The opposite process is also possible - generating binary packets from parsed
data that comes from JSON or other format.

## YANG extentions {#yang-extensions}

The definitions of the YANG extensions.

~~~~ yang
<CODE BEGINS> file "petrov-youpi-file@2019-07-22.yang"
module youpi {
    namespace "http://ackl.io/youpi";

    prefix "youpi";

    organization
	"Acklio";

    contact
	"Ivaylo Petrov
	<mailto:ivaylo@ackl.io>";

    description
	"This module defines the extentions used by youpi.";

    revision 2019-07-22 {
	description "Initial revision.";
    }

    /**
     *
     * Extension for Binary data to CBOR mapping.
     *
     **/
    extension position {
        argument object;
    }

    extension fieldIndex {
        argument object;
    }

    extension condition {
        argument object;
    }

    extension multiplier {
        argument object;
    }

    extension offset {
        argument object;
    }

    extension units-subject {
        argument object;
    }

    extension js {
        argument object;
    }
}
~~~~

## Position

Information about which bits need to be used in order to find the value of a field.

### Bit positions

If the position is not present or is empty, the value contains 0 bits and has a
default value of 0 (or equivalent for the given type). Could be useful if a
field needs to be the result of arithmetic operations from different fields.

It is possible to have a single bit read by giving only its value in the position extension.

If continuous bits need to be used to obtain the value of a given field, this
can be achieved using the `..` syntax. For example `0..3` means bits 0, 1, 2
and 3.

If non-continuous bits need to be used, one can use the concatenation of bit ranges using the `|` operator. For example `0..1 | 3`.

### Cursor

Starts at 0 and changes with each read to the last bit index that was read. Used in {{relative-cursor}} to determine where the read will start from. {{absolute-cursor}} is not affected by it, but changes its value.

### Absolute position {#absolute-cursor}

The default one if no keyword is used. Alternatively `absolute` keyword can be
provided to explicitly request such position.

Example:

~~~~ yang
leaf temp {
    type uint8;
    default -19;
    description "The temperature";
    youpi:position "0..6";
}
~~~~

### Relative position {#relative-cursor}

Example:

~~~~ yang
leaf temp {
    type uint8;
    default -19;
    description "The temperature";
    youpi:position "relative 1..7";
}
~~~~

This means that the value starts 1 bit after the current cursor and will read up to 7 bits after the current cursor position, including that 7th bit.

## FieldIndex

Can be used to change the order in which fields are processed. By default the order in which fields appear in the document is the order in which they are processed.

## Multiplier {#multiplier}

A value or another field by which a given field needs to be multiplied before
the final value is obtained. The operations are executed in the order of
appearance (this includes "offset" extension defined in {{offset}}).

## Offset {#offset}

A value or another field to which a given field needs to be added before
the final value is obtained. The operations are executed in the order of
appearance (this includes "offset" extension defined in {{offset}}).

## Units-subject

Meta information used to compute JSON-LD.

## Data definitions

### Supported built-in type

* binary
* enumeration
* int8
* int16
* int32
* int64
* string
* uint8
* uint16
* uint32
* uint64

### Leafs

Simple fields like integers and strings are represented by leafs in YOUPI.

### Type min/max values

`range` attribute can be used for giving a `min`/`max` acceptable value for a
type. If the value is outside of the defined range, it is silently excluded
from the final result.

Example:

~~~~
  typedef temp {
      type int8 {
          range "-20 .. 107";
      }
  }
~~~~

### Type fraction digits

It is possible to specify how many fraction digits are expected for a value to
have.

Example:

~~~~
  leaf temp {
      type decimal64 {
          fraction-digits 2;
      }
  }
~~~~

### Containers

Complex fields like objects are represented by containers in YOUPI.

### Condition

#### Choice

Inside a choice statement, the condition extension gives information based on what value the choice will be decided.

For example considering that there is a value "mode" with the value of btn inside the model

~~~~ yang
leaf mode {
    ...
}
choice data {
    case _btn {
        container button-data {
            ...
        }
    }
    case _temp {
        container temperature-data {
            ...
        }
    }
    youpi:condition "../mode";
}
~~~~

Then the button-data container will be used to parse the data.

#### When

With when statement it is possible to link the presence of some piece of data
to a value of another field. For example it is possible to have button-data or
temperature-data depending of the value of the mode field.

~~~~ yang
container button-data {
    when "../mode[.=1]"
    ...
}
container temperature-data {
    when "../mode[.=2]"
    ...
}
~~~~

### Lists

List statements are supported and they generate an array of a given composite
type.

#### With explicit length {#list-explicit-len}

A list of minimum and maximum temperatures can be defined as:

~~~~ yang
leaf temperature-len {
    type int32;
}

list temperatures {
    youpi:length "../temperatures-len";
    leaf min-value {
        type int32;
    }
    leaf max-value {
        type int32;
    }
}
~~~~

#### Until the end of input

The list as defined in {{list-explicit-len}} can omit the length extension
statement if all the remaining bytes in the payload are part of the list.

#### Until a specific value

The list as defined in {{list-explicit-len}} can also omit the length if it has
a defined key and if it only has one leaf or container in the list apart from
the key and it is a subject to when statement that defines a stop value for the
key.

~~~~ yang
list temperatures {
    key option-id;
    leaf option-id {
        type int32;
    }
    container value {
        when "../option-id[.!=0xffffffff]";
        ...
    }
}
~~~~

### Enumerations as mappings

Enumerations can be used inside a typedef in order to restrict a field only to
a set of acceptable values or in order to accomplish mapping between some
values and other values (for example 0 stands for "temperature", 1 stands for
"humidity", etc).

Example:

~~~~
    typedef mode-type {
        type enumeration {
            enum temp {
                value 0;
            }
            enum humidity {
                value 1;
            }
            enum light {
                value 2;
            }
            ...
        }
        ...
    }
~~~~

### Groupings

Groupings can be used for better reuse of definitions. They don't affect the
generated output.

### Typedefs

Typedefs can be used to provide extra information about the type of a field,
including semantic information about it.


# Security Considerations

The YANG file should be valid.

Segmentation faults might result from invalid data being provided with a given
YANG model.

Resource exhaustion can be looked for.

# IANA Considerations

This document registers a YANG model.

# Acknowledgements
{:numbered="false"}

# Contributors
{:numbered="false"}

--- back

# Complete examples
