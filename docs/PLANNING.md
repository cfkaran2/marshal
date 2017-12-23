# Goals

As stated in [README.md](../README.md), marshal's goal is to be **the**
[marshalling](https://en.wikipedia.org/wiki/Marshalling_(computer_science))
library for [rust](https://www.rust-lang.org).  To do that marshal will need to:

    * Handle arbitrary object graphs.  This includes object graphs that have
      cycles in them.
    * Handle objects that cannot be serialized. This includes things such as 
      open file pointers (which may no longer be valid after the object graph 
      has be unmarshalled).
    * Handle malformed input.
    * Abstract that container format from the objects being marshalled.  The 
      allows storing the object graph in any format that can handle an object 
      graph.

Explicit non-goals of marshal include:
    * The ability to handle different versions of a format (it's not 
      [Google Protocol Buffers](https://developers.google.com/protocol-buffers/)).
    * The ability to serialize to multiple related documents (it's not a 
      database, a file system, or other multi-document system, although the 
      backends could serialize to such systems).

# Plans

## Data Formats

One method of accomplishing the goals above is to follow in the footsteps of
[serde](https://serde.rs/), and choose a single data model that all backends are
required to accept, and to have the front end promise that it will represent the
object graph using that data model, as well as construct object graphs from that
data model. Since it is familiar to many people, I choose to use the data model
from the [YAML 1.2 specification](http://www.yaml.org/spec/1.2/spec.html); in
particular, see the [representation
graph](http://www.yaml.org/spec/1.2/spec.html#id2763754).  I plan to create
nodes as shown in that figure, with the front end outputting a graph of such
nodes, and  accepting those nodes to reconstitute the object graph.  Backends
(including the default YAML backend) must be able to consume that object graph
for marshalling, and produce such an object graph for the front end to
unmarshal.

## Traits

Unlike [serde](https://serde.rs/), marshal requires all backends have the
ability to both marshal and unmarshal object graphs; thus, there will only be a
single trait to implement, and it will have two functions, `marshal()` and
`unmarshal()`.  All objects that wish to participate in the marshal system will
need to implement these functions.  To simplify things, we can follow in
[serde's](https://serde.rs/) footsteps and implement macros for `#derive` to
process.  This will allow user code to use marshal without thinking too hard
about how it works under the hood.

## Known issues

There are a number of issues that I already know about, but which I don't have
good solutions for (yet).  If anyone has any ideas on how to handle these
issues, I'd appreciate it if you would let me know.

### Unmarshaling requires knowledge of types

Although it is relatively easy to marshal an object graph, there is a serious
issue in unmarshalling it; namely, you need to know the type of the object you
are unmarshalling in to in order to call the correct unmarshalling code.  There
are a several ways around this:

#### Do what [serde](https://serde.rs/) does

[serde](https://serde.rs/) requires that users specify the type when they
deserialize an object.  If you don't know a-priori what the type is supposed to
be, then you can't deserialize the object.

The biggest advantage is that untrusted input is limited in the damage it can
cause.  At best, the unmarshaler will crash the program (although a  well-
designed program will notify the user that input was malformed).

#### Register unmarshalers in some global space that each return a `Result<Box<Any>>` 

The marshalers will encode the type information into the YAML output as they
produce it.  These strings can be used as keys in a map, with the corresponding
unmarshalers as the values.  When the unmarshaler driver encounters a new object
in the YAML output, it can lookup the appropriate value in the map, and call it
on the top-level YAML object.  Assuming that marshal uses similar methods to
[serde](https://serde.rs/), that will be the only object that needs to be looked
up in the map; the unmarshaler will call sub-unmarshalers as needed directly. If
any of them fail, they can return the appropriate result code to indicate what
happened, which can be propogated up to the original caller.

Malformed input can trick a program to use the wrong unmarshaler, but just like
in the case above, there is a limit to the damage it can do; at most, the
unmarshaler will inform the user program that the input was bad, and cease to
work with it further.

#### Hack the foreign function interface

If all unmarshalers are marked as `#[no_mangle] pub extern`, then they'll fully
obey the C calling convention.  Since the C calling convention is well-known,
we can put the name of the unmarshaler that should be used in the YAML output
directly, and construct a call to it.  We don't need to bother with a global
unmarshaling map at all.

There are multiple issues with this method.  First, if the function doesn't
exist, then there will be a crash.  Second, this is a method for untrusted input
to call any arbitrary function.  While the function called would always be
passed the decoded YAML output, and therefore would have limits on what could be
passed  in, if the rust code was running as the superuser then the untrusted
input could still cause some kind of damage.  This could be mitigated by forcing
all unmarshalers to use some arbitrary suffix in their name, like 
10e7a452_ee40_4a2b_99f0_c96bc2a514cd.  If the driver found something that didn't
have the suffix, then it would know that the function wasn't an unmarshaler, 
which means that trying to call any of the standard operating system functions
would be impossible.  However, there is always the possiblity that someone will
figure out a way around the restriction and cause arbitrary code execution.

