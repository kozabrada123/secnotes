\>\> https://ir0nstone.gitbook.io/notes <<

https://syscall.sh/ <-- to niti ni tolk pomembn

ldd - view versions of ld and glibc - asterisk, view where they are linked to, not the version!
    - also, to kaže dependencije depencencijev, tko da ne bo nujno required od osnovnega elfa, ampak lahko je 
      dependency od depencija

pwninit - use the correct libc

objdump -d <stuff>

(objdump --disassembler-color=on -M intel -d $@)

also readelf

zalaufi ropr!!! (also, one_gadget)
- GOT address of libc means the start of the RW section -- https://david942j.blogspot.com/2017/02/project-one-gadget-in-glibc.html

-- also also pwntools also zna iskat gadgete - https://docs.pwntools.com/en/stable/rop/rop.html

https://dogbolt.org/ (decomp) in https://godbolt.org/ (comp)

pwn template (tudi iz dockerfila vzame), pwninit ti zalaufa patchelf
- sicer pa dockerfile pač docker exec -it <container> bash in potem docker cp <container>:/path ./path

pwndbg
- x/n -> prints n things -> if you have e.g. x interpreter and that doesn't work (it thinks the value is an addr or sth), * and & work (x &interpreter)
- p(/x)
- tele
- ni (next instruction)
- si (step into)
- finish (go until ret of current function)
- start -> very useful when you have no main
- set follow-fork-mode child / parent --- za forke
- disass <func>
- core <coredump>

- context
- stack
- bins
- vis
- break (info breakpoints / del <num>)
- run (can have redirects (<))

- file
- target remote localhost:1234
- attach <pid>

- info functions
- show npr. architecture, set npr. architecture i386

gef tudi obstaja

in na tej temi tudi vagd

-- random stvar:

    (cat solve/solve.bin; cat) | ./app --- stdin je še vedno odprt

    ulimit -c unlimited
    Run your exploit: ./app < solve/solve.bin
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

man x86-<sth>

checksec !!!

če maš system("/bin/sh") si že zmagal(a)
    -> system(neki) naredi sh -c "neki"

#include <unistd.h>

c headerji so v /usr/include

stack itak veš da gre višji del -> nižjem
stack "frame":

[local vars]
[rbp]
[return address]

wonky ass arhitekture: qemu, gdb-multiarch
- npr. qemu-riscv32-static <myapp> in ga lepo zalaufa
- debug: qemu-riscv32-static -g <port> <myapp> (čaka za gdb connection) -- sicer sta tud -s (odpre gdbserver na 1234) in -S (čaka na connection) (če tadva moraš oba dat)
    - gdb -ex "target remote localhost:<port>" -ex "set architecture riscv:rv32" -ex "file <myapp>"
        - -ex : laufi to komando

- če ne zna najdt librarijev: export QEMU_LD_PREFIX=/usr/aarch64-linux-gnu npr

also, 8 byte stvari so 8 byte aligned v memoriju

ASLR obstaja
--> leakej pointer
- PIE -> ASLRamo še binary; če ne, je na statični poziciji
--> leak libcja prek heapa: unsorted bin v main arena

disable ASLR
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space

enable ASLR
echo 2 | sudo tee /proc/sys/kernel/randomize_va_space

če daš gdb <app> bo vedno mel aslr off
pol pa pač attachaš, sicer pa set disable-randomization off

heap:
---> __free_hook in __malloc_hook, nastaviš na function in se bo klical ob freeju in mallocu z istimi argi
    -> npr. __free_hook = system, free(moj_string_k_ma_bin_sh_noter)
---> poleg hookov delaš tud ROP prek stacka lahko
---> also: majhni bini so O(n) checked, unsorted niso!! -> samo ne freejej dvakrat zapored istega
---> also z overflowom lahko nastaviš size, overlapping chunki
---> dost velk chunk: noter narediš šeen fake chunk, potem daš nek pointer na njega v binih
---> tcache je na začetk heap-a (tcache_perthread_struct)
    -> v dost stari (<2.42 afaik) se alocira takoj ob prvem mallocu
    -> v 2.42 afaik se alocira ob dovolj manjhnem mallocu
    -> v 2.43 afaik se alocira 

