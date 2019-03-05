### PE anatomy
The most descriptive picture was [this one][0]. Don't get lost in the details,
follow this post.

0. `RVA`: Relative Virtual Address is an offset to some item/table within the
file. When the windows loader maps the file into memory, it maps it to some base
address. The `Base Address` corresponds to the `RVA` 0. Let's say that the loader
maps the PE at the base address `10000`. If a table within the memory map is at
`Virtual Address` (i.e. mapped at that address) at `10464`, then the `RVA` of that
table is `464`.

`Base Address + RVA = Virtual Address (in runtime)`

Remember that the Windows Loader maps the file continouusly in memory.

1. `MS-DOS header`: MS-DOS stub that prints the usual message. The first byte that
gets mapped when the PE file is mapped in memory is the first byte of the MS-DOS
header. The MS-DOS header contains the exact offset of the PE header. This is
how we locate the PE header, as an offset from the MS-DOS header. Specifically
the `e_lfanew` field of the MS-DOS header is the `RVA` of the PE headeT.
To get a pointer to the PE header in memory, we add the `pDosHeader->e_lfanew` 
to the `image base`:

`pNtHeader = pImageBase + pDosHeader->e_lfanew`

To get a pointer to the `image base` when loaded in memory, we need to map the
file with the [CreateFileMapping][2] and [MapViewOfFile][3]. For 64-bit we need
to cast this carefully according to [MSDN][4].

2. `PE header`: contains information such as location and sizes of the code and
data areas, matching Operating System version, initial stack size and more.
The PE [header structure prototype][1] is this:
```
typedef struct _IMAGE_NT_HEADERS {
  DWORD                   Signature;
  IMAGE_FILE_HEADER       FileHeader;
  IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

`Signature`: 4-byte signature idnetifying the file as a PE image. the magic
bytes are "PE\0\0"

`FileHeader`: an [IMAGE_FILE_HEADER] structure that specifies the file header.

`OptionalHeader`: an [IMAGE_OPTIONAL_HEADER] structure that specifies the
optional file header.

So, if you want to verify that a file is indeed a PE file, you can do:

`pNtHeader->Signature == "PE\0\0"`

`PE\0\0` is `#define`-d as `IMAGE_NT_SIGNATURE`

3. `IMAGE_FILE_HEADER`: Check the link for more information. The important
fields are the: `NumberOfSections`, `TimeDateStamp`, `SizeOfOptionalHeader`.

4. `IMAGE_OPTIONAL_HEADER`: Check the link for more information. Many fields are
of interest to us here:
    * `SizeOfCode`: total size of all code sections. Normally usually one, the
    .text section.
    * `AddressOfEntryPoint`: RVA, the address of where the loader will begin
    executing. In normal exe/pe files should point in the .text section.
    * `BaseOfCode`: RVA where the file's code sections begin. Usually `0x1000`.
    * `BaseOfData`: self-explanatory RVA. Typically last in memory.
    * `ImageBase`: The assumed memory location from the linker that the
    executable will be loaded in memory. Usually `0x40000`. If you use
    `dumpbin.exe /headers filename.exe` and you see the flag `Dynamic base` at
    the DLL characteristics under the `OPTIONAL HEADER VALUES`, this means that
    the executable `ImageBase` is randomized at load.
    * `SectionAlignment`: each section should start at a multiple of this value.
    the default is `0x1000`.
    * `SizeOfImage`: Total size of the of the portions of the image that the
    loader has to take care of. It is the size of the region starting at the
    image base up to the end of the last section. The end of the last section is
    rounded up to the nearest `SectionAlignment` section.
    * `SizeOfHeaders`: Size of the PE header and the section table.
    * `SubSystem`: Type of the subsystem that runs the executable like, `Native`
    (1), `Windows_GUI`(2), `Console` (3), `OS2_CUI` (5), `POSIX` (6).
    * `DllCharacteristics`: Flags that indicate when the DLL's initialization
    function will be called (DllMain).
    * `NumberOfRvaAndSizes`: Number of entries in the DataDirectory array
    (below) almost always 16.
    * `IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]`: An
    array of `IMAGE_DATA_DIRECTORY` structures. Initial array elements contain RVA
    and sizes of important section within the file. The first element of the
    array is always the address and the size of the exported function table (if
    the file exports any functions, like a dll). The second array is the address
    and size of the imported function table, etc. For more information check the
    `IMAGE_DIRECTORY_ENTRY_XXX` `#define`s in WINNT.H. Note the
    IMAGE_DIRECTORY_DEBUG element that can be useful. `.debug` section is the only
    section that is excluded from the hash signature generation of the file[!] [5]

#### Recap
We have the `MS-DOS` header, the `PE` header that is a structure and contains the 
`Signature` and two structures: `IMAGE_FILE_HEADER` and
the`IMAGE_OPTIONAL_HEADER`.

Following these two headers and before the actual data within the sections,
there is the `Section Table`.

5. `Section Table`: essentially is a "phone book" containing information about
each section in the image. The sections are sorted by their `RVA`s. The
information includes its type, its size and its location elsewhere in the file.
In essence, each `Section Table` entry stores the address of where the file's
raw data has been mapped into memory; memory address ranges whithin the process
address space.
    * the `Section Table` programmatically is actually an array of
    `IMAGE_SECTION_HEADER` structures. The number of elements in this array is
    given in the `PE` header itself, in the:
    `IMAGE_NT_HEADER.FileHeader.NumberOfSections` field
    * this `NumberOfSections` shows that the `Section Table` contains one
    `IMAGE_SECTION_HEADER` for every section.
    * `IMAGE_SECTION_HEADER`: The struct declaration is the following:
    
    ```
    typedef struct _IMAGE_SECTION_HEADER {
        BYTE Name[IMAGE_SIZEOF_SHORT_NAME];
        union {
            DWORD PhysicalAddress;
            DWORD VirtualSize;
        } Misc;
        DWORD VirtualAddress;
        DWORD SizeOfRawData;
        DWORD PointerToRawData;
        DWORD PointerToRelocations;
        DWORD PointerToLinenumbers;
        WORD  NumberOfRelocations;
        WORD  NumberOfLinenumbers;
        DWORD Characteristics;
        } IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
    ``` 
    
    `Name`: 8-byte null padded utf-8 string contains the section name. e.g.:
    `.text`, `.reloc`, etc.
    
    `Misc.PhysicalAddress`: The file address.

    `Misc.VirtualSize`: The size of the section when loaded into memory.



[0]: https://upload.wikimedia.org/wikipedia/commons/1/1b/Portable_Executable_32_bit_Structure_in_SVG_fixed.svg
[1]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_nt_headers (IMAGE_NT_HEADERS)
[IMAGE_FILE_HEADER]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_file_header
[IMAGE_OPTIONAL_HEADER]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_optional_header
[2]: https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-createfilemappinga
[3]: https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-mapviewoffile
[4]: https://docs.microsoft.com/en-us/windows/desktop/winprog64/rules-for-using-pointers
[5]: https://docs.microsoft.com/en-us/windows/desktop/debug/pe-format#the-debug-section
