
1、fork()
int fork()用于在当前进程的基础上创建一个新的子进程。它会复制当前进程的状态和代码，并为子进程分配一个新的进程ID（PID）。子进程是父进程的副本，它从fork()调用的位置开始执行。
在调用fork()后，父进程和子进程都会继续执行代码，但它们会有不同的返回值。在父进程中，fork()会返回子进程的PID，而在子进程中，fork()会返回0。这样，通过返回值的不同，父进程和子进程可以根据需要执行不同的操作。
下面是一个示例代码片段，展示了如何使用fork()创建一个子进程：
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        // 创建子进程失败
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // 子进程代码
        printf("This is the child process.\n");
        // 子进程执行其他操作...
    } else {
        // 父进程代码
        printf("This is the parent process. Child PID: %d\n", pid);
        // 父进程执行其他操作...
    }

    return 0;
}
需要注意的是，fork()系统调用的返回值可能有以下几种情况：
如果返回负值，表示创建子进程失败。
如果返回0，表示当前进程是子进程。
如果返回正值，表示当前进程是父进程，返回的值是子进程的PID。

2、exit
int exit(int status)用于终止当前进程的执行并将状态报告给wait()函数。它没有返回值。
当调用exit()时，当前进程会立即停止执行，并将状态信息传递给父进程或等待该进程终止的进程。该状态信息可以通过wait()系统调用中的参数来获取。
status参数是一个整数，用于表示进程的退出状态。通常情况下，0表示进程成功终止，非零值则表示进程异常终止或返回了某个错误码。
下面是一个示例代码片段，展示了如何使用exit()系统调用来终止当前进程的执行：
#include <stdio.h>
#include <stdlib.h>

int main() {
    // 执行一些操作...

    // 终止当前进程并返回状态码
    exit(0);

    // 下面的代码不会被执行
    printf("This line will not be executed.\n");

    return 0;
}
3、wait
int wait(int *status)用于等待一个子进程的退出，并将子进程的退出状态存储在status指针指向的位置。它返回子进程的PID。
当父进程调用wait()时，如果当前没有子进程终止，父进程将会被阻塞，直到有子进程终止。一旦有子进程终止，父进程会收到其终止的相关信息，并将其退出状态存储在提供的status指针指向的位置。
status参数是一个指向整型的指针，用于存储子进程的退出状态。通过检查status指针指向的位置，可以确定子进程是正常终止还是异常终止，并获取相应的退出状态码。
下面是一个示例代码片段，展示了如何使用wait()系统调用来等待子进程的终止，并获取其退出状态： 
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // 子进程代码
        printf("This is the child process.\n");
        sleep(2); // 子进程休眠2秒后退出
        return 42;
    } else {
        // 父进程代码
        printf("This is the parent process. Child PID: %d\n", pid);

        int childStatus;
        pid_t terminatedChildPid = wait(&childStatus);
        if (terminatedChildPid == -1) {
            perror("wait failed");
            return 1;
        }

        if (WIFEXITED(childStatus)) {
            printf("Child process %d exited with status: %d\n", terminatedChildPid, WEXITSTATUS(childStatus));
        } else {
            printf("Child process %d terminated abnormally.\n", terminatedChildPid);
        }
    }

    return 0;
}
下面是代码的运行过程和输出结果的示例：

添加图片注释，不超过 140 字（可选）
在上面的示例中，父进程调用wait()等待子进程的终止，并通过检查WIFEXITED宏来确定子进程是否正常终止。如果子进程正常终止，父进程输出子进程的PID和退出状态。如果子进程异常终止，父进程会收到相应的信号并输出相应的信息。
需要注意的是，wait()只会等待直接子进程的终止，如果有多个子进程，需要逐个调用wait()来等待它们的终止。此外，如果不关心子进程的退出状态，可以将status参数设置为NULL，这样wait()将不会返回退出状态。
4、fork()、exit()和wait()实例
 结合xv6 books书上的实例：
