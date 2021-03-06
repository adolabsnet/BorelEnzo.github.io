# Crack me 2 - Reverse

### [~$ cd ..](../)

The first Reverse challenge ([crack_me](../crack_me/)) was quite easy, but this one was a little bit more difficult as we could not easily get the extracted file. By loading the [program](crack_me_hard) in `r2`, we saw that the extracted binary was immediately `unlink`ed:

```
|     ||    0x00000a9f      e85cfdffff     call sym.imp.open          ; int open(const char *path, int oflag);
|     ||    0x00000aa4      8945dc         mov dword [rbp - local_24h], eax
|     ||    0x00000aa7      837ddc00       cmp dword [rbp - local_24h], 0
|     ||,=< 0x00000aab      790a           jns 0xab7
|     |||   0x00000aad      b800000000     mov eax, 0
|    ,====< 0x00000ab2      e9a1000000     jmp 0xb58
|    ||||   ; JMP XREF from 0x00000aab (main)
|    |||`-> 0x00000ab7      488d45e9       lea rax, qword [rbp - local_17h]
|    |||    0x00000abb      4889c7         mov rdi, rax
|    |||    0x00000abe      e8cdfcffff     call sym.imp.unlink        ; int unlink(const char *path);
|    |||    0x00000ac3      85c0           test eax, eax
|    |||,=< 0x00000ac5      740a           je 0xad1
|    ||||   0x00000ac7      b800000000     mov eax, 0
|   ,=====< 0x00000acc      e987000000     jmp 0xb58
```

For the previous challenge, we noticed that the extracted file came from the `.data` section:

```
objdump -s -j .data crack_me

crack_me:     file format elf64-x86-64

