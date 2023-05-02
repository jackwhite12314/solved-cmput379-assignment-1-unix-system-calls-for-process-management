Download Link: https://assignmentchef.com/product/solved-cmput379-assignment-1-unix-system-calls-for-process-management
<br>
This programming assignment is intended to give you experience in using UNIX system calls for process management (e.g., fork(), execv(), wait(), open(), and close()) and establishing pipes for interprocess communication.

An Interactive Shell

Shell programs, such as the standard shell (sh), Bourne-again shell (bash) and C shell (csh), provide a powerful programming environment that allows users to utilize many of the services provided by the operating system. In this assignment, you will implement a simple, <em>interactive </em>command-line interpreter or shell, which we call dragonshell, in C or C++ (whichever you prefer). An interactive shell displays a prompt, reads and executes commands from user, prints out the result, and displays the prompt again. Unlike shells that support batch processing only, the user can interact with such a shell.

Your shell program must be able to execute a small set of built-in commands besides any external command that can be located in the path. In the former case, the shell contains the code to execute these commands. In the latter case, the shell does not understand the command, so it merely searches for the command name in its path or the working directory to identify the corresponding program and executes it with the provided arguments.

To implement your shell program, you will have to use system calls directly in your code. Thus, the first step is to familiarize yourself with UNIX system calls including the ones mentioned in this table:

<table width="162">

 <tbody>

  <tr>

   <td width="86">system call</td>

   <td width="76">man page</td>

  </tr>

  <tr>

   <td width="86">chdir()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/chdir.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">fork()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/fork.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">execve()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/execve.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">exit()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/exit.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">wait()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/wait.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">open()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/open.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">close()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/close.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">dup2()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/dup.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">pipe()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/pipe.2.html">man</a></td>

  </tr>

  <tr>

   <td width="86">kill()</td>

   <td width="76"><a href="http://man7.org/linux/man-pages/man2/kill.2.html">man</a></td>

  </tr>

 </tbody>

</table>

<h2>Feature Highlights</h2>

Your task is to implement an interactive shell with the following required features:

<ol>

 <li>Support the basic commands listed below. These commands must be implemented in your shell.

  <ul>

   <li>cd for changing the present working directory;</li>

   <li>pwd for printing the present working directory;</li>

   <li>a2path for overwriting or appending an address to the path variable; (d) exit for gracefully terminating the shell and all forked processes.</li>

  </ul></li>

 <li>Run an external program with the provided arguments when its full (absolute) path is provided, or when the program can be located in the path or in the present working directory.</li>

 <li>Run multiple commands written in a single line, separated by “;”.</li>

 <li>Support background execution when an “&amp;” is put at the end of the command line.</li>

 <li>Support redirecting the output from one program to a file.</li>

 <li>Support piping the output of one program to another program.</li>

 <li>Handle signals, generated by specific keyboard shortcuts, for stopping or pausing processes created by the shell. In particular, it should capture and handle Ctrl-C which sends SIGINT, and Ctrl-Z which sends SIGTSTP. Note that pressing Ctrl-C should not kill the shell, nor should Ctrl-Z interrupt the shell and redirect you to bash.</li>

</ol>

Below we describe these requirements in detail.

<h2>Create a New Process for Each Command</h2>

When user enters a new command, dragonshell should either create a new process for executing this command with the specified arguments or print out an error message (dragonshell: Command not found) if the command is not a recognized built-in command, and it is not found in the present working directory and in the shell’s path. The shell must return control to user (i.e., displays the prompt) only after the new process terminates. Note that a program can take zero, one, or several space separated arguments. Example

dragonshell &gt; pwd

/home/&lt;csid&gt;/CMPUT-379/Assignment-1 dragonshell &gt; /usr/bin/which which

/usr/bin/which dragonshell &gt; rand dragonshell: Command not found

In the above example, we assumed that /usr/bin/which is the full path to the which program, and the rand program was not found in the shell’s path or in the current working directory.

Note: You must use the shell’s path variable defined using a built-in command (described below) rather than the PATH environment variable.

<h2>Add Support for Built-in Commands</h2>

The following built-in commands must be implemented by dragonshell rather than invoking external programs which perform the same operations.

<h3>A. <strong>cd </strong>Command</h3>

Like bash and a lot of other shells, cd is used to change the current working directory. cd should take only one argument which is the full or relative path of a directory that user wants to change to. Example

dragonshell &gt; pwd

/home/&lt;csid&gt;/CMPUT-379/Assignment-1

dragonshell &gt; cd .. dragonshell &gt; pwd

/home/&lt;csid&gt;/CMPUT-379 dragonshell &gt; cd Assignment-12 dragonshell: No such file or directory dragonshell &gt; cd dragonshell: expected argument to “cd” dragonshell &gt;

You need to handle possible error conditions. Specifically, the shell must prompt the user if it cannot find the directory or is missing an argument. In the former case, it should output dragonshell: No such file or directory and in the latter case it should output dragonshell: expected argument to “cd”.

<h3>B. <strong>pwd </strong>Command</h3>

Like bash and a lot of other shells pwd is used to show the current directory. pwd should take no arguments to show the present working directory. Example

dragonshell &gt; pwd

/home/&lt;csid&gt;/CMPUT-379/Assignment-1 dragonshell &gt;