#include <stdio.h>
#include <unistd.h>
int main() {
    int pid = fork();
    if(pid > 0) {
        printf("parent: child=%d\n", pid);
        pid = wait((int *) 0);
        printf("child %d is done\n", pid);
    } else if(pid == 0) {
        printf("child: exiting\n");
        exit(0);
    } else {
        printf("fork error\n");
    }
    return 0;
}
我简单对这段代码分析一下：
这段代码是一个使用`fork()`、`wait()`和`exit()`系统调用的示例，用于创建一个子进程并等待子进程执行完成。
代码的逻辑如下：
1. 在`main()`函数中，调用`fork()`创建一个新的子进程。父进程中的`pid`变量将会保存子进程的PID。
2. 如果`fork()`返回的值大于0，则表示当前进程是父进程。在父进程中，打印出子进程的PID，并调用`wait()`函数等待子进程的终止。`wait()`函数会阻塞父进程，直到子进程终止。
3. 在子进程中，`fork()`的返回值为0。在子进程中，打印一条退出信息，并调用`exit()`函数终止子进程的执行。
4. 如果`fork()`返回的值小于0，则表示创建子进程失败，打印出错误信息。
下面是代码的运行过程和输出结果的示例：

添加图片注释，不超过 140 字（可选）

在上面的示例中：首先由父进程创建了一个子进程，子进程打印了一条退出信息后就终止了。父进程在调用`wait()`函数时阻塞，等待子进程的终止。一旦子进程终止，父进程恢复执行，并打印出子进程的PID和完成信息。

5、kill
`int kill(int pid, int sig)` ，用于向指定PID的进程发送信号以终止它。它返回0表示成功，返回-1表示出现错误。
`pid`参数是要终止的进程的PID。`sig`参数是要发送的信号编号。通常，使用信号编号为`SIGTERM`（15）来请求进程正常终止。其他常用的信号包括`SIGKILL`（9），它会强制终止进程。

下面是一个示例代码片段，展示了如何使用`kill()`系统调用来终止指定PID的进程：
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

int main() {
    pid_t pid = 18888; // 要终止的进程的PID

    int result = kill(pid, SIGTERM);
    if (result == 0) {
        printf("Process with PID %d terminated successfully.\n", pid);
    } else {
        perror("kill failed");
        return 1;
    }

    return 0;
}
在上面的示例中，调用`kill(pid, SIGTERM)`将向PID为18888的进程发送`SIGTERM`信号，请求它正常终止。如果`kill()`调用返回0，则表示终止请求成功。否则，如果返回-1，则出现了错误，可以通过`perror()`函数打印出错误信息。
注意，使用`SIGKILL`信号强制终止进程可能会导致进程无法进行清理操作或释放资源，因此应该谨慎使用。通常情况下，应该首先尝试使用`SIGTERM`信号请求进程正常终止，然后再考虑使用`SIGKILL`信号。

6、getpid
`int getpid()` ，用于返回当前进程的PID（进程标识符）。
下面是一个示例代码片段，展示了如何使用`getpid()`函数来获取当前进程的PID：
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = getpid();
    printf("Current process PID: %d\n", pid);

    return 0;
}
在上面的示例中，调用`getpid()`函数将返回当前进程的PID，并将其存储在`pid`变量中。然后可以使用`printf()`函数将PID打印出来。
注意，`getpid()`函数是一个系统调用，它可以在不同的操作系统上提供不同的实现。在Linux和UNIX系统中，它通常会返回一个整数值作为进程的唯一标识符。每个进程都有一个独特的PID，用于在系统中识别和管理进程。

7、sleep
`int sleep(int n)` ，用于使当前进程暂停执行，暂停的时间长度由参数 `n` 指定，单位是秒。
当调用 `sleep(n)` 时，当前进程将会被挂起，暂停执行约 `n` 秒的时间。在暂停期间，进程不会消耗 CPU 资源，并且不会进行任何活动。一旦暂停时间结束，进程将恢复执行。
注意的是，`sleep()` 函数的实际暂停时间可能会稍微超过指定的秒数。这是因为操作系统的调度和其他系统活动可能会导致实际的睡眠时间略微延长。
下面是一个示例代码片段，展示了如何使用 `sleep()` 函数来使当前进程暂停执行指定的时间：
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("before sleep\n");

    sleep(5); // 暂停执行5秒钟

    printf("delay 5s\n");

    return 0;
}
在上面的示例中，调用 `sleep(5)` 会使当前进程暂停执行5秒钟。在暂停期间，"before sleep"会被打印出来，然后进程会暂停5秒钟。最后，暂停结束后，"delay 5s"会被打印出来。

