# IPC

IPC is a fundamental part of the design of Popcorn2, and underpins all interaction between processes and the outside world. To support this, a rich protocol called the Popcorn2 IPC Protocol (PIP) is used.

PIP at its core is an extension of Unix's "everything is a file" philosophy to an "everything is an object" philosophy. Instead of opening a file, a process creates an object, returning an object handle instead of a file descriptor. When created this way, a path to the object must be specified. Objects can also be returned as the result of a method call, in which case they may not have an underlying path.

## Interfaces

Objects implement one or more interfaces. An interface is defined in a PIP Source file, and contains a number of methods, each associated with a name and a numeric identifier.

Interfaces are registered with the kernel by passing it a PIP Binary file. This is normally handled by the `interfaced` daemon, which searches for `.pipb` files in `/System/proto` (TBD: other locations) and registers interfaces it finds.

### Core interfaces

Popcorn2 defines 

## Default objects

In the same way that UNIX systems provide a set of already open file descriptors on process start, Popcorn2 provides a set of already created objects.

Object handles `0`, `1` and `2` correspond to `stdin`, `stdout`, and `stderr`, and implement interfaces accordingly. Additionally, a `process` handle is opened at `3`.