### PE anatomy
1. MS-DOS header
2. PE header: contains information such as location and sizes of the code and
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


[1]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_nt_headers (IMAGE_NT_HEADERS)
[IMAGE_FILE_HEADER]: https://docs.microsoft.com/en-gb/windows/desktop/api/winnt/ns-winnt-_image_file_header