8、exec
`int exec(char *file, char *argv[])` 函数系列是一组系统调用函数，用于加载并执行一个新的程序。具体而言，`exec()` 函数会替换当前进程的映像，加载并执行指定的文件，并使用给定的参数传递给新程序。这些函数不会在成功时返回，只有在出错时才会返回。
在C语言中，`exec()` 函数系列通常是通过调用其中的一个函数来实现的，如 `execl()`、`execv()`、`execle()`、`execve()` 等。这些函数的区别在于参数的传递方式和文件搜索的方式。

下面是一个示例代码片段，展示了如何使用 `exec()` 函数加载一个文件并使用给定的参数执行它：
#include <stdio.h>
#include <unistd.h>

int main() {
    // 以 execl() 函数为例
    // 执行 ls 命令
    execl("/bin/ls", "ls", "-l", NULL);

    // 如果 exec() 调用成功，以下代码不会执行
    // 只有在出错时才会执行到这里
    perror("exec failed");

    return 1;
}
在上面的示例中，调用 `execl("/bin/ls", "ls", "-l", NULL)` 加载并执行 `/bin/ls` 程序，并传递 `-l` 参数给它。如果 `execl()` 调用成功，当前进程的映像将被替换为 `/bin/ls` 程序的映像，而且以下代码不会执行。只有当 `execl()` 调用出错时，才会执行 `perror()` 函数打印出错误信息。
`exec()` 函数系列是一组系统调用函数，用于加载并执行一个新的程序。这些函数会替换当前进程的映像，加载并执行指定的文件，并使用给定的参数传递给新程序。这些函数不会在成功时返回，只有在出错时才会返回。
以下是一些常见的 `exec()` 函数及其主要特点：
1. `int execl(const char *path, const char *arg, ...)`: 以参数列表的形式传递参数给新程序。参数列表以空指针 `NULL` 结束。
示例：`execl("/bin/ls", "ls", "-l", NULL)`。
调用execl("/bin/ls", "ls", "-l", NULL)会启动/bin/ls程序，并将其替换当前进程的映像。新程序将被执行，并以参数"ls"和"-l"运行。
2. `int execv(const char *path, char *const argv[])`: 以字符串数组的形式传递参数给新程序。参数数组以空指针 `NULL` 结束。
示例：`char *args[] = {"ls", "-l", NULL}; execv("/bin/ls", args)`;
调用execv("/bin/ls", args)会启动/bin/ls程序，并将其替换当前进程的映像。新程序将被执行，并以参数"ls"和"-l"运行。
3. `int execle(const char *path, const char *arg, ..., char *const envp[])`: 与 `execl()` 类似，但可以指定新程序的环境变量。
示例：`execle("/bin/ls", "ls", "-l", NULL, envp)`。
调用execle("/bin/ls", "ls", "-l", NULL, envp)会启动/bin/ls程序，并将其替换当前进程的映像。新程序将被执行，并以参数"ls"和"-l"以及指定的环境变量运行。  
4. `int execve(const char *path, char *const argv[], char *const envp[])`: 与 `execv()` 类似，但可以指定新程序的环境变量。
示例：`char *args[] = {"ls", "-l", NULL}; execve("/bin/ls", args, envp)`。
调用 execve("/bin/ls", args, envp) 将执行 /bin/ls 程序，并将参数数组 args 和环境变量数组 envp 传递给它。
5. `int execvp(const char *file, char *const argv[])`: 在系统的搜索路径中查找可执行文件，并以字符串数组的形式传递参数给新程序。
示例：`char *args[] = {"ls", "-l", NULL}; execvp("ls", args)`。
调用 execvp("ls", args) 会在系统的路径变量中搜索可执行文件名为 "ls" 的程序，并执行找到的程序。根据常规配置，ls 命令在 /bin 目录下，因此实际上会执行 /bin/ls 程序，并以参数 "ls" 和 "-l" 运行。
这些函数在执行成功时不会返回，因此如果需要在 `exec()` 调用后继续进行其他操作，可以使用 `fork()` 系列函数创建一个新进程，然后在子进程中调用 `exec()` 函数。这样，子进程会被替换为新的程序映像，而父进程可以继续执行其他操作。
注意，在调用 `exec()` 函数时，路径参数需指定正确的可执行文件路径。另外，参数数组或参数列表的最后一个元素必须是空指针 `NULL`。在使用 `exec()` 函数时，还应注意错误处理和错误信息的输出。
9、sbrk
char *sbrk(int n), 该函数接受一个整数参数 n，表示要增加的字节数。
sbrk 函数会将当前进程的堆空间增加 n 字节，并返回新分配内存的起始地址。如果增加成功，返回的地址是原堆空间的末尾，即新分配内存的起始地址。如果增加失败，函数返回 -1。
以下是一个示例代码片段，展示了如何使用 sbrk 函数来增加当前进程的内存空间：
#include "types.h"
#include "user.h"

