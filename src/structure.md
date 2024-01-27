# Structure

## The Core

The core popcorn2 kernel is split across three main crates: `kernel`, `kernel_hal` and `kernel_api`.

### `kernel_hal`

`kernel_hal` contains all architecture specific code required for the core kernel to operate, and no more. This means it specifically **does not include** any peripheral [^1] drivers, with the exception of a UART controller, to aid in debugging the preboot environment. This includes paging code, platform initialisation, etc.

### `kernel_api`

### `kernel`

## Kernel modules

[^1]: The line is not clear cut. For example, on x86, the x87 was originally a coprocessor, however it would be very difficult to write userspace drivers for it due to how it integrates with context switching. Therefore this would be something that would be considered close enough to part of the architecture that it should be in the HAL.
