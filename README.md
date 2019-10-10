# Overview

For this discussion, you will be adding the process status (`ps`) system call to
xv6. The `ps` command in linux like systems is used to print details about
running processes. We will be adding a system call which will do just that,
print the **name** and the **process id**.

# Getting Started

Start docker or the VM, and navigate to the xv6 source directory that you cloned
in a previous discussion homework.

```console
docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined --privileged -it -v [/absolute/path/to/here]:/mnt/files mcorner/os377 bash
```

## Note

If you wish, you can start with a fresh installation of xv6 for this discussion.
Use the following commands, don't forget to add the `-nographic` option to
`QEMUOPTS` before building the source.

```console
>> git clone https://github.com/mit-pdos/xv6-public.git
>> cd xv6-public
```

# Adding a System Call

Adding a system call to xv6 requires changing multiple files. Here is a list of
the files you'll need to modify:

    syscall.h
    syscall.c
    sysproc.c
    usys.S
    user.h
    defs.h
    proc.c

## Registering the call with xv6

Start by adding a system call number to `syscall.h`. Take a look at how other
system calls are numbered, we will be using `SYS_ps` as the identifier for our
system call.

Next, we will add an assembly definition for this call. xv6 makes it easier and
we do not have to write any assembly code, save one line. Open `usys.S`, and add
`SYSCALL(ps)` to the very end of this file.

We will register this call to the xv6 system call table. To do this, open
`syscall.c`, add `extern int sys_ps(void);` after other system call
declarations, and `[SYS_ps] sys_ps,` to the `syscalls` array.

We need to make this system call do some work. Add the following to `sysproc.c`

```C
int sys_ps(void) {
  return ps();
}
```

The `ps` function will do the grunt work for us. The rest of the steps add `ps`
to the appropriate files. Note that up to this point we have registered the call
in xv6, and we simply need to fill in the details before we get a working system
call!

## Making the call work

Let's add the declaration of the `ps` function to `defs.h`. Open `defs.h`, and
look for the comment `\\ proc.c`. On my system, it is close to line number 105.
Add `int ps(void);` to the set of functions that can work on processes.

Also add the declaration above to `user.h`. It is interesting to note that only
functions in `user.h` are available to use as system calls to a user program.

We are ready to add the definition for this function inside `proc.c`. If you
recall from earlier discussions, only `proc.c` has the ability to read/modify
the process table. Add the following function declaration to the very end of the
file, hold onto writing the complete function yet.

```C
int ps(void) {
  cprintf("hi from ps\n");
  return 0;
}
```

We will come back to it later, when we have the whole framework up and running.

## Adding a shell program

To make sure everything works, we will write a program that calls this system
call, and run it inside the shell.

Add a file `ps.c` with the following contents to the xv6 source directory.

```C
#include "types.h"
#include "user.h"

int main(int argc, char* argv[]) {
  ps();
  exit();
}
```

Modify UPROGS inside the Makefile, to add ps as a program.

## Testing

We are done with adding the system call, and a program, time to test it! Run the
following commands in the xv6 src directory.

```
make
make qemu
```

In xv6 shell, run ps

```
$ ps
hi from ps
```

## Finishing up

Let's go back to the `ps` function inside `proc.c`. Your task is to add code
that iterates over the process table (`ptable` in code). Look at how it is used
inside `kill()` in `proc.c` for example. You can essentially copy and paste the
implementation of `kill()` and modify to print out information about a process.
You can find the details of the `struct proc` structure inside `proc.h`. Make
sure to acquire and release locks appropriately. You have to print pid and name
of all processes except ones that are `UNUSED`. A sample execution is provided.

```
$ ps
1 init
2 sh
3 ps
```

### Note: Our implementation has 15 lines of code including empty lines. If you have a lot of code, you are probably doing it wrong.

# Submission

A zip containing all the files that were modified, except `ps.c` and `Makefile`.
A screenshot of xv6 shell running `ps`.