int main() {
    printf("Before sbrk\n");

    int n = 4096; // 增加 4096 字节（4 KB）的内存空间

    char *new_mem = sbrk(n);
    if (new_mem == (char *)-1) {
        printf("sbrk failed\n");
        return 1;
    }

    printf("New memory allocated: %p\n", new_mem);

    return 0;
}
在上面的示例中，调用 sbrk(n) 会增加当前进程的内存空间，大小为 n 字节。如果增加成功，sbrk 函数返回新分配内存的起始地址，这个地址存储在 new_mem 变量中。如果增加失败，sbrk 函数返回 -1，在示例中会打印出错误信息。
注意，这个示例是在 xv6 操作系统下运行的，types.h 和 user.h 是 xv6 操作系统的头文件。在其他操作系统或环境中使用 sbrk 函数时，可能需要使用不同的头文件和库。
第一次更新：2023/9/8
10、write()
`int write(int fd, char *buf, int n)` 函数用于将指定数量的字节从缓冲区 `buf` 写入文件描述符为 `fd` 的文件。
函数原型为 `int write(int fd, char *buf, int n)`，它接受三个参数：`fd` 表示文件描述符，`buf` 是要写入的数据的缓冲区指针，`n` 表示要写入的字节数。
`write` 函数将缓冲区 `buf` 中的 `n` 个字节写入文件，返回实际写入的字节数。如果写入成功，返回值将等于参数 `n`；如果写入失败，返回值可能小于 `n`，表示写入的字节数少于请求的字节数。
以下是一个示例代码片段，展示了如何使用 `write` 函数将数据写入文件：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    printf("before write\n");

    int fd = open("hello.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open");
        return 1;
    }
    char *buf = "Hello, World!";
    int n = 13; // 要写入的字节数

    int bytes_written = write(fd, buf, n);
    if (bytes_written == -1) {
        perror("write");
        close(fd);
        return 1;
    }
    printf("bytes written: %d\n", bytes_written);
    close(fd);
    return 0;
}
下面是代码的运行过程和输出结果的示例：

添加图片注释，不超过 140 字（可选）

添加图片注释，不超过 140 字（可选）
可以知道，首先使用 `open` 函数打开一个名为 "hello.txt" 的文件，如果打开失败则输出错误信息并返回。然后，使用 `write` 函数将长度为 13 的字符串 "Hello, World!" 写入文件。写入成功后，`write` 函数返回实际写入的字节数，这里应该是 13。最后，使用 `close` 函数关闭文件描述符。

注意，代码中的文件打开模式 `O_WRONLY | O_CREAT | O_TRUNC` 指定了以写入方式打开文件，并且如果文件不存在则创建新文件。文件权限 `0644` 表示文件所有者具有读写权限，而其他用户只具有读权限。在使用 `write` 函数时，需要确保文件已经成功打开，并在使用完后关闭文件描述符。

11、read()
`read` 函数用于从文件描述符为 `fd` 的文件中读取指定数量的字节到缓冲区 `buf` 中。
该函数原型为 `int read(int fd, char *buf, int n)`，它接受三个参数：`fd` 表示文件描述符，`buf` 是接收读取数据的缓冲区指针，`n` 表示要读取的字节数。
`read` 函数从文件描述符 `fd` 指定的文件中读取最多 `n` 个字节数据，并将其存储到缓冲区 `buf` 中。函数返回实际读取的字节数。如果已经到达文件末尾（即没有更多数据可读），则返回值为 0。
以下是一个示例代码片段，展示了如何使用 `read` 函数从文件中读取数据：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    printf("before read\n");
    int fd = open("hello.txt", O_RDONLY);
    if (fd == -1) {
        perror("file open failed");
        return 1;
    }
    char buf[100];
    int n = 10; // 要读取的字节数

    int bytes_read = read(fd, buf, n);
    if (bytes_read == -1) {
        perror("read");
        close(fd);
        return 1;
    }
    if (bytes_read == 0) {
        printf("end of file\n");
    } else {
        printf("bytes read: %d\n", bytes_read);
        printf("data read: %.*s\n", bytes_read, buf);
    }
    close(fd);
    return 0;
}
下面是代码的运行过程和输出结果的示例：

