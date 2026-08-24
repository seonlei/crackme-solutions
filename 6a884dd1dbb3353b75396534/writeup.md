# Two-byte patch solution
## In short
Disassemble it or debug, jump to main.exe+1D3CF, replace `mov rcx, rbx` to `xor rcx, rcx` and enter password `1000`.

## Right to the deep investigation
This file is SFX GZIP executable. When running it, main.lua can be found in %TEMP% directory.

[pic 1]

We'll patch math.random() function on the disk (or right in the memory because no protection was used) to make the key predictable. Let's try. At first, we can try to analyze public source of LuaRT library. In the library file (https://github.com/samyeyo/LuaRT/blob/61ababd46fe3bd9e833e3508d6526cdf03611d4a/src/core/lua/lmathlib.c#L582) the math_random() function was found. We can mention some string error messages in this function. After searching them in disassembler, the entry point for function was found.

[pic 2]

Green block contains error message. Tracing it back to purple block, we know that the file offset to this function is 1D2F8 (because 0x14001D2F8 - 0x140000000 = 0x1D2F8).

[pic 3]

Now we can jump to this function in the disassembler (or debugger if live patching).

[pic 4]

To return predefined random value, let's find project() function from source in assembly.

[pic 5]

As we can see, function accepts three arguments. In disassembler's pseudo-code listing, we can see it either.

[pic 6] [pic 7]

As pseudo-code includes remark that v6=rsi=rv (rv seems to stand for random vector), it seems that we can replace rsi with 0 to make seed pre-determined to the lower range number. For that, we'll need to patch assembly code . It can be done either with static patch or in-memory patching. We'll do it with replacing `mov rcx, rbx` to `xor rcx, rcx`. Now, if to launch the patched executable (or continue execution of the process if patch was proceeded in debugger like x64dbg) and enter the lower range number (1000), we'll get the victory message. Win.

[pic 8]