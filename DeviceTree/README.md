

# What is a Flattened Device Tree or Device Tree Blob (FDT)?
This is the binary representation of the device tree. It is sometimes refered to as a device-tree blob.

It's name comes from the flattening process of taking a tree structure and representing it as a single array of bytes following a specific format outlined in the documentation [device tree specifications](https://devicetree-specification.readthedocs.io/en/stable/) for a specific version. All device tree binaries start with a header that can be used to identify the device tree version used. 

# Device Tree Confusion
The term "device tree" refers to the heiarchal tree representation of nodes describing the hardware layout of a machine. 

> [!NOTE]
> There is an important distinction between the [flattened device tree](#what-is-a-flattened-device-tree-or-device-tree-blob-fdt)(FDT) and the device tree as described in this section. They differ in that the FDT is the device tree represented as an array of bytes.

Some operating systems will unflatten the device tree into a data structure that can be modified to add and remove nodes. 

# Navigating the Device Tree

Before reading a device tree it is important to ensure you have the correct mental model of its layout to prevent miunderstandings. 
## Actual FDT Structure Layout
![Actual FDT Structure Layout](https://devicetree-specification.readthedocs.io/en/stable/_images/graphviz-b5aa4014171b1529d167469518b57197f06cdf75.png)

The FDT blob is a structure of contiguous memory that should be viewed like a stream of bytes. The block starts with the `ftd_header` structure which describes the characteristics of the device tree including it's overall size, the size of each defined section, and the offsets from the fdt's address where those sections can be found. It also includes references necessary to understand the environment and tree format including the device tree version used, the physical cpu running the boot sequence, and the layout of reserved physical memory.

## Navigating The FDT
Since the device tree is in congtiguous memory we can access anything in that memory given an offset from the device tree's memory location. This standard is used frequently in properties within the device tree nodes for things like describing register blocks by their starting address and size.

Each node should be considered it's OWN address space. A nodes children will use the parents address space to describe address locations in their property. In these cases and address is the offset from the beginning of the parent node.

Some properties contain addresses values that describe the offset from the beginning of the FDT regardless of where they are found in the device-tree. The \<phandle> data type used in some properties is a reference to that nodes offset from the FDT root node's start address. Additionally any node with a `virtual-reg` property contains the offset from the root node to the value of the first register described in the `reg` property.

Another noteworthy exception is the `ranges` property which provides a mapping that can be used by arbitrary node K's children to calculate the offset of the child node within the address space of node K's parent.
See [Device Tree Basics: 2.3.8. ranges](https://devicetree-specification.readthedocs.io/en/stable/devicetree-basics.html#properties)

### Root Node
The root node is the starting point of the device tree and is normally refered to the path `/`. Normally it will have at least one child node.

### Child Node
Any node that is a decendent of the root node. Each node is a new address space for its children.

# Reading Integer & Address Values

See: [Device Tree Basics: 2.2.4.2. Property Values](https://devicetree-specification.readthedocs.io/en/v0.2/devicetree-basics.html)

- Values are stored in *big-endian* format
- **\<u64>** values span two cells
    - `fdt64_to_cpu(x)` function in `libfdt.h` will read a 64 bit value in *bit-endian* and return it as a 64-bit integer in the cpu's endieness.
    > [!CAUTION]
    > #### Using `printf`
    > Do not use printf to read a value of a different endiness. Since the bit order is reversed the formatter will give you the wrong character representation of that nibble. 
    >
    > For example the value decimal value 13 has a hex value of `D`. This value has a representation of `1011b` in big-endian. If you attempt to read this nibble with printf on a little endian machine it will inturrpret your bit order in reverse giving you a decimal value of `11` and a hex value of `B` which is incorrect.

## Useful Functions

### Reading Integers and Address'

Values are stored in the device tree in big-endian format. A series of functions have been provided that convert a given integer value between big and little endian representation. 
> [!CAUTION]
> When reading 64-bit values make sure you read the entire value then use `fdt64_to_cpu` to convert it at once. They are stored into two consecutive 32-bit cells and intended to be read in the same way. Do not read them as two 32-bit values because the bit order will be corrupted by the process if the CPU is not big endien. 
> Example: 
>```c
> uint64_t addr = fdt64_to_cpu(((fdt64_t *)prop)[addrIdx]);
> uint64_t size = fdt64_to_cpu(((fdt64_t *)prop)[sizeIdx]);
> ```
> The `((fdt64_t *)prop)` treats the address as a `fdt64_t` type ensuring it reads the correct number of bytes for that value. The dereferencing by index will then get the value at the location of `prop` plus an offset of `size*index` bytes which points to the value we want to convert. The number represented is in big endian format and we use the `fdt64_to_cpu` to convert the `fdt64_t` to `uint64_t` in the CPU's endieness.
- `fdt32_to_cpu(x)`
    - Converts a `fdt32_t` bigendian 32-bit integer value from the device tree and returns the value translated for the cpu's endieness. 
- `fdt64_to_cpu(x)`
    - Converts a `fdt64_t` bigendian 64-bit integer value from the device tree and returns the value translated for the cpu's endieness. 
-