添加图片注释，不超过 140 字（可选）
在上述的示例中，首先使用 `open` 函数打开一个名为 "hello.txt" 的文件进行读取，如果打开失败则输出错误信息并返回。然后，使用 `read` 函数从文件中读取最多 10 个字节的数据，并存储到大小为 100 的缓冲区 `buf` 中。读取成功后，`read` 函数返回实际读取的字节数，并根据返回值进行相应的处理。
如果返回值为 0，则表示已经到达文件末尾，输出 "end of file"。否则，输出实际读取的字节数和读取到的数据。
注意，在使用 `read` 函数之前，需要确保文件已经成功打开，并在使用完后关闭文件描述符。

12、close()
`close` 函数用于关闭先前通过文件描述符 `fd` 打开的文件，并释放该文件描述符。
函数原型为 `int close(int fd)`，它接受一个整数参数 `fd`，表示要关闭的文件描述符。
`close` 函数将关闭与文件描述符 `fd` 相关联的文件，并释放该文件描述符以供重用。
以下是一个示例代码片段，展示了如何使用 `close` 函数关闭文件：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    printf("before close\n");

    int fd = open("file.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }
    // 在此处执行文件读取或写入操作...

    int result = close(fd);
    if (result == -1) {
        perror("close");
        return 1;
    }
    printf("file closed\n");
    return 0;
}
在上面的示例中，首先使用 `open` 函数打开名为 "file.txt" 的文件，并将返回的文件描述符存储在变量 `fd` 中。如果打开失败，则输出错误信息并返回。然后可以在 `open` 和 `close` 之间执行文件的读取或写入操作。最后，使用 `close` 函数关闭文件描述符 `fd`。如果关闭成功，`close` 函数返回 0。如果关闭失败，将返回 -1，并通过 `perror` 函数输出错误信息。
注意，关闭文件描述符是一个良好的编程习惯，以确保及时释放系统资源。在使用完文件后，应该尽早关闭文件描述符。
13、dup()
`dup` 函数用于复制文件描述符 `fd`，并返回一个新的文件描述符，该描述符指向与 `fd` 相同的文件。
函数原型为 `int dup(int fd)`，它接受一个整数参数 `fd`，表示要复制的文件描述符。
`dup` 函数会创建一个新的文件描述符，它指向与 `fd` 相同的文件。新的文件描述符与 `fd` 具有相同的文件偏移量和文件状态标志。
以下是一个示例代码片段，展示了如何使用 `dup` 函数复制文件描述符：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    printf("before dup\n");

    int fd1 = open("hello.txt", O_RDONLY);
    if (fd1 == -1) {
        perror("open");
        return 1;
    }

    int fd2 = dup(fd1);
    if (fd2 == -1) {
        perror("dup");
        close(fd1);
        return 1;
    }

    char buf1[100];
    char buf2[100];

    int n = 10; // 要读取的字节数
    // 在此处使用 fd1 和 fd2 进行文件读取或写入操作...
    int bytes_read1 = read(fd1, buf1, n);
    int bytes_read2 = read(fd2, buf2, n);


    printf("bytes read1: %d\n", bytes_read1);
    printf("-------------------------------\n");
    printf("bytes read2: %d\n", bytes_read2);

    printf("data read1: %.*s\n", bytes_read1, buf1);
    printf("-------------------------------\n");
    printf("data read2: %.*s\n", bytes_read2, buf2);

    close(fd1);
    close(fd2);

    return 0;
}
下面是代码的运行过程和输出结果的示例：

添加图片注释，不超过 140 字（可选）

