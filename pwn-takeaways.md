\>\> https://ir0nstone.gitbook.io/notes <<

\>https://github.com/ljagiello/ctf-skills/tree/main/ctf-pwn<

https://syscall.sh/ \<-- to niti ni tolk pomembn (pač kaj naredi in kako se uporablja `syscall` instruction (aka `int 0x80`))

Hardware fault injection: turn the power down for e.g. one specific instruction to skip it

Some boards have a voltage regulator, where you can just wire your own thingy on to trick it

do not underestimate fuzzing, especially parsers

rev c# - [dotpeek](https://www.jetbrains.com/decompiler/) -- na tej temi [ysoserial.net](https://github.com/pwntester/ysoserial.net) in [ysonet](https://github.com/irsdl/ysonet)

ldd - view versions of ld and glibc - asterisk, view where they are linked to, not the version!
    - also, to kaže dependencije depencencijev, tko da ne bo nujno required od osnovnega elfa, ampak lahko je 
      dependency od depencija

pwninit - use the correct libc

objdump -d \<stuff>

(objdump --disassembler-color=on -M intel -d $@)

also readelf

zalaufi [ropr](https://github.com/Ben-Lichtman/ropr)!!! (also, [one_gadget](https://github.com/david942j/one_gadget)) ((ta najbl klasično orodje - npr. za ctf unplugged ipd. je [ropgadget](https://github.com/jonathansalwan/ropgadget)))
- GOT address of libc means the start of the RW section -- https://david942j.blogspot.com/2017/02/project-one-gadget-in-glibc.html

-- also also pwntools also zna iskat gadgete - https://docs.pwntools.com/en/stable/rop/rop.html

https://dogbolt.org/ (decomp) in https://godbolt.org/ (comp)

sicer za decomp [ida](https://hexrays.su/) / [ghidra](https://github.com/nationalsecurityagency/ghidra), [angr](https://angr.io/) bi znal bit uporaben še posebi za kkšn rust

[pwntoolse](https://docs.pwntools.com/en/stable/) že poznamo

pwn template (tudi iz dockerfila vzame), pwninit ti zalaufa patchelf
- sicer pa dockerfile pač docker exec -it \<container> bash in potem docker cp \<container>:/path ./path

checksec !!!

[pwndbg](https://pwndbg.re/stable/)
- x/n -> prints n things -> if you have e.g. x interpreter and that doesn't work (it thinks the value is an addr or sth), * and & work (x &interpreter)
- p(/x)
- tele
- ni (next instruction)
- si (step into)
- finish (go until ret of current function)
- start -> very useful when you have no main
- set follow-fork-mode child / parent --- za forke
- disass \<func>
- core \<coredump>

- context
- stack
- bins
- vis
- tls
- break (info breakpoints / del \<num>)
- run (can have redirects (\<))

- file
- target remote localhost:1234
- attach \<pid>
- thread \<cifra>

- info npr. functions
- show npr. architecture, set npr. architecture i386

[gef](https://github.com/bata24/gef) tudi obstaja

in na tej temi tudi [vagd](https://github.com/gfelber/vagd) in [docker-dbg](https://github.com/junron/docker-dbg/tree/main)

-- random stvar:

    (cat solve/solve.bin; cat) | ./app --- stdin je še vedno odprt

    ulimit -c unlimited
    Run your exploit: ./app \< solve/solve.bin
    Once it segfaults, open the core dump: gdb ./app core
    Run bt (backtrace).

--

https://wiki.osdev.org/System_V_ABI
- return value is in RAX
- args: rdi, rsi, rdx, rcx, r8, r9
(destination, source)

movaps causes a SIGSEGV if RSP is not 16 byte aligned

SIGBUS -> fajn je pop-at RBP tolkrat kt ga PUSH-as

RBP, RSP, RIP
--> RIP relative je probaby v bss-ju

man x86-\<sth> ([ttmo-O/x86-manpages](https://github.com/ttmo-O/x86-manpages))

https://www.felixcloutier.com/x86/$instruction

če maš system("/bin/sh") si že zmagal(a)
    -> system(neki) naredi sh -c "neki"

#include \<unistd.h>

c headerji so v /usr/include

stack itak veš da gre višji del -> nižjem
stack "frame":

\[local vars]

\[rbp]

\[return address]

wonky ass arhitekture: qemu, gdb-multiarch
- npr. qemu-riscv32-static \<myapp> in ga lepo zalaufa
- debug: qemu-riscv32-static -g \<port> \<myapp> (čaka za gdb connection) -- sicer sta tud -s (odpre gdbserver na 1234) in -S (čaka na connection) (če tadva moraš oba dat)
    - gdb -ex "target remote localhost:\<port>" -ex "set architecture riscv:rv32" -ex "file \<myapp>"
        - -ex : laufi to komando

- če ne zna najdt librarijev: export QEMU\_LD\_PREFIX=/usr/aarch64-linux-gnu npr

also, 8 byte stvari so 8 byte aligned v memoriju

ASLR obstaja
--> leakej pointer
- PIE -> ASLRamo še binary; če ne, je na statični poziciji

disable ASLR
echo 0 | sudo tee /proc/sys/kernel/randomize\_va\_space

enable ASLR
echo 2 | sudo tee /proc/sys/kernel/randomize\_va\_space

če daš gdb \<app> bo vedno mel aslr off
pol pa pač attachaš, sicer pa set disable-randomization off

default kaj naredi gcc (a so canariji by default, kdaj, ...) nastavijo distroji; nekateri (khm khm debian) še vedno ne uporablajo canarijev

stack canary:
- na stack je pushan nek random thingy iz kernel-set thread local space-a (note: 🦑), kernel ga zgenerira, ampak je samo *per process* unique
- uporaba zgleda kot 
```asm
začetek:
mov    rax,QWORD PTR fs:0x28
mov    QWORD PTR [rbp-0x8],rax

konec:
mov    rax,QWORD PTR [rbp-0x8]
sub    rax,QWORD PTR fs:0x28
je     11c5
call   1040 <__stack_chk_fail@plt>
leave (na 11c5)
ret
```
- 7 random bytov + null da ne moreš leakat
- to se na konc funkcije prever, če ni ok -> bad things
- -fstack-protector-strong, -fno-stack-protector
- kako beatat canary? usually leak, brute force questionably

if you have arbitrary file read --> /proc/self!!!! /proc/self/fd, /proc/self/maps, ...

`test eax eax` -> ne nastavi registrov, samo zero flag
- npr. `jeq` potem, če je `eax = 0`

`xor eax eax` cleara rax; vse 32 bitne inštrukcije tudi zero-ajo višje dele 64-bitnih registrov

fino je vedt da je `0x90` 1-byte `nop`

včasih redko bi znalo bit koristno brat https://wiki.osdev.org/X86-64_Instruction_Encoding

in v tej smeri https://gist.github.com/mikesmullin/6259449

če delaš artitmetiko s pointerji je to probably array offset
-> + mul, če je pa samo + je pa struct offset

word = 2 bytes

floati: https://float.exposed/0x0000

ROP samo pomen da si controllov return address, skoču na "gadget" -> snippet z ret-om
--> as mentioned before, stack more bit 16-bit aligned, če nočeš SIGSEGV ob system -> gadget z ret-om

NX samo pomen da sta stack in heap no execute

endbr64 - basically nop, ideja je da cpu checka, da si actually jumpov na ok lokacijo, v praksi se redko uporablja

temu se reče IBT - indirect branch tracking, na nekaterih intel / amd (se sicer imenuje drugače)

SHSTK - shadow stack - ločen stack samo za ret addre, prepreči ret hijacking -> samo Intel!

- usually samo predvidevamo da shadow stack ni *actually* vklopljen (ker moraš met še pravi procesor in bogve kaj), če pa probaš na remotu in actually je, pač ROP ne bo delal

note da ima mips recimo (pa druge arhitekture) precej bl shit protectione

ofc ne `free` ne stack pop ne karkoli actually ne deleta memorija, isto gre za stack variable ki niso initialized, `malloc` ti returna stare podatke..

## libc
- GOT (global offset table) -> libc addressi
    - napolne po actual callu funkcije
        - partial relro: pred .bss
        - no relro: i guess po .bss-ju?
- PLT (procedure linking table) -> actually tja kamor jumpaš da kličeš stvari

- navadno leak libc -> calculate offset -> kej pametnega (system / one_gadget)

- ret2win -> retaš v win funkcijo
- ret2shellcode -> brez NX retaš v kodo
- ret2plt 

- stare verzije: \_\_malloc\_hook in \_\_free\_hook na system za direct rce

- don't know libc version? https://libc.rip/

- libc source: https://elixir.bootlin.com/glibc/glibc-2.39/source/iconv

- if you can run arbitrary asm, you don't even need libc! execve syscall (also, you can do environment = null and args = null)

\_\_ctype\_b\_loc --> glej [stackoverflow ctype-b-loc-what-is-its-purpose](https://stackoverflow.com/questions/37702434/ctype-b-loc-what-is-its-purpose), uporablen je za is\_uppercase, lowercase, alphanumeric etc. na charih
- bitmaske se ločijo med little endian in big endian sistemi
    - (na big endian ali pa če te zanima konkretno): glej ctype.h
        - `(1 << bit) >> 8` na LE (`(bit) < 8 ? ((1 << (bit)) << 8) : ((1 << (bit)) >> 8)`)
        - `1 << bit` na BE
- na navadnem LE so tole maske:
- 0x100 - [isupper](https://en.cppreference.com/c/string/byte/isupper) (uppercase letter)
- 0x200 - [islower](https://en.cppreference.com/c/string/byte/islower) (lowercase letter)
- 0x400 - [isalpha](https://en.cppreference.com/c/string/byte/isalpha) (alphabeteic)
- 0x800 - [isdigit](https://en.cppreference.com/c/string/byte/isdigit) (decimal digit)
- 0x1000 - [isxdigit](https://en.cppreference.com/c/string/byte/isxdigit) (hex digit)
- 0x2000 - [isspace](https://en.cppreference.com/c/string/byte/isspace) (whitespace)
- 0x4000 - [isprint](https://en.cppreference.com/c/string/byte/isprint) (printable, including space)
- 0x8000 - [isgraph](https://en.cppreference.com/c/string/byte/isgraph) (printable, excluding space)
- 0x1 - [isblank](https://en.cppreference.com/c/string/byte/isblank) (space / tab)
- 0x2 - [iscntrl](https://en.cppreference.com/c/string/byte/iscntrl) (control character)
- 0x4 - [ispunct](https://en.cppreference.com/c/string/byte/ispunct) (punctuation)
- 0x8 - [isalnum](https://en.cppreference.com/c/string/byte/isalnum) (alphanumeric)


- direct printf je itak amazing: https://github.com/whatsyourask/basics-of-pwn/blob/main/content/format-string/format-string.md
%number$x
- offset: 
    - Send a recognizable prefix followed by multiple pointer specifiers.
    - Example payload: AAAA %p %p %p %p %p %p %p
    - Check the output for the hexadecimal representation of your prefix (AAAA = 0x41414141).
    - Count which %p prints 0x41414141. That number is your offset.

https://docs.pwntools.com/en/stable/fmtstr.html

https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/return-to-libc-ret2libc

## heap
- glej [ir0nstone](https://ir0nstone.gitbook.io/notes)
- \_\_free\_hook in \_\_malloc\_hook, nastaviš na function in se bo klical ob freeju in mallocu z istimi argi
    -> npr. \_\_free\_hook = system, free(moj\_string\_k\_ma\_bin\_sh\_noter)
- poleg hookov delaš tud ROP prek stacka lahko
- leak libcja prek heapa: unsorted bin v main arena
- also: majhni bini so O(n) checked, unsorted niso!! -> samo ne freejej dvakrat zapored istega
- also z overflowom lahko nastaviš size, overlapping chunki
- dost velk chunk: noter narediš šeen fake chunk, potem daš nek pointer na njega v binih
- tcache je na začetk heap-a (tcache\_perthread\_struct)
    - v dost stari (\<2.42 afaik) se alocira takoj ob prvem mallocu
    - v 2.42 afaik se alocira ob dovolj majhnem mallocu
    - v 2.43 afaik se alocira ob dovolj majhnem free-ju (actually ko ga rabimo)

[shellphish/how2heap](https://github.com/shellphish/how2heap) !!

[https://github.com/DhavalKapil/heap-exploitation](https://github.com/shellphish/how2heap)

## glibc version thingies

- tega mostly ne rabš, razen na hooke pa fastbine, unlikely je glibc cve-je exploitat

---
glibc!
https://sourceware.org/glibc/wiki/Glibc%20Timeline

2.3 | Malloc is rewritten, unsure what was before, but this brings about bins (although without any protections)

2.3.3 | New syscall interface, lots of PLT entries removed, fexecve added

2.16 | support for the x32 ABI on x86-64

2.18
- CVE-2013-2207 Incorrectly granting access to another user's pseudo-terminal
- CVE-2013-0242 Buffer overrun in regexp matcher has been fixed
- CVE-2013-1914 Stack overflow in getaddrinfo with many results

2.19 | CVE-2013-4788 The pointer guard used for pointer mangling was not
  initialized for static applications resulting in the security feature
  being disabled

2.21 | CVE-2015-1472 Under certain conditions wscanf can allocate too little
  memory for the to-be-scanned arguments and overflow the allocated
  buffer
     
2.24
- \_\_malloc\_initialize\_hook removed
- lots of network cves
     
2.25 | Most of glibc can now be built with the stack smashing protector
  enabled.  It is recommended to build glibc with --enable-stack-
  protector=strong.

2.26 | Tcache added

2.29 | Tcache keys, double-free protection; also makes glibc verify the size of the previous block

2.32 | Safe linking - XOR mangling of pointers for singly linked lists (tcache, fastbins)

2.34 | \_\_malloc\_hook, \_\_free\_hook and \_\_realloc\_hook are kill; tcache keys are now random instead of heap pointers

2.38 | CVE-2023-25139: When the printf family of functions is called with a
  format specifier that uses an \<apostrophe> (enable grouping) and a
  minimum width specifier - potential buffer overflow

2.39 | vulnerable na https://ubuntu.com/security/CVE-2024-2961 (iconv)

2.43 | fastbins are kill

---

pdb is the symbols file

## TLS (🦑)
- 🦑 - to je thread local storage (TLS) od main threada, ki je tist goofy RW anonymous page mappan direktno pred libc
- **razen za zelo crazy challenge *ni zelo pomembno za pwn*! rajši prej heap pa vse ostalo** - to je podrobno opisan zaradi avtističnosti in ker sta heap in stack drugje lepše dokumentirana (ir0nstone!)
- pwndbg commanda tls
- veliki malloci, AKA mmap(MAP\_ANONYMOUS), so dani direktno pred ta page
    - če maš tm primitive lahko overridaš canary in thread local stvari
        - tam so juicy libc stvari (main arena, tcache)
        - also, tam so tud function pointerji (tls dtors aka destructors (non-trivial destructorji za thread local C++ classe), ki se kličejo ob exit()u - `__call_tls_dtors` in `dtor_list`, ki je [dtor_list](https://elixir.bootlin.com/glibc/glibc-2.42/source/stdlib/cxa_thread_atexit_impl.c#L82)* (singly linked list))
            - xor-ani so s pointer_guardom in ror-jani, ta guard je direktno po stack canaryjem ([bootlin](https://elixir.bootlin.com/glibc/glibc-2.42/source/sysdeps/x86_64/nptl/tls.h#L42) - tcbhead_t = thread control block head)
            - glej https://github.com/ljagiello/ctf-skills/blob/main/ctf-pwn/advanced-exploits-2.md#tls-destructor-overwrite-for-rce-via-__call_tls_dtors

- layout za x86 (ja, vir je clanker):
```
 high addresses
 ┌────────────────────────────────────────────┐
 │ rest of struct pthread                     │  tid, cancel state, robust list,
 │                                            │  specific[] (pthread_setspecific), ...
 ├────────────────────────────────────────────┤
 │ tcbhead_t (first member of struct pthread) │
 │   +0x00  tcb  (self-pointer)               │ ◄── $fs_base = thread pointer (TP)
 │   +0x08  dtv  ─────────────────────────────┼──► dtv[] (see below)
 │   +0x10  self (THREAD_SELF / pthread_self) │
 │   +0x18  multiple_threads                  │
 │   +0x1c  gscope_flag                       │
 │   +0x20  sysinfo                           │
 │   +0x28  stack_guard   (stack canary)      │  $fs:0x28
 │   +0x30  pointer_guard (PTR_MANGLE key)    │  $fs:0x30
 │   ...    feature_1, shadow-stack, padding  │
 ├────────────────────────────────────────────┤ ◄── $fs:0
 │ module 1: YOUR executable (.tdata + .tbss) │  $fs:-N, fixed offset
 ├────────────────────────────────────────────┤
 │ module 2..k: libc.so.6, libm, other libs   │  libc's errno, tcache ptr,
 │   loaded at startup ("static TLS")         │  thread_arena, ctype/locale pointers, ...
 ├────────────────────────────────────────────┤
 │ static TLS surplus (small spare area)      │  for dlopen'd libs using initial-exec TLS
 └────────────────────────────────────────────┘
 low addresses
```

The dtv (dynamic thread vector) is what __tls_get_addr uses:

- dtv\[-1].counter = number of slots
- dtv\[0].counter  = generation count
- dtv\[1].pointer  → module 1 block (inside the static area)
- dtv\[2].pointer  → module 2 block (e.g. libc)
- ...
- dtv\[n].pointer  → malloc'd block for a dlopen'd lib (allocated lazily on first access)

How code reaches each kind of variable (this is why the offsets differ):
		
| Variable in                      | TLS model       | Access                                                   |
|----------------------------------|-----------------|----------------------------------------------------------|
| Your executable                  | local-exec      | mov fs:-4, $eax, offset known at link time              |
| libc.so (e.g. errno)             | initial-exec    | load the offset from the GOT (@gottpoff), then %fs:(reg) |
| Other shared libs, -fPIC default | general-dynamic | call __tls_get_addr, which goes through the dtv          |
| Library loaded by dlopen         | general-dynamic | same, with a dynamically allocated block                 |

Za npr. `__thread int x;` je `&x` = `$fs_base - neka_konstanta`

### več threadov

`pthread_create` mmapa za vsak thread nov anon RW block (spet, tik pred TLS!):

```
 high ┌──────────────────────────────┐ ◄── mmap end
      │ struct pthread (tcbhead_t    │ ◄── this thread's $fs_base
      │   is its first member)       │
      ├──────────────────────────────┤
      │ static TLS: exe + libc +     │  size = _dl_tls_static_size,
      │ startup libs + surplus       │  fixed for ALL threads
      ├──────────────────────────────┤ ◄── initial stack pointer
      │ thread stack (grows down)    │
      ├──────────────────────────────┤
 low  │ guard page (PROT_NONE)       │
      └──────────────────────────────┘
```

## segmenti ELFa
- readelf -l \<main>
```
  Segment Sections...
   00
   01     .interp
   02     .note.gnu.build-id .interp .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .text .fini
   04     .rodata .eh_frame_hdr .eh_frame .note.gnu.property .note.ABI-tag
   05     .tdata .init_array .fini_array .dynamic .got .got.plt .data .bss
   06     .dynamic
   07     .note.gnu.build-id
   08     .note.gnu.property
   09     .note.ABI-tag
   10     .tdata
   11     .note.gnu.property
   12     .eh_frame_hdr
   13
   14     .tdata .init_array .fini_array .dynamic .got
```
- segment 2 so R headerji in linking metadata
- segment 3 je RX koda
- segment 4 so R podatki
- segment 5 so RW podatki

- .text - koda od programa
    - .plt in .plt.got so stubi za shared librarije (.so-je)
    - .init in .fini so kratke startup in shutdown stvari

- .rodata - raw stringi
- .bss - globalne variable
    - .data so initialized globalne, .bss so zero-initialized (aka ne v fajlu), 
    - .got in .got.plt
    - .init_array in .fini_array so arraji constructor in destructor function pointerjev
    - .dynamic in .tdata so thread-local initial valuji

## random

brunners takeaways
- TLS before libc can be VERY useful in rare cases (tud sam za shranjevat podatke, če veš samo libc address npr.)
- **sometimes calling main again is very useful - first try leak stuff, second try uses the leaks!**

PE so windows portable executabli - na linuxu glej "pev"
-> peres -v program.exe | awk '{print $3}'

https://oneraynyday.github.io/dev/2020/05/03/Analyzing-The-Simplest-C++-Program/

setvbuf modes:
- IOFBF = 0 (full buffer)
- IOLBF = 1 (line buffer)
- IONBF = 2 (no buffer)

setvbuf(stdin, 0, 2, 0) -> set mode to not buffered

open / read / write -> *file descriptor*, 0 STDIN, 1 STDOUT, 2 STDERR
open naredi nov file descriptor, ki je literally samo incremental od 3 naprej (you had one job)

fopen / fread / fwrite -> FILE*

[srop (sigreturn oriented)](https://hacktricks.wiki/en/binary-exploitation/rop-return-oriented-programing/srop-sigreturn-oriented-programming/index.html) in [sfop (segfault oriented)](https://media.defcon.org/DEF%20CON%2034/DEF%20CON%2034%20presentations/DEF%20CON%2034%20presentations/DEF%20CON%2034%20-%20Marcos%20Bajo%2C%20Ritvik%20Goyal%20-%20Crashing%20the%20Party%20Pwning%20Control-Flow%20Integrity%20with%20Segmentation%20Fault-Oriented%20Programming%20-%20Crashingthe.pdf) obstaja apparently

tudi [fsop](https://niftic.ca/posts/fsop/) - file system oriented

hidden permission je setuid (za sudo npr.) ki naredi da se zalaufa kot njegov owner

bpftrace - useful tracing and debugging tool

inotify - trace-anje za file evente

https://youtu.be/3F5icGjDWfg?t=1404 --- LD\_PRELOAD, druge env variable: ephemeral file, preko /proc/self/fd !!!!!

LD\_DEBUG: https://youtu.be/3F5icGjDWfg?t=1597 -- debug the dynamic linker in glibc (since 2.1)

cron je zelo permissive, dovoli \t namest presledka za delimiterje, /etc/cron.d/hax tud dela

you execute a directory when you cd into it! (you need x permissions to cd)
-> in vsi parenti morajo met +x

start qemu virtual network: sudo virsh net-start default

vsi terminali so shite in ti ne dovoljo pastat dolgih stvari

C bitlags: https://github.com/jhynes94/C\_BitPacking

## i386
- glej sys v abi - **argumenti so pushed na stack v reverse orderju - prvi pop je prvi arg itd.**
    - ravno zato ima zelo rad mov \[esp], ... chaine!
- libc želi met v ebx startn address od GOTa da karkoli dela
    - glej: [ttmo-O/x86-manpages#130](https://github.com/david942j/one_gadget/issues/130) oz https://david942j.blogspot.com/2017/02/project-one-gadget-in-glibc.html (Data access method)
        - tldr:

        ```
        $ readelf -d libc.so.6 | grep PLTGOT
        0x00000003 (PLTGOT)                     0x1b2000
        ```
