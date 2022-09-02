# Introduction to Linux
---
## Installation
### Install latest `VMware Workstation` and `Ubuntu`.
### Essential packages in `Ubuntu`:
```shell
# open-vm-tools
sudo apt install open-vm-tools
# vim Environment
sudo apt install vim
# open-ssh
sudo apt install openssh-server
# show net information by 'ifconfig'
sudo apt install net-tools
# Tree folder preview
sudo apt install tree
```
### Install `Xshell` and `Xftp` link to virtual server
### Install `VSCode` and `Remote Development` extensions. Edit `\.ssh\config` file
```shell
Host Ubuntu22
    HostName (Server ip)
    User (Server username)
```
---
## GCC (GNU Compiler Collection)
### Installation: `sudo apt install gcc g++`
### Understanding compilation
![](https://static.packt-cdn.com/products/9781789801491/graphics/C11557_01_01.jpg)

### Useful compiling options
1. Use option `-o` to specify the output file name for the executable
  ```shell
  gcc main.c -o main
  ```

2. `-Wall` enables all the warnings. `-w` no warnings
  ```shell
  gcc -Wall main.c -o main
  ```
3. Compile files in stages
  ```shell
  # preprocessing stage
  gcc -E main.c -o main.i
  # assembly level output
  gcc -S main.c -o main.s
  # only the compiled code (without any linking)
  gcc -C main.c -o main.o
  # output at all the stages of compilation
  gcc -save-temps main.c
  $ ls
  a.out  main.c  main.i  main.o  main.s
  ```
4. Link with shared libraries using `-l` option
  ```shell
  # links the code main.c with the shared library libCPPfile.so
  gcc -Wall main.c -o main -l CPPfile
  ```
5. Define compile time macros in code using `-D`.
```C
#include<stdio.h>
int main(void){
    #ifdef MY_MACRO
      printf("\n Macro defined \n");
    #endif
    printf("Hello");
    return 0;
}
```
```shell
gcc -D MY_MACRO main.c -o main
./main
#output
Macro defined
Hello
```
---
## Shared and Static Libraries
### Static Libraries
- `libxxx.a` files in Linux, `libxxx.lib` files in Windows. It is directly linked into the program at compile time.
- Making static lib: create compiled code using `-C`, after that archive them
  ```shell
  # generate add.o, sub.o
  gcc -c add.c sub.c
  # archive .o files to a static lib called calc.a
  ar rcs libcalc.a add.o sub.o
  ```
- Using static lib
  - Must have **lib file** and corresponding **header file**
```shell
gcc main.c -o app -I ./include -l calc -L ./lib
```
  - `-I ./include` indicate the header file directory
  - `-l calc` indicate the static lib file
  - `-L ./lib` indicate the static lib file directory

### Shared Libraries
- `libxxx.so` (executable) files in Linux, `libxxx.dll` files in Windows. It is referenced by programs using it at run-time.
- Making shared lib: create **position independent** compiled code using `-C` and `-fPIC`
  ```shell
  # generate .o file
  gcc -c -fpic add.c sub.c
  # archive .o files to a shared lib called calc.so
  gcc -shared add.o sub.o -o libcalc.so
  ```
- Using shared lib
  - Must have **lib file** and corresponding **header file**
  ```shell
  gcc main.c -o app -I ./include -l calc -L ./lib
  ```
  - Must tell the operating system where it can locate it at runtime.
  ```shell
  # (optional)Find where the library is placed if you don't know it
  sudo find / -name the_name_of_the_file.so
  # edit ~/.bashrc
  vim ~/.bashrc
  # add a new line in ~/.bashrc, save and exit
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/u/Desktop/Linux/calc/lib (your lib location)
  # validate the environment variables
  source ~/.bashrc
  # try the application with shared lib
  ./app
  ```

### Advantages and Disadvantages
- **Static libraries** increase the overall **size** of the binary, but it means that you don't need to carry along a copy of the library that is being used. As the code is connected at compile time there are not any additional run-time loading costs.
- **Shared libraries** reduce the amount of code that is duplicated in each program that makes use of the library. However a small additional cost for the execution of the functions as well as a run-time loading cost as all the symbols in the library need to be connected to the things they use. Additionally, shared libraries can be loaded into an application at run-time, which is the general mechanism for implementing binary plug-in systems.
---
## Makefile
### Description
- `Make` is Unix utility that is designed to start execution of a `makefile` which defines set of tasks to be executed. It is a simple way to organize (auto) code compilation.
- Installation : `sudo apt install make`
- File name must be `Makefile` or `makefile`
- A makefile consists one or several rules
```shell
# one rule: the recipe(shell command) uses prerequisites to make a target.
target: prerequisites
<TAB> recipe
```
```shell
# several rules. only the first target in the makefile is the default target
final_target: sub_target final_target.c
        Recipe_to_create_final_target
sub_target: sub_target.c
        Recipe_to_create_sub_target
```

### Write a Makefile
- Suppose we have these source code files
```shell
ubuntu:~/Desktop/Linux$ ls
add.c div.c sub.c mult.c head.h main.c
```
- compile each source file seperately + linking
```shell
app:add.o div.o multi.o sub.o main.o
        gcc add.o div.o multi.o sub.o main.o -o app
add.o:add.c
        gcc -c add.c -o add.o
div.o:div.c
        gcc -c div.c -o div.o        
multi.o:multi.c
        gcc -c multi.c -o multi.o        
sub.o:sub.c
        gcc -c sub.c -o sub.o        
main.o:main.c
        gcc -c main.c -o main.o
```
- customized `variable`=`value`, use `$` get variable
`$@`: target
`$<`: the first dependency
`$^`: all dependencies
`%.o:%.c`: `%` is a wildcard representing a string
```shell
src=add.o div.o multi.o sub.o main.o
target=app
$(target):$(src)
        $(CC) $^ -o $@
%.o:%.c
        $(CC) -c $< -o $@
```
- `$(wildcard pattern) `
This is replaced by a space-separated list of names of existing files that match one of the given file name patterns.
- `$(patsubst <pattern>,<replacement>,<text>)`
Finds whitespace-separated words in text that match pattern and replaces them with replacement.
- `$(notdir names…)`
Extracts all but the directory-part of each file name in names. If the file name contains no slash, it is left unchanged. Otherwise, everything through the last slash is removed from it.
```shell
# self defined vars for file directory
DIR_INC = ./include
DIR_SRC = ./src
DIR_OBJ = ./obj
DIR_BIN = ./bin
# get all .c files
src=$(wildcard $(DIR_SRC)/*.c)
# replace .c with .o files
obj=$(patsubst %.c,$(DIR_OBJ)/%.o,$(notdir $(src)))
target=app
bin_target=$(DIR_BIN)/$(target)
CC=gcc
CFLAGS= -g -Wall -I $(DIR_INC)
# rules
$(bin_target):$(obj)
        $(CC) $^ -o $@
$(DIR_OBJ)/%.o:$(DIR_SRC)/%.c
        $(CC) $(CFLAGS) -c $< -o $@
# .PHONY used where we define all the targets that are not files.
.PHONY:clean
clean:
        find $(DIR_OBJ) -name *.o -exec rm -rf {}
```
---
## GDB: The GNU Project Debugger
### Preparation
- `gcc -g -Wall program.c -o program`
`-g` generate debugging information, `-Wall` turn on all warning
- `gdb program` open gdb on your program
- `quit/q` quit gdb process
- `help` open gdb help doc

### Commands
- `set args 10 20` and `show args`
- show program code from start `list/l`
show program code from specific row `list/l rowNumber`
show program code from specific function `list/l funcName`
- show source file code associated to current programs
`list/l fileName:rowNumber`
`list/l fileName:funcName`

### Breakpoints
- show bp `i/info b/break`
- set normal bp
`b/break rowNumber`
`b/break funcName`
`b/break fileName:rowNumber`
`b/break fileName:funcName`
- set conditional bp `b/break rowNumber if i==5` usually used in if condition
- delete bp `d/del/delete bpNumber`
- disable bp `dis/disable bpNumber`
- enable bp `ena/enable bpNumber`

### Debug Commands
- `start` Sets a temporary breakpoint on main() and starts executing a program under GDB. Stop at first line.
`run` Starts executing a new instance of a program under GDB. Only stop at breakpoint.
- `c/continue` Executing until next breakpoint
- `p/print varName` Print variable names
`ptype varName` Print variable types
- `finish` jump out of functions
`until` jump out of loop
---
## File I/O
### C Library functions and Linux functions
- C Library functions start with `f`, e.g. `fopen` `fclose` `fflush` `fread` `fwrite`
- Linux functions are lower level without `f`: `open` `close` `flush` `read` `write`
- When C library functions are called, they simply call Linux functions in the background.
- Manual guide `man man`
```shell
1   Executable programs or shell commands
2   System calls (functions provided by the kernel)
3   Library calls (functions within program libraries)
```
Use `man 2 API` find Linux functions guide
Use `man 3 API` find C library functions guide

### Virtual Address Space
In Linux each process has its [virtual address space](https://maodanp.github.io/2019/06/02/linux-virtual-space/) (e.g. 4 GB in case of 32 bit system, wherein 3GB is reserved for process and 1 GB for kernel).

### File Descriptor
- File descriptor is an integer number that uniquely represents an opened file for the process.
- Each process has a file descriptor table which is tracked by kernel.
The table is an array which size is 1024.
Whenever you execute a program/command at the terminal, `0->STDIN` `1->STDOUT` `2->STDERR`  are always open.
When a new file is opened, the **smallest** FD is associated with it.

### Linux I/O
#### open & close
- `int open(const char *pathname, int flags);`
- `int open(const char *pathname, int flags, mode_t mode);` create a file
- `int close(int fd);`
  - `flags` must include one of the following access modes: `O_RDONLY`, `O_WRONLY`, or `O_RDWR`.
  - `flags` can have file creation flags and file status flags: `O_CLOEXEC`, `O_CREAT`, - `O_DIRECTORY`, `O_EXCL`, `O_NOCTTY`, `O_NOFOLLOW`, `O_TMPFILE`, and `O_TRUNC`.
  - `flags` access modes and other flags connected by **bitwise-or** `|`
  - `mode` specifies the file mode bits to be applied when a new file is created. e.g `0775`
  - The file descriptor returned by a successful call will be the lowest-numbered file descriptor not currently open for the process. If failed return -1 and set `errno`.

#### read & write
- `ssize_t read(int fd, void *buf, size_t count);`
  - `read()` attempts to read up to `count` bytes from file descriptor `fd` into the buffer starting at `buf`.
  - On success, the number of bytes read is returned (`0` indicates end of file). On error, `-1` is returned and `errno` is set.
- `ssize_t write(int fd, const void *buf, size_t count);`
  - `write()` writes up to `count` bytes from the buffer starting at `buf` to the file referred to by the file descriptor `fd`.
  - On success, the number of bytes written is returned. On error, `-1` is returned and `errno` is set

```C
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main()
{
    // 1.open english.txt
    int srcfd = open("english.txt", O_RDONLY);
    if(srcfd == -1) {
        perror("open");
        return -1;
    }

    // 2.Create a new file
    int destfd = open("cpy.txt", O_WRONLY | O_CREAT, 0664);
    if(destfd == -1) {
        perror("open");
        return -1;
    }

    // 3.read and write until end
    char buf[1024] = {0};
    int len = 0;
    while((len = read(srcfd, buf, sizeof(buf))) > 0) {
        write(destfd, buf, len);
    }

    // 4.close files
    close(destfd);
    close(srcfd);

    return 0;
}
```

#### lseek
- `off_t lseek(int fd, off_t offset, int whence);`
  - `lseek()` repositions the file offset of the open `fd` to the argument `offset` according to the directive `whence`:
  `SEEK_SET`: The file offset is set to offset bytes.
  `SEEK_CUR`: The file offset is set to its current location + offset bytes.
  `SEEK_END`: The file offset is set to the size of the file + offset bytes.
  - On success returns the resulting offset location as measured in bytes from the beginning of the file. If error return `-1` and set errno.
- Using scenarios
  - `lseek(fd, 0, SEEK_SET);` move to the beginning of the file.
  - `lseek(fd, 0, SEEK_CUR);` get the pointer current location.
  - `lseek(fd, 0, SEEK_END);` get the length of the file.
  - `lseek(fd, 100, SEEK_END)` extend current file by 100 bytes.
  If data is later written at this point, subsequent reads of the data in the gap return null bytes ('\0') until data is actually written into the gap. Need to write sth to validate the extension.

#### stat & lstat
- `int stat(const char *pathname, struct stat *statbuf);`
- `int lstat(const char *pathname, struct stat *statbuf);`
- These functions retrieve information about the file pointed to by `pathname`, in the buffer pointed to by `statbuf`.
- Returned value: On success return `0`; If error return -1 and set `errno`.
- `stat structure`
```C
struct stat {
               dev_t     st_dev;         /* ID of device containing file */
               ino_t     st_ino;         /* Inode number */
               mode_t    st_mode;        /* File type and mode */
               nlink_t   st_nlink;       /* Number of hard links */
               uid_t     st_uid;         /* User ID of owner */
               gid_t     st_gid;         /* Group ID of owner */
               dev_t     st_rdev;        /* Device ID (if special file) */
               off_t     st_size;        /* Total size, in bytes */
               blksize_t st_blksize;     /* Block size for filesystem I/O */
               blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

               struct timespec st_atim;  /* Time of last access */
               struct timespec st_mtim;  /* Time of last modification */
               struct timespec st_ctim;  /* Time of last status change */
};
```
- Linux command `stat` + file name. Shows file status.

#### Use `stat` implementing `ls -l`
- implementing `ls -l`
show result: `-rw-rw-r-- 1 user user 12 Dec 3 15:48 a.txt`

```C
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <pwd.h>        // for getpwuid()
#include <grp.h>        // for getgrgid()
#include <time.h>       // for ctime()
#include <string.h>     // for strncpy(), strlen()

int main(int argc, char * argv[]){
  //check arguments validation
    if(argc < 2) {
        printf("%s filename\n", argv[0]);
        return -1;
      }
  // get file info by `stat`
    struct stat st;
    int ret = stat(argv[1], &st);
    if(ret == -1) {
        perror("stat");
        return -1;
    }
  //get file mode and permission
    char perms[11] = {0};
    switch(st.st_mode & S_IFMT) {
        case S_IFLNK:
            perms[0] = 'l';
            break;
        case S_IFDIR:
            perms[0] = 'd';
            break;
        case S_IFREG:
            perms[0] = '-';
            break;
        case S_IFBLK:
            perms[0] = 'b';
            break;
        case S_IFCHR:
            perms[0] = 'c';
            break;
        case S_IFSOCK:
            perms[0] = 's';
            break;
        case S_IFIFO:
            perms[0] = 'p';
            break;
        default:
            perms[0] = '?';
            break;
    }
    perms[1] = (st.st_mode & S_IRUSR) ? 'r' : '-';
    perms[2] = (st.st_mode & S_IWUSR) ? 'w' : '-';
    perms[3] = (st.st_mode & S_IXUSR) ? 'x' : '-';
    perms[4] = (st.st_mode & S_IRGRP) ? 'r' : '-';
    perms[5] = (st.st_mode & S_IWGRP) ? 'w' : '-';
    perms[6] = (st.st_mode & S_IXGRP) ? 'x' : '-';
    perms[7] = (st.st_mode & S_IROTH) ? 'r' : '-';
    perms[8] = (st.st_mode & S_IWOTH) ? 'w' : '-';
    perms[9] = (st.st_mode & S_IXOTH) ? 'x' : '-';

    int linkNum = st.st_nlink;

    char* fileUser = getpwuid(st.st_uid)->pw_name;

    char* fileGrp = getgrgid(st.st_gid)->gr_name;

    long int fileSize = st.st_size;

    char* time = ctime(&st.st_mtime);
    char mtime[512] = {0};
    strncpy(mtime, time, strlen(time) - 1);

    char buf[1024];
    sprintf(buf, "%s %d %s %s %ld %s %s", perms, linkNum, fileUser, fileGrp, fileSize, mtime, argv[1]);

    printf("%s\n", buf);

    return 0;
}
```

### File edit functions
#### access
```C
/*
    #include <unistd.h>
    int access(const char *pathname, int mode);
        checks whether the calling process can access the file pathname.
            - mode:
                R_OK: read permission
                W_OK: write permission
                X_OK: execute permission
                F_OK: tests for the existence of the file
        return value : on success 0, fail -1
*/
#include <unistd.h>
#include <stdio.h>

int main()
{
    int ret = access("a.txt", F_OK);
    if(ret == -1) {
        perror("access");
    }
    printf("File exists\n");

    return 0;
}
```

#### chmode
```C
/*
    #include <sys/stat.h>
    int chmod(const char *pathname, mode_t mode);
        change permissions of a file
            - mode: The new file mode, octal number
        return value : on success 0, fail -1
*/
#include <sys/stat.h>
#include <stdio.h>

int main()
{
    int ret = chmod("a.txt", 0777);
    if(ret == -1) {
        perror("chmod");
        return -1;
    }

    return 0;
}
```

#### truncate
```C
/*
    #include <unistd.h>
    #include <sys/types.h>
    int truncate(const char *path, off_t length);
        truncate a file to a specified length
        return value : on success 0, fail -1
*/

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>

int main()
{
    int ret = truncate("b.txt", 1000);
    if(ret == -1) {
        perror("truncate");
        return -1;
    }

    return 0;
}
```

### Directory functions
#### mkdir & rename
```C
/*
    #include <sys/stat.h>
    #include <sys/types.h>
    int mkdir(const char *pathname, mode_t mode);
        create a directory
    int rename(const char *oldpath, const char *newpath);
        rename a directory
        return value : on success 0, fail -1
*/
#include <sys/stat.h>
#include <sys/types.h>
#include <stdio.h>

int main(){
    int ret = mkdir("lesson01", 0777);
    if(ret == -1){
        perror("mkdir");
        return -1;
    }
    int ret2 = rename("lesson01", "lesson02");
    if(ret2 == -1){
        perror("rename");
        return -1;
    }
    return 0;
}
```

#### chdir & getcwd
```C
/*
    #include <unistd.h>
    int chdir(const char *path);
        changes the current working directory of the calling process to path
        return value : on success 0, fail -1
    char *getcwd(char *buf, size_t size);
        copies an absolute pathname of the current working directory to the array pointed to by buf, which is of length size.
        return value : On success, return a pointer to a string containing the pathname of the current working directory. On failure, return NULL.
*/
#include <unistd.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>

int main(){
    //get current working dir
    char buff[128];
    getcwd(buf, sizeof(buf));
    printf("current working dir is: %s", buf);

    //change current working dir
    int ret = chdir("/home/user/Linux/Lesson01");
    if(ret == -1){
        perror("chdir");
        return -1;
    }

    //try create a file in the new dir
    int fd = open("chdir.txt", O_CREAT | O_RDWR, 0664);
    if(fd == -1) {
        perror("open");
        return -1;
    }
    close(fd);

    return 0;
}
```
#### opendir & readdir & closedir
```C
/*
    #include <sys/types.h>
    #include <dirent.h>
    DIR *opendir(const char *name);
        return value: directory stream. On fail return NULL
    struct dirent *readdir(DIR *dirp);
        return value: struct dirent is file info. On fail return NULL
    int closedir(DIR *dirp);
*/
struct dirent {
    ino_t          d_ino;       /* Inode number */
    off_t          d_off;       /* Not an offset */
    unsigned short d_reclen;    /* Length of this record */
    unsigned char  d_type;      /* Type of file */
    char           d_name[256]; /* Null-terminated filename */
};
```
- `d_type`
    `DT_BLK` This is a block device.  
    `DT_CHR` This is a character device.
    `DT_DIR` This is a directory.
    `DT_FIFO` This is a named pipe (FIFO).
    `DT_LNK` This is a symbolic link.
    `DT_REG` This is a regular file.
    `DT_SOCK` This is a UNIX domain socket.
    `DT_UNKNOWN` The file type could not be determined.

```C
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int getFileNum(const char * path){
    DIR * dir = opendir(path);
    if(dir == NULL){
        perror("opendir");
        exit(0);
    }
    struct dirent *d;
    int total = 0;
    while(d = readdir(dir) != NULL){
        char * name = d->d_name;
        if(strcmp(name, ".") == 0 || strcmp(name, "..") == 0) continue;
        if(d->d_type == DT_DIR){
            total += getFileNum(path/name);
        }else if(d->d_type == DT_REG){
            total++;
        }
    }
    closedir(dir);
    return total;
}

int main(int argc, char * argv[]){
    if(argc < 2) {
        printf("%s path\n", argv[0]);
        return -1;
    }
    int num = getFileNum(argv[1]);
    printf("File number：%d\n", num);

    return 0;
}
```
### FD manipulation
#### dup & dup2
```C
/*
      #include <unistd.h>
      int dup(int oldfd);
      int dup2(int oldfd, int newfd);
          duplicate a file descriptor refer to the same file
          dup returns the lowest-numbered unused fd
          dup2 returns the number specified in newfd
*/

#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
int main()
{
    int fd = open("a.txt", O_RDWR | O_CREAT, 0664);
    int fd1 = dup(fd);
    if(fd1 == -1) {
        perror("dup");
        return -1;
    }
    printf("fd : %d , fd1 : %d\n", fd, fd1);
    close(fd);

    char * str = "hello,world";
    int ret = write(fd1, str, strlen(str));
    if(ret == -1) {
        perror("write");
        return -1;
    }
    close(fd1);

    return 0;
}
```

#### fcntl
```C
/*
    #include <unistd.h>
    #include <fcntl.h>
    int fcntl(int fd, int cmd, ...);
    - cmd: how to operate fd
            - F_DUPFD : Duplicate the fd using the lowest-numbered available
            - F_GETFL : Return the file access mode and the file status flags
            - F_SETFL : Set the file status flags to the value specified by arg.
                Can only change O_APPEND, O_ASYNC, O_DIRECT, O_NOATIME, and O_NONBLOCK flags.
                Cannot change O_RDONLY, O_WRONLY, O_RDWR, O_CREAT, O_EXCL, O_NOCTTY, O_TRUNC
*/
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>

int main(){
    int fd = open("1.txt", O_RDWR);
    if(fd == -1) {
        perror("open");
        return -1;
    }

    // get flags
    int flag = fcntl(fd, F_GETFL);
    if(flag == -1) {
        perror("fcntl");
        return -1;
    }
    flag |= O_APPEND;

    //change flags
    int ret = fcntl(fd, F_SETFL, flag);
    if(ret == -1) {
        perror("fcntl");
        return -1;
    }

    char * str = "Hey";
    write(fd, str, strlen(str));
    close(fd);

    return 0;
}
```