在上面的示例中，首先使用 `open` 函数打开名为 "file.txt" 的文件，并将返回的文件描述符存储在变量 `fd1` 中。如果打开失败，则输出错误信息并返回。
然后，使用 `dup` 函数复制文件描述符 `fd1`，并将返回的新文件描述符存储在变量 `fd2` 中。如果复制失败，则输出错误信息并关闭 `fd1`，然后返回。在复制成功后，可以使用 `fd1` 和 `fd2` 进行文件的读取或写入操作。它们都指向相同的文件，因此对其中一个文件描述符的操作会影响另一个。最后，使用 `close` 函数关闭文件描述符 `fd1` 和 `fd2`，释放系统资源。
注意，`dup` 函数会复制文件描述符，但并不会复制文件状态标志 `O_RDONLY`、`O_WRONLY` 等。它只是复制文件描述符的底层值和指向相同文件的指针。
14、pipe()
`pipe` 函数用于创建一个管道，并将读取和写入的文件描述符分别存储在数组 `p` 的 `p[0]` 和 `p[1]` 中。
函数原型为 `int pipe(int p[])`，它接受一个整数数组 `p` 作为参数，用于存储管道的读取和写入文件描述符。
`pipe` 函数会创建一个无名管道，其中 `p[0]` 表示读取端的文件描述符，`p[1]` 表示写入端的文件描述符。通过管道，可以在两个进程之间进行进程间通信（IPC）。
以下是一个示例代码片段，展示了如何使用 `pipe` 函数创建管道，并使用read和write进行简单通信：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("before pipe\n");

    int p[2]; // 用于存储读取和写入文件描述符的数组

    int result = pipe(p);
    if (result == -1) {
        perror("pipe");
        return 1;
    }

    int read_fd = p[0]; // 读取端的文件描述符
    int write_fd = p[1]; // 写入端的文件描述符

    // 在此处使用 read_fd 和 write_fd 进行进程间通信...

    char message[] = "Hello,World!";
    write(write_fd, message, sizeof(message));

    char buffer[100];
    read(read_fd, buffer, sizeof(buffer));
    printf("Received message: %s\n", buffer);

    close(read_fd);
    close(write_fd);

    return 0;
}


添加图片注释，不超过 140 字（可选）
在上面的示例中，首先定义一个整数数组 `p`，用于存储读取和写入文件描述符。然后，使用 `pipe` 函数创建管道，并将读取端和写入端的文件描述符分别存储在 `p[0]` 和 `p[1]` 中。如果创建管道失败，则输出错误信息并返回。在创建管道成功后，可以使用 `p[0]` 作为读取端的文件描述符，使用 `p[1]` 作为写入端的文件描述符，进行进程间通信。最后，使用 `close` 函数关闭读取端和写入端的文件描述符，释放系统资源。
注意，管道是一种半双工通信机制，数据在管道中以先进先出（FIFO）的顺序传输。一个进程可以将数据写入管道的写入端，另一个进程可以从管道的读取端读取数据。
15、chdir
`chdir` 函数用于改变当前的工作目录为指定的目录。
函数原型为 `int chdir(const char *dir)`，它接受一个字符串参数 `dir`，表示要切换到的目标目录。
`chdir` 函数会将当前进程的工作目录更改为指定的目录 `dir`。如果目录切换成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `chdir` 函数改变当前的工作目录：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("before chdir\n");
    const char *dir = "/home/ubuntu/test_chdir";
    int result = chdir(dir);
    if (result == -1) {
        perror("chdir");
        return 1;
    }
    printf("Current working directory changed to: %s\n", dir);
    return 0;
}

添加图片注释，不超过 140 字（可选）
在上面的示例中，首先定义一个字符串 `dir`，表示要切换到的目标目录。然后，使用 `chdir` 函数将当前的工作目录更改为指定的目录。如果目录切换成功，将输出 "Current working directory changed to: /path/to/directory"。如果发生错误，将输出错误信息并返回。
注意，`chdir` 函数在切换目录时需要确保目标目录的存在和可访问性。另外，不同的操作系统可能对目录的表示方式有所差异（例如使用相对路径或绝对路径）。因此，在使用 `chdir` 函数时，需要根据特定的操作系统和文件系统进行适当的路径处理。

16、mkdir()
`mkdir` 函数用于创建一个新的目录。
函数原型为 `int mkdir(const char *dir, mode_t mode)`，它接受两个参数：`dir` 表示要创建的目录的路径名，`mode` 表示目录的权限模式。
`mkdir` 函数会在指定路径 `dir` 下创建一个新的目录。如果目录创建成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `mkdir` 函数创建新的目录：
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>