Contents of section .data:
 201000 00000000 00000000 08102000 00000000  .......... .....
 201010 00000000 00000000 00000000 00000000  ................
 201020 7f454c46 02010100 00000000 00000000  .ELF............
 201030 03003e00 01000000 30050000 00000000  ..>.....0.......
 201040 40000000 00000000 28110000 00000000  @.......(.......
 201050 00000000 40003800 09004000 1b001a00  ....@.8...@.....
 201060 06000000 04000000 40000000 00000000  ........@.......
 201070 40000000 00000000 40000000 00000000  @.......@.......
 201080 f8010000 00000000 f8010000 00000000  ................
 201090 08000000 00000000 03000000 04000000  ................
 2010a0 38020000 00000000 38020000 00000000  8.......8.......
 2010b0 38020000 00000000 1c000000 00000000  8...............
 2010c0 1c000000 00000000 01000000 00000000  ................
 ```

 For this 2nd part, we tried first to exploit a race condition to get the extracted file but it was unsuccessful. We guessed that the file would be extracted in a similar way, hence we ran the same command:

 ```
 objdump -s -j .data crack_me_hard

crack_me_hard:     file format elf64-x86-64

Contents of section .data:
 201000 00000000 00000000 08102000 00000000  .......... .....
 201010 00000000 00000000 00000000 00000000  ................
 201020 7bc4817a 703e4043 00000000 00000000  {..zp>@C........
 201030 00000000 00000000 00000000 00000000  ................
 201040 b389808a cecdcdcc cccccccc cccccccc  ................
 201050 cfccf2cc cdcccccc 4cc9cccc cccccccc  ........L.......
 201060 8ccccccc cccccccc e4ddcccc cccccccc  ................
 201070 cccccccc 8cccf4cc c5cc8ccc d7ccd6cc  ................
 201080 cacccccc c8cccccc 8ccccccc cccccccc  ................
 201090 8ccccccc cccccccc 8ccccccc cccccccc  ................
 2010a0 34cdcccc cccccccc 34cdcccc cccccccc  4.......4.......
 ```

 We guessed that the file had been XOR'ed with the byte `0xCC`. We then extracted the [content of the .data section](data_Section):

 ```bash
 objcopy -O binary -j .data crack_me_hard data_Section
 ```
and applied the XOR using `python`

```python
f = open("data_Section", "rb")
data = f.read()
f.close()
f = open("extracted_elf", "wb")
f.write("".join(chr(ord(x)^0xCC) for x in data))
f.close()
```


We knew that some garbage were put at the beginning of the extracted file (final extracted binary [here](40.elf)):
```
binwalk extracted_elf

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
64            0x40            ELF, 64-bit LSB shared object, AMD x86-64, version 1 (SYSV)
```

Let's open it in `r2`:

```
[0x00000580]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[0x00000580]> afl
0x00000528    3 23           sub.__cxa_finalize_232_528
0x00000550    2 16   -> 32   sym.imp.puts
0x00000560    2 16   -> 48   sym.imp.strlen
0x00000570    1 16           sub.__cxa_finalize_248_570
0x00000580    1 43           entry0
0x000005b0    4 50   -> 40   sub.__cxa_finalize_216_5b0
0x0000068a    9 138          sub.strlen_68a
0x00000714    7 96           main
[0x00000580]> sf main
[0x00000714]> pdf
/ (fcn) main 96
|   main ();
|           ; var int local_10h @ rbp-0x10
|           ; var int local_4h @ rbp-0x4
|           ; DATA XREF from 0x0000059d (entry0)
|           0x00000714      55             push rbp
|           0x00000715      4889e5         mov rbp, rsp
|           0x00000718      4883ec10       sub rsp, 0x10
|           0x0000071c      897dfc         mov dword [rbp - local_4h], edi
|           0x0000071f      488975f0       mov qword [rbp - local_10h], rsi
|           0x00000723      837dfc01       cmp dword [rbp - local_4h], 1 ; [0x1:4]=0x2464c45
|       ,=< 0x00000727      7f13           jg 0x73c
|       |   0x00000729      488d3d210100.  lea rdi, qword str.Usage:_crack_me_hard__flag_ ; 0x851 ; str.Usage:_crack_me_hard__flag_ ; "Usage: crack_me_hard <flag>" @ 0x851
|       |   0x00000730      e81bfeffff     call sym.imp.puts          ; int puts(const char *s);
|       |   0x00000735      b801000000     mov eax, 1
|      ,==< 0x0000073a      eb36           jmp 0x772
|      ||   ; JMP XREF from 0x00000727 (main)
|      |`-> 0x0000073c      488b45f0       mov rax, qword [rbp - local_10h]
|      |    0x00000740      4883c008       add rax, 8
|      |    0x00000744      488b00         mov rax, qword [rax]
|      |    0x00000747      4889c7         mov rdi, rax
|      |    0x0000074a      e83bffffff     call sub.strlen_68a        ; size_t strlen(const char *s);
|      |    0x0000074f      84c0           test al, al
|      |,=< 0x00000751      740e           je 0x761
|      ||   0x00000753      488d3d130100.  lea rdi, qword str.You_got_it_ ; 0x86d ; str.You_got_it_ ; "You got it!" @ 0x86d
|      ||   0x0000075a      e8f1fdffff     call sym.imp.puts          ; int puts(const char *s);
|     ,===< 0x0000075f      eb0c           jmp 0x76d
|     |||   ; JMP XREF from 0x00000751 (main)
|     ||`-> 0x00000761      488d3d110100.  lea rdi, qword str.Try_again_:_ ; 0x879 ; str.Try_again_:_ ; "Try again :(" @ 0x879
|     ||    0x00000768      e8e3fdffff     call sym.imp.puts          ; int puts(const char *s);
|     ||    ; JMP XREF from 0x0000075f (main)
|     `---> 0x0000076d      b800000000     mov eax, 0
|      |    ; JMP XREF from 0x0000073a (main)
|      `--> 0x00000772      c9             leave
\           0x00000773      c3             ret
```

The only interesting routine is `sub.strlen_68a`:

```
[0x00000714]> sf sub.strlen_68a
[0x0000068a]> pdf
/ (fcn) sub.strlen_68a 138
|   sub.strlen_68a ();
|           ; var int local_18h @ rbp-0x18
|           ; var int local_8h @ rbp-0x8
|           ; CALL XREF from 0x0000074a (main)
|           0x0000068a      55             push rbp
|           0x0000068b      4889e5         mov rbp, rsp
|           0x0000068e      4883ec20       sub rsp, 0x20
|           0x00000692      48897de8       mov qword [rbp - local_18h], rdi
|           0x00000696      488b45e8       mov rax, qword [rbp - local_18h]
|           0x0000069a      4889c7         mov rdi, rax
|           0x0000069d      e8befeffff     call sym.imp.strlen        ; size_t strlen(const char *s);
|           0x000006a2      4883f820       cmp rax, 0x20               ; "@" 0x00000020
|       ,=< 0x000006a6      7407           je 0x6af
|       |   0x000006a8      b800000000     mov eax, 0
|      ,==< 0x000006ad      eb63           jmp 0x712
|      ||   ; JMP XREF from 0x000006a6 (sub.strlen_68a)
|      |`-> 0x000006af      48c745f80000.  mov qword [rbp - local_8h], 0
|      |,=< 0x000006b7      eb42           jmp 0x6fb
|      ||   ; JMP XREF from 0x0000070b (sub.strlen_68a)
|     .---> 0x000006b9      488b55e8       mov rdx, qword [rbp - local_18h]
|     |||   0x000006bd      488b45f8       mov rax, qword [rbp - local_8h]
|     |||   0x000006c1      4801d0         add rax, rdx                ; '('
|     |||   0x000006c4      0fb608         movzx ecx, byte [rax]
|     |||   0x000006c7      488d153a0100.  lea rdx, qword str.000000000000400A00A4BB000A00A0B0 ; 0x808 ; str.000000000000400A00A4BB000A00A0B0 ; "000000000000400A00A4BB000A00A0B0" @ 0x808
|     |||   0x000006ce      488b45f8       mov rax, qword [rbp - local_8h]
|     |||   0x000006d2      4801d0         add rax, rdx                ; '('
|     |||   0x000006d5      0fb600         movzx eax, byte [rax]
|     |||   0x000006d8      31c1           xor ecx, eax
|     |||   0x000006da      488d154f0100.  lea rdx, qword str.scbKqqqqqqqqZTouWQpZxxoWByoZqRcM ; 0x830 ; str.scbKqqqqqqqqZTouWQpZxxoWByoZqRcM ; "scbKqqqqqqqqZTouWQpZxxoWByoZqRcM" @ 0x830
|     |||   0x000006e1      488b45f8       mov rax, qword [rbp - local_8h]
|     |||   0x000006e5      4801d0         add rax, rdx                ; '('
|     |||   0x000006e8      0fb600         movzx eax, byte [rax]
|     |||   0x000006eb      38c1           cmp cl, al
|    ,====< 0x000006ed      7407           je 0x6f6
|    ||||   0x000006ef      b800000000     mov eax, 0
|   ,=====< 0x000006f4      eb1c           jmp 0x712
|   |||||   ; JMP XREF from 0x000006ed (sub.strlen_68a)
|   |`----> 0x000006f6      488345f801     add qword [rbp - local_8h], 1
|   | |||   ; JMP XREF from 0x000006b7 (sub.strlen_68a)
|   | ||`-> 0x000006fb      488b55e8       mov rdx, qword [rbp - local_18h]
|   | ||    0x000006ff      488b45f8       mov rax, qword [rbp - local_8h]
|   | ||    0x00000703      4801d0         add rax, rdx                ; '('
|   | ||    0x00000706      0fb600         movzx eax, byte [rax]
|   | ||    0x00000709      84c0           test al, al
|   | `===< 0x0000070b      75ac           jne 0x6b9
|   |  |    0x0000070d      b801000000     mov eax, 1
|   |  |    ; JMP XREF from 0x000006ad (sub.strlen_68a)
|   |  |    ; JMP XREF from 0x000006f4 (sub.strlen_68a)
|   `--`--> 0x00000712      c9             leave
\           0x00000713      c3             ret
```

The code here is quite simple as it's a simple loop applying a XOR between each character of 2 constant strings, `str.000000000000400A00A4BB000A00A0B0` and `str.scbKqqqqqqqqZTouWQpZxxoWByoZqRcM`. We decocded it using `python` and got our flag:

```python
str1 = int("000000000000400A00A4BB000A00A0B0".encode("hex"), 16)
str2 = int("scbKqqqqqqqqZTouWQpZxxoWByoZqRcM".encode("hex"), 16)
hex(str1^str2)[2:-1].decode("hex")
  'CSR{AAAAAAAAnd_4ga1n::_gr8_j0b!}'
```
