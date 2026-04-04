# Manual PE Loader – Windows Internals Study

**Author:** Fabio Baensch (GitHub: KernelPhantom-010)  
**Type:** Low‑level systems programming / Reverse Engineering

---

## 🎯 The Hook – Why This Project Matters

This project **re‑implements core parts of the Windows PE loader** – the component responsible for starting any executable on Windows. By manually parsing PE headers, performing relocations (ASLR bypass), resolving imports, and jumping to the entry point, I demonstrate:

- Deep understanding of the **Portable Executable (PE) format**
- Ability to work with **low‑level memory management** (`VirtualAlloc`, `VirtualProtect`)
- Practical knowledge of **relocation tables** and **import address tables (IAT)**
- Experience in **bypassing standard OS loading mechanisms** for research purposes

This is **not a beginner project**. It shows that I can read Microsoft documentation, debug complex structures, and build a working loader from scratch.

![demonstration](https://s4.ezgif.com/tmp/ezgif-4a5339b7da97dff4.gif)
..Or Video this Link (if the GIF won't load)
[▶ Watch Demo](./2026-04-03%2017-00-55.mp4)
---

## ⚠️ Disclaimer

This code is provided **strictly for educational and research purposes** – specifically for learning about Windows internals, reverse engineering, and PE format analysis. Because there are many bad people out there, I won't publish the source code on purpose to prevent misuse. The author does not condone misuse. Compile and run only on systems you own or have explicit permission to test.

---

## 🧠 How the PE Loader Works – Step by Step

Below is the exact sequence of operations my loader performs. Each step corresponds to a phase of the Windows loader’s job.

| Step | Action | Key Structures / APIs |
|------|--------|------------------------|
| 1 | **Parse DOS & NT headers** – Validate MZ and PE signatures, locate NT headers | `IMAGE_DOS_HEADER`, `IMAGE_NT_HEADERS`, `e_lfanew` |
| 2 | **Create file mapping object** – Map the PE file into memory as read‑only | `CreateFileA`, `CreateFileMappingA`, `MapViewOfFile` |
| 3 | **Allocate memory for the image** – Reserve space of `SizeOfImage` bytes | `VirtualAlloc(NULL, SizeOfImage, MEM_COMMIT, PAGE_READWRITE)` |
| 4 | **Copy sections** – For each section, copy raw data to the correct virtual address | `IMAGE_SECTION_HEADER`, `PointerToRawData`, `VirtualAddress` |
| 5 | **Perform relocations (ASLR bypass)** – If actual base ≠ preferred base, patch addresses using `.reloc` section | `IMAGE_BASE_RELOCATION`, `IMAGE_REL_BASED_DIR64` / `HIGHLOW` |
| 6 | **Resolve imports** – Load required DLLs and fill the IAT with actual function addresses | `IMAGE_IMPORT_DESCRIPTOR`, `OriginalFirstThunk`, `FirstThunk`, `GetProcAddress` |
| 7 | **Set correct memory protections** – Apply `PAGE_EXECUTE_READ`, `PAGE_READWRITE`, etc. based on section characteristics | `VirtualProtect`, `IMAGE_SCN_MEM_*` flags |
| 8 | **Jump to entry point** – Call `AddressOfEntryPoint` as a function | `(int(*)()) (alloc_addr + ep_rva)` |



---

## 🔗 Resources Used

Two main references helped me build this loader:

1. **Microsoft Official PE Format Documentation**  
   [https://docs.microsoft.com/en-us/windows/win32/debug/pe-format](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)  
   *The definitive source – every field, every structure.*

2. **0xrick’s PE Loading Tutorial**  
   [https://0xrick.github.io/win-internals/pe2/](https://0xrick.github.io/win-internals/pe2/)  
   *A practical walkthrough that helped me understand the correct order of operations.*

Because many online tutorials contained conflicting information (especially about casting `PIMAGE_SECTION_HEADER`, handling `IMAGE_DATA_DIRECTORY`, and distinguishing between `OriginalFirstThunk` and `FirstThunk`), I **cross‑referenced multiple sources and validated each step** – including using AI to double‑check logic. This project taught me how to resolve contradictory documentation through systematic testing.

---
## 🚀 What This Says to Recruiters
✅ Understands Windows internals (PE format, virtual memory, relocations, imports)

✅ Can write low‑level C/C++ code that interacts directly with the OS API

✅ Able to debug complex memory layouts and pointer arithmetic

✅ Demonstrates curiosity and ability to learn from conflicting documentation

Ready for roles in: Security Research, Reverse Engineering, Low‑Level Systems Development, Malware Analysis (defensive), EDR/AV internals.

## 🛠️ Compile & Test

### Compile a 64‑bit test program (`test.c`)

```c
#include <windows.h>
#pragma comment(lib, "user32.lib")

int main() {
    MessageBoxA(NULL, "PE loaded successfully!", "PE Loader Test", MB_OK);
    printf("Hello from PE Loader! :)");
    return 0;
}
```
Compile it with cl /EHsc peloader.cpp .. I think that's it! Thanks for reading. 
