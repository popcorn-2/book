# Implentation details

## amd64

The bootloader allocates a PML4 for the initial process, as well as preallocating 256 contiguous PDPTs. The upper half of the PML4 is mapped to point to the allocated PDPTs.

The `KTable` type contains the 256 PDPTs:
```rust,ignore
struct KTable {
	tables: NonNull<[Table<PDPT>; 256]>
}
```

On creation of a new `TTable`, the 256 upper entries are all initialised to point to the `KTable` tables. In addition, `TTable` enforces that modifications can only be directly made to the lower 256 entries. Since the `KTable` tables are mapped into every `TTable`, any modification to `KTable` will update all `TTable`s. The `CR3` register is then pointed at the correct `TTable` instance.