<h3>C. Showing the path and <strong>a2path </strong>Command</h3>

Typing in $PATH should tell the user the shell’s path. The a2path command should allow the user to update the path either by overwriting it or appending to it by using $PATH:&lt;new-path&gt;. Like bash, you should use colon (:) as the delimiter to separate different paths. Example

dragonshell &gt; $PATH

Current PATH: /bin/:/usr/bin/ dragonshell &gt; a2path $PATH:/usr/local/bin dragonshell &gt; $PATH

Current PATH: /bin/:/usr/bin/:/usr/local/bin dragonshell &gt;

You can assume that /bin/ and /usr/bin/ are initially included in the path.

<h3>D. <strong>exit </strong>Command</h3>

When the user types exit or presses Ctrl – D , the shell should gracefully terminate all processes running in the background (if any) before terminating itself.

<h3>Example</h3>

dragonshell &gt; exit

Exiting

<em># back to normal shell</em>

The same behaviour is expected if Ctrl – D          is pressed.

<h2>Run External Programs</h2>

Your shell must be able to run any program that is found within its PATH; in the present working directory; or in the full path specified by user, and print out its output (similar to what happens when you run this program in bash). If the program is not found in the shell’s PATH or the specified path, dragonshell should prompt the user that an error had occurred that the program is not found. Example

dragonshell &gt; /usr/bin/touch readme.md dragonshell &gt; ls readme.md readme.md

In the above example, the ls command was found in the shell’s PATH, and therefore could be launched by the shell.

<h2>Output Redirection</h2>

When running programs, it’s sometimes useful to direct the output to a file. The syntax [process] &gt; [file] tells your shell to redirect the process’s standard output to a file. Bash allows you to chain input and output redirection within a single line, but for simplicity we will only support output redirection. Example

dragonshell &gt; ls &gt; lsout.txt

This should redirect the output from ls to the file lsout.txt. It should not display anything to the shell. If the file already exist then it will overwrite the previous content of the file. Otherwise, it will create a new file.

<h2>Pipe</h2>

A useful feature of dragonshell is that allows for processing the output from one command using another command. This is where pipes are useful. For example, you want to list the first ten files within a directory (recursively) that has a .cpp file extension. You can do in bash by running the line below.

$ find ./ | grep “.cpp” | head

dragonshell needs to support piping as well. The syntax ”[process1] | [process2]” tells your shell to redirect the output from process 1 to the input of process 2.

Note: For pipes you only need to support one level of piping. i.e., you will not need to support multiple, or chained pipes.

<h3>Example</h3>

dragonshell &gt; find ./ | sort

./

./dragonshell

./dragonshell.cpp

./dragonshell.hpp

./lsout.txt

./Makefile

./README.md

This should return a sorted list of all files and folders listed in the current working directory.

<h2>Running Multiple Commands</h2>

Most shells can run multiple commands written in a single command line. For example, in bash you can run multiple commands in a single line using “;” as the delimiter.

Similarly, in dragonshell the syntax ”[command1] ; [command2] will first run command1 and then command2. Many commands can be executed sequentially using this syntax as long as a “;” is put between every two commands. The shell will give back control to the user after all the commands have been processed. Example

dragonshell &gt; ls -al &gt; lsout.txt ; find ./ | sort ; whereis gcc

./

./dragonshell

./dragonshell.cpp

./dragonshell.hpp

./lsout.txt ./Makefile gcc: /usr/bin/gcc /usr/lib/gcc /usr/share/man/man1/gcc.1.gz dragonshell &gt;

This will write the output from ls -al to a file called lsout.txt, then it will return a sorted list of all files and folders listed in the current working directory, finally it will return the location of where the gcc binary is stored.

You program should be able to run an ”infinite” amount of commands in sequence.

<h2>Handling Signals</h2>

Most shells let you stop or pause processes with special keystrokes. These special keystrokes, such as Ctrl-C and Ctrl-Z, work by sending signals to the shell’s subprocesses. If we send these signals to your shell either by keystrokes or by terminal then it will terminate or stop the shell itself; we do not want that to happen in dragonshell, so we have to send signals to the shell’s subprocesses only. Expected behaviour: this is what we expect to happen

dragonshell &gt; ˆC dragonshell &gt; ˆC dragonshell &gt; ˆZ dragonshell &gt;

Rather than this

dragonshell &gt; ˆC

<em># back to normal bash</em>

or this

dragonshell &gt; ˆZ

[1]+ Stopped                                                             ./dragonshell

<em># back to normal bash</em>

<h2>Putting Jobs in Background</h2>

So far, your shell waits for each process to finish before starting the next one. But most UNIX shells allow the user to run a command in the background by putting an “&amp;” at the end of the command line. In this case, the shell allows the user to run other commands without waiting for the background process to finish.

Your shell program must support the background execution of a command when the “&amp;” character appears at the end of the command line. You will have at most one program running in the background and it will be an external program rather than a built-in command. Example

dragonshell &gt; ls &amp;

PID 1741 is running in the background dragonshell &gt;

When you put a process in the background, your shell should output the same message as in the above example, except that 1741 is replaced with the PID of the new process that is put in the background.

You do not need to write a function that will bring a background process to the foreground. Furthermore, the process running in the background does not need to send a message to the terminal upon completion.