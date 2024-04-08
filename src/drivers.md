# Popcorn2 Driver Model

Drivers are located in `/System/drivers`, [should there be a non system folder too?] and have a name of the format `[TBD].exec`.

Device manager initially starts the root driver for the system as a fixed pseudo-device, and then a recursive enumeration procedure is followed.

For each driver, [an IPC channel] is initialised between the device manager and the driver. The device manager starts by sending an `INIT` command to the driver. The driver responds with
a driver descriptor object. The `NEXT` command is then sent repeatedly. If the driver is a bus-type driver, it responds to each request with a child descriptor object, returning `DONE` 

## Driver descriptor object

> or should this be part of the binary or something instead?

## Child descriptor object

---

Loosely based on [UDI device enumeration](https://wiki.osdev.org/User:Gravaera/UDI_Device_Enumeration).

---

## IRQ VM idea????

Using event queues for userspace IRQ handling poses a latency problem. Due to scheduler design or something, the driver may not get CPU time quickly enough and for long enough to handle IRQs
immediately. For something like a PS/2 keyboard or mouse controller, this poses a problem due to the ease with which data can be overwritten under high workload.

It is less immediately obvious why approaching this with POSIX style signals is not the most optimal idea. This could be designed to reduce the latency requirements, but it adds some overhead
and complexity in the scheduler design. It also requires driver devlopers to ensure all interrupt code is signal safe, as well as ensuring proper syncronisation between the IRQ handlers and the
main driver code.

A potentially silly idea to solve this is to define a basic virtual machine to execute IRQ handlers, designed to be versatile enough to read IRQ data and send EOIs, but not complex enough to
pose a security risk (as it is run directly in the kernel) or lock up the kernel. Programs are limited to [TBD] bytes, and are compiled and statically checked by the kernel upon creation.

IRQ handlers are expected to read any required data from the device, and push it into the **data queue**,
where the main driver process will be able to read and process it.

There is an implicit return at the end of all programs.

Handlers may be run with interrupts enabled. A `CLI` instruction is provided to be called before any
end-of-interrupt is signalled, to prevent recursive calls that result in stack overflow. Stack overflows
will result in the entire driver process being terminated.

The machine has X and Y registers, both 64 bits in size. The X register is populated with the **logical
interrupt number** (specific to the driver) upon start. The Y register is zeroed.

> Is 64 bits fine?

| Nmemonic | Operation |
| -------- | --------- |
| `PUSH r` | Pushes the contents of register `r` onto the **data queue** |
| `RET`    | Exits the interrupt handler |
| `CLI`    | Disables interrupts |
| `COPY r` | Copies the contents of register `r` into the other register |
| `IMM r, imm`  | Loads register `r` with `imm` |
| `BEQ r, imm, i` | If the contents of register `r` is equal to `imm`, jump `i` instructions ahead. |
| `BLE r, imm, i` | If the contents of register `r` is less than or equal to `imm` jump `i` instructions ahead. |
| `INB r, p` | (x86 only) Performs a leftshift of register `r` by 1 byte, before reading a byte from port `p` into the lowest byte. |
| `INW r, p` | (x86 only) Performs a leftshift of register `r` by 2 bytes, before reading 2 bytes from port `p` into the lowest 2 bytes. |
| `IND r, p` | (x86 only) Performs a leftshift of register `r` by 4 bytes, before reading 4 bytes from port `p` into the lowest 4 bytes. |
| `OUTB p, r` | (x86 only) Writes the lowest byte of register `r` to port `p`, then rightshifts the register by 1 byte |
| `OUTW p, r` | (x86 only) Writes the lowest 2 bytes of register `r` to port `p`, then rightshifts the register by 2 bytes |
| `OUTD p, r` | (x86 only) Writes the lowest 4 bytes of register `r` to port `p`, then rightshifts the register by 4 bytes |

> TODO: memory operations, improve branching, basic arithmetic(?)

IO port and memory access is only allowed to ports and memory the driver has prior permission to access. If access to required IO/memory is lost while an interrupt handler is in place, it will be removed and the driver notified [mechanism tbd].

### Example routines

(`;` used to begin a line comment)

#### i8042 PS/2 controller handler
```armasm
PUSH X      ; So the driver knows which device this interrupt is from
INB Y, 0x60 ; Read from the i8042 data port
PUSH Y      ; Pass the incoming data onto the driver
; The i8042 has no EOI routine so we just implicitly return here
```

#### i8259 PIC EOI handler

Assume here that logical interrupt numbers map to standard dual PIC lines.
This example does **not** handle spurious interrupts.

```armasm
CLI          ; Disable interrupts before sending the EOI to prevent any loops
IMM Y, 0x20  ; Load the Y register with the EOI command
BLE X, 7, 2  ; If interrupt lines 0-7, jump 2 instructions ahead
OUTB 0xA0, Y ; Otherwise send the EOI command to the second PIC
OUTB 0x20, Y ; In all cases, send the EOI command to the first PIC
```

---

Loosely modelled on RP2040 PIO blocks

---