shellphish/how2heap !!
https://github.com/DhavalKapil/heap-exploitation

default kaj naredi gcc (a so canariji by default, kdaj, ...) nastavijo distroji; nekateri (khm khm debian) še vedno ne uporablajo canarijev

stack canary:
- na stack je pushan nek random thingy iz kernel-set thread local space-a (1), kernel ga zgenerira, ampak je samo *per process* unique
- 7 random bytov + null da ne moreš leakat
- to se na konc funkcije prever, če ni ok -> bad things
- -fstack-protector-strong, -fno-stack-protector

1 - to je thread local storage, ki je tist goofy RW anonymous page mappan direktno pred libc
-> če maš thread_local variable bodo (usually? citation needed?) direktno pred (0x7c vs 0xa4)
-> veliki malloci, AKA mmap(MAP_ANONYMOUS), so dani direktno pred ta page
    -> če maš tm privitive lahko overridaš canary in thread local stvari
        -> pred tvojimi thread local (ja, isto če maš v velikem mallocu oob) so juicy libc stvari (main arena, tcache)

--> kako beatat? leak / brute force

.text - koda
.data - raw stringi
.bss - globalne variable

brunners takeaway - anon RW before libc can be VERY useful in rare cases
sometimes calling main again is very useful - first try leak stuff, second try uses the leaks!

if you have arbitrary file read --> /proc/self!!!! /proc/self/fd, /proc/self/maps, ...

test eax eax -> ne nastavi registrov, samo zero flag
-> npr. jeq potem, če je eax = 0

xor eax eax cleara rax; vse 32 bitne inštrukcije tudi zero-ajo višje dele 64-bitnih registrov

če delaš artitmetiko s pointerji je to probably array offset
-> + mul, če je pa samo + je pa struct offset, sej nisi stupid

word = 2 bytes

__ctype_b_loc --> glej https://stackoverflow.com/questions/37702434/ctype-b-loc-what-is-its-purpose, uporablen je za is_uppercase, lowercase, alphanumeric etc za chare; 0x2000 je is_blank

ROP samo pomen da si controllov return address, skoču na "gadget" -> snippet z ret-om
--> as mentioned before, stack more bit 16-bit aligned, če nočeš SIGSEGV ob system -> gadget z ret-om

NX samo pomen da sta stack in heap no execute

libc
- GOT (global offset table) -> libc addressi
    - napolne po actual callu funkcije
        - partial relro: pred .bss
        - no relro: i guess po .bss-ju?
- PLT (procedure linking table) -> actually tja kamor jumpaš da kličeš stvari

- navadno leak libc -> calculate offset -> kej pametnega

- ret2win -> retaš v win funkcijo
- ret2shellcode -> brez NX retaš v kodo
- ret2plt 

- stare verzije: __malloc_hook in __free_hook na system za direct rce

direct printf je itak amazing: https://github.com/whatsyourask/basics-of-pwn/blob/main/content/format-string/format-string.md
%number$x

-> offset: 
    Send a recognizable prefix followed by multiple pointer specifiers.
    Example payload: AAAA %p %p %p %p %p %p %p
    Check the output for the hexadecimal representation of your prefix (AAAA = 0x41414141).
    Count which %p prints 0x41414141. That number is your offset.

https://docs.pwntools.com/en/stable/fmtstr.html

https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/return-to-libc-ret2libc

https://www.felixcloutier.com/x86/$instruction

endbr64 - basically nop, ideja je da cpu checka, da si actually jumpov na ok lokacijo, ampak ni nikjer implemented :)))
- IBT - indirect branch tracking, na nekaterih intel / amd