int main() {
    printf("before mkdir\n");

    const char *dir = "/home/ubuntu/test_mkdir";
    mode_t mode = 0755; // 目录的权限模式

    int result = mkdir(dir, mode);
    if (result == -1) {
        perror("mkdir");
        return 1;
    }

    printf("Directory created: %s\n", dir);

    return 0;
}
在上面的示例中，首先定义一个字符串 `dir`，表示要创建的目录的路径名。然后，定义一个 `mode_t` 类型的变量 `mode`，表示目录的权限模式。使用 `mkdir` 函数创建新的目录时，需要指定目录的路径名和权限模式。在示例中，目录的权限模式设置为 0755。如果目录创建成功，将输出 "Directory created: /path/to/new_directory"。如果发生错误，将输出错误信息并返回。
注意，`mkdir` 函数在创建目录时需要确保路径的合法性，并具有足够的权限进行创建操作。另外，不同的操作系统可能对路径名的表示方式有所差异（例如使用相对路径或绝对路径）。因此，在使用 `mkdir` 函数时，需要根据特定的操作系统和文件系统进行适当的路径处理。
17、mknod
`mknod` 函数用于创建一个设备文件。
函数原型为 `int mknod(const char *file, mode_t mode, dev_t dev)`，它接受三个参数：`file` 表示要创建的设备文件的路径名，`mode` 表示文件的权限模式，`dev` 表示设备的 ID。
`mknod` 函数会根据给定的路径名 `file`、权限模式 `mode` 和设备 ID `dev` 创建一个设备文件。如果设备文件创建成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `mknod` 函数创建设备文件：
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    printf("before mknod\n");

    const char *file = "/home/ubuntu/device_file";
    mode_t mode = S_IFCHR | 0666; // 文件类型为字符设备文件，权限模式设置为 0666
    dev_t dev = makedev(256, 0); // 设备 ID

    int result = mknod(file, mode, dev);
    if (result == -1) {
        perror("mknod");
        return 1;
    }

    printf("Device file created: %s\n", file);

    return 0;
}
在上面的示例中，首先定义一个字符串 `file`，表示要创建的设备文件的路径名。然后，定义一个 `mode_t` 类型的变量 `mode`，表示文件的权限模式。通过使用 `S_IFCHR` 宏，将文件类型设置为字符设备文件。权限模式设置为 0666，表示具有读写权限。
使用 `makedev` 函数创建设备 ID，其中第一个参数是主设备号，第二个参数是次设备号。在示例中，设备 ID 设置为主设备号为 256，次设备号为 0。然后，使用 `mknod` 函数根据给定的路径名、权限模式和设备 ID 创建设备文件。如果设备文件创建成功，将输出 "Device file created: /home/ubuntu/device_file"。如果发生错误，将输出错误信息并返回。
注意，使用 `mknod` 函数创建设备文件需要具有适当的权限和特权。设备文件通常用于与底层硬件进行交互，因此需要谨慎操作，并确保了解设备文件的使用和配置要求。
18、fstat
`fstat` 函数用于将打开文件的信息存储在 `struct stat` 结构体中。
函数原型为 `int fstat(int fd, struct stat *st)`，它接受两个参数：`fd` 是已打开文件的文件描述符，`st` 是指向 `struct stat` 结构体的指针。
`fstat` 函数会获取与文件描述符 `fd` 关联的文件信息，并将其存储在 `st` 指向的 `struct stat` 结构体中。如果获取文件信息成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `fstat` 函数获取文件信息：
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    printf("before fstat\n");

    const char *file = "/home/ubuntu/test";
    int fd = open(file, O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    struct stat st;
    int result = fstat(fd, &st);
    if (result == -1) {
        perror("fstat");
        close(fd);
        return 1;
    }

    printf("File size: %ld bytes\n", st.st_size);
    printf("uid: %d\n", st.st_uid);
    printf("gid: %d\n", st.st_gid);

    close(fd);

    return 0;
}

