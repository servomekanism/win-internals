### PE anatomy
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
how we locate the PE header, as an offset from the MS-DOS header.

2. `PE header`: contains information such as location and sizes of the code and
data areas, matching Operating System version, initial stack size and more.
The PE `[header structure prototype]`[1] is this:
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

[1]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_nt_headers (IMAGE_NT_HEADERS)
[IMAGE_FILE_HEADER]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_file_header
[IMAGE_OPTIONAL_HEADER]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_optional_header