SHSTK - shadow stack - ločen stack samo za ret addre, prepreči ret hijacking -> samo Intel!

če pozab initializat kkšn field: star chunk ponovno mallocan, podatki ostanejo :)))

don't know libc version? https://libc.rip/

libc source: https://elixir.bootlin.com/glibc/glibc-2.39/source/iconv

if you can run arbitrary asm, you don't even need libc! execve syscall (also, you can do environment = null and args = null)

ofc ne free ne stack pop ne karkoli actually ne deleta memorija, malloc ti returna start podatke..

note da ima mips recimo (pa druge arhitekture) precej bl shit protectione

glibc!
https://sourceware.org/glibc/wiki/Glibc%20Timeline
2.3 | Malloc is rewritten, unsure what was before, but this brings about bins (although without any protections)
2.3.3 | New syscall interface, lots of PLT entries removed, fexecve added
2.16 | support for the x32 ABI on x86-64

2.18 | CVE-2013-2207 Incorrectly granting access to another user's pseudo-terminal
     | CVE-2013-0242 Buffer overrun in regexp matcher has been fixed
     | CVE-2013-1914 Stack overflow in getaddrinfo with many results

2.19 | CVE-2013-4788 The pointer guard used for pointer mangling was not
  initialized for static applications resulting in the security feature
  being disabled

2.21 | CVE-2015-1472 Under certain conditions wscanf can allocate too little
  memory for the to-be-scanned arguments and overflow the allocated
  buffer
     
2.24 | __malloc_initialize_hook removed
     | lots of network cves
     
2.25 | Most of glibc can now be built with the stack smashing protector
  enabled.  It is recommended to build glibc with --enable-stack-
  protector=strong.

2.26 | Tcache added
2.29 | Tcache keys, double-free protection; also makes glibc verify the size of the previous block
2.32 | Safe linking - XOR mangling of pointers for singly linked lists (tcache, fastbins)
2.34 | __malloc_hook, __free_hook and __realloc_hook are kill; tcache keys are now random instead of heap pointers

2.38 | CVE-2023-25139: When the printf family of functions is called with a
  format specifier that uses an <apostrophe> (enable grouping) and a
  minimum width specifier - potential buffer overflow

2.39 | vulnerable na https://ubuntu.com/security/CVE-2024-2961 (iconv)
2.43 | fastbins are kill

pdb is the symbols file

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

srop (system rop) in sfop (segfault oriented) obstaja apparently

hidden permission je setuid (za sudo npr.) ki naredi da se zalaufa kot njegov owner

bpftrace - useful tracing and debugging tool

inotify - trace-anje za file evente

https://youtu.be/3F5icGjDWfg?t=1404 --- LD_PRELOAD, druge env variable: ephemeral file, preko /proc/self/fd !!!!!

LD_DEBUG: https://youtu.be/3F5icGjDWfg?t=1597 -- debug the dynamic linker in glibc (since 2.1)

cron je zelo permissive, dovoli \t namest presledka za delimiterje, /etc/cron.d/hax tud dela

you execute a directory when you cd into it! (you need x permissions to cd)
-> in vsi parenti morajo met +x

start qemu virtual network: sudo virsh net-start default

vsi terminali so shite in ti ne dovoljo pastat dolgih stvari

C bitlags: https://github.com/jhynes94/C_BitPacking

i386
- glej sys v - **argumenti so pushed na stack v reverse orderju - prvi pop je prvi arg itd.**
    - ravno zato ima zelo rad mov [esp], ... chaine!
- libc želi met v rbx startn address od GOTa da karkoli dela
    - glej: https://github.com/david942j/one_gadget/issues/130 oz https://david942j.blogspot.com/2017/02/project-one-gadget-in-glibc.html (Data access method)
        - tldr $ readelf -d libc.so.6 | grep PLTGOT
               0x00000003 (PLTGOT)                     0x1b2000