在上面的示例中，首先定义一个字符串 `file`，表示要打开的文件路径名。然后，使用 `open` 函数打开文件，获取文件的文件描述符 `fd`。如果打开文件失败，将输出错误信息并返回。然后，定义一个 `struct stat` 结构体变量 `st`，用于存储文件信息。然后，使用 `fstat` 函数将文件描述符 `fd` 关联的文件信息存储在 `st` 结构体中。如果获取文件信息成功，将输出文件大小 "File size: [size] bytes"，其中 `[size]` 是文件的大小（以字节为单位）。之后再打印uid和gid，最后，使用 `close` 函数关闭文件描述符，释放系统资源。
注意，`fstat` 函数需要一个已打开的文件描述符作为参数。通过获取文件信息，可以获得诸如文件大小、访问权限、创建时间等属性。`struct stat` 结构体包含了许多字段，可以根据需要访问其中的信息。
19、stat
`stat` 函数用于将指定文件的信息存储在 `struct stat` 结构体中。
stat函数原型为 `int stat(const char *file, struct stat *st)`，它接受两个参数：`file` 是要获取信息的文件路径名，`st` 是指向 `struct stat` 结构体的指针。
`stat` 函数会获取指定文件 `file` 的信息，并将其存储在 `st` 指向的 `struct stat` 结构体中。如果获取文件信息成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `stat` 函数获取文件信息：
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    printf("before stat\n");

    const char *file = "/home/ubuntu/test";
    struct stat st;
    int result = stat(file, &st);
    if (result == -1) {
        perror("stat");
        return 1;
    }

    printf("File size: %ld bytes\n", st.st_size);

    return 0;
}
在上面的示例中，首先定义一个字符串 `file`，表示要获取信息的文件路径名。然后，定义一个 `struct stat` 结构体变量 `st`，用于存储文件信息。使用 `stat` 函数将指定文件 `file` 的信息存储在 `st` 结构体中。如果获取文件信息成功，将输出文件大小 "File size: [size] bytes"，其中 `[size]` 是文件的大小（以字节为单位）。
注意，`stat` 函数需要指定一个文件路径作为参数，以获取该文件的属性信息。`struct stat` 结构体包含了许多字段，可以根据需要访问其中的信息，如文件大小、访问权限、创建时间等。
总结：
stat 使用文件路径名作为参数，可以获取指定路径的文件属性。
fstat 使用文件描述符作为参数，可以获取与文件描述符关联的文件的属性。
要注意的是，stat 和 fstat 函数执行成功时返回值为 0，执行失败时返回值为 -1，并设置相应的错误码。在使用这些函数时，应该检查返回值以处理可能的错误情况。
20、link
`link` 函数用于为文件 `file1` 创建另一个名称 `file2`。
函数原型为 `int link(const char *file1, const char *file2)`，它接受两个参数：`file1` 是要创建链接的源文件路径名，`file2` 是新链接文件的路径名。
`link` 函数会创建一个新的链接文件 `file2`，它与源文件 `file1` 共享相同的内容。如果创建链接成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `link` 函数创建文件链接：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("before link\n");

    const char *file1 = "/home/ubuntu/test";
    const char *file2 = "/home/ubuntu/test_link_file";

    int result = link(file1, file2);
    if (result == -1) {
        perror("link");
        return 1;
    }

    printf("Link created: %s\n", file2);

    return 0;
}

在上面的示例中，首先定义两个字符串 `file1` 和 `file2`，分别表示源文件和新链接文件的路径名。
使用 `link` 函数将源文件 `file1` 创建一个新的链接文件 `file2`。如果创建链接成功，将输出 "Link created: /home/ubuntu/test_link_file"。
注意，`link` 函数在创建链接时需要确保路径的合法性，并具有足够的权限进行创建操作。另外，链接的文件系统必须支持文件链接功能。
创建链接后，源文件和链接文件将引用相同的物理文件内容。修改源文件或链接文件中的内容将影响另一个文件。同时，删除源文件或链接文件中的一个将不会影响另一个文件，因为它们只是引用同一份物理内容。
21、unlink
`unlink` 函数用于删除一个文件。
函数原型为 `int unlink(const char *file)`，它接受一个参数 `file`，表示要删除的文件的路径名。
`unlink` 函数会删除指定路径名的文件。如果删除成功，函数返回值为 0。如果发生错误，返回值为 -1，并设置相应的错误码。
以下是一个示例代码片段，展示了如何使用 `unlink` 函数删除文件：
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("before unlink\n");

    const char *file = "/home/ubuntu/test_link_file";

    int result = unlink(file);
    if (result == -1) {
        perror("unlink");
        return 1;
    }

    printf("File deleted: %s\n", file);

    return 0;
}
在上面的示例中，首先定义一个字符串 `file`，表示要删除的文件的路径名。
使用 `unlink` 函数删除指定路径名的文件。如果删除成功，将输出 "File deleted: /home/ubuntu/test_link_file"。
注意，`unlink` 函数在删除文件时需要确保路径的合法性，并具有足够的权限进行操作。一旦文件被删除，将无法恢复。因此，在使用 `unlink` 函数删除文件之前，请确保要删除的文件是正确的，并且你有足够的权限和权责来执行此操作。
第二次更新：2023/9/10
