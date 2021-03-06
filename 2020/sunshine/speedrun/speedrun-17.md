# speedrun-17

## Task

nc chal.2020.sunshinectf.org 30017

File: chal_17

Tags: binary analysis

## Solution

```nasm
[0x00000850]> pdf@main
            ; DATA XREF from entry0 @ 0x86d
┌ 135: int main (int argc, char **argv, char **envp);
│           ; var int64_t var_10h @ rbp-0x10
│           ; var uint32_t var_ch @ rbp-0xc
│           ; var int64_t canary @ rbp-0x8
│           0x000009b9      55             push rbp
│           0x000009ba      4889e5         mov rbp, rsp
│           0x000009bd      4883ec10       sub rsp, 0x10
│           0x000009c1      64488b042528.  mov rax, qword fs:[0x28]
│           0x000009ca      488945f8       mov qword [canary], rax
│           0x000009ce      31c0           xor eax, eax
│           0x000009d0      bf00000000     mov edi, 0                  ; time_t *timer
│           0x000009d5      e816feffff     call sym.imp.time           ; time_t time(time_t *timer)
│           0x000009da      89c7           mov edi, eax                ; int seed
│           0x000009dc      e8fffdffff     call sym.imp.srand          ; void srand(int seed)
│           0x000009e1      e84afeffff     call sym.imp.rand           ; int rand(void)
│           0x000009e6      8945f4         mov dword [var_ch], eax
│           0x000009e9      488d45f0       lea rax, [var_10h]
│           0x000009ed      4889c6         mov rsi, rax
│           0x000009f0      488d3df30000.  lea rdi, [0x00000aea]       ; "%d" ; const char *format
│           0x000009f7      b800000000     mov eax, 0
│           0x000009fc      e80ffeffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x00000a01      8b45f0         mov eax, dword [var_10h]
│           0x00000a04      3945f4         cmp dword [var_ch], eax
│       ┌─< 0x00000a07      7507           jne 0xa10
│       │   0x00000a09      e84cffffff     call sym.win
│      ┌──< 0x00000a0e      eb19           jmp 0xa29
│      ││   ; CODE XREF from main @ 0xa07
│      │└─> 0x00000a10      8b45f0         mov eax, dword [var_10h]
│      │    0x00000a13      8b55f4         mov edx, dword [var_ch]
│      │    0x00000a16      89c6           mov esi, eax
│      │    0x00000a18      488d3dce0000.  lea rdi, str.Got:__d__Expected:__d ; 0xaed ; "Got: %d\nExpected: %d\n" ; const char *format
│      │    0x00000a1f      b800000000     mov eax, 0
│      │    0x00000a24      e897fdffff     call sym.imp.printf         ; int printf(const char *format)
│      │    ; CODE XREF from main @ 0xa0e
│      └──> 0x00000a29      90             nop
│           0x00000a2a      488b45f8       mov rax, qword [canary]
│           0x00000a2e      644833042528.  xor rax, qword fs:[0x28]
│       ┌─< 0x00000a37      7405           je 0xa3e
│       │   0x00000a39      e872fdffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0xa37
│       └─> 0x00000a3e      c9             leave
└           0x00000a3f      c3             ret
```

This binary expects a number that has to match the first random number generated by `sym.imp.rand` that is seeded with the return value from `sym.imp.time`.

If you do a simple:

```bash
$ for x in $(seq 1 10);do echo 1 | nc chal.2020.sunshinectf.org 30017;done
Got: 1
Expected: 1192309984
Got: 1
Expected: 1192309984
Got: 1
Expected: 1192309984
Got: 1
Expected: 1959586606
Got: 1
Expected: 1959586606
Got: 1
Expected: 1959586606
```

You'll notice that they are the same, at least sometimes when you are quick enough.

You can solve this with a shell script:

```bash
last=1
for x in $(seq 1 10);do
  buf=$(echo "$last" | nc chal.2020.sunshinectf.org 30017)
  last=$(echo "$buf" | grep Expected | awk '{print $2;}')
  printf "$buf\n"
done
```

Executing it:

```bash
$ bash solve.
Got: 1
Expected: 82551575
sun{unholy-confessions-b74c1ed1f1d486fe}
```
