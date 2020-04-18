---
title: "Thoughts on Linux"
layout: post
date: 2020-04-20 12:00:00 CEST
---

I've been working on system-level software for Linux for some time now,
which is a job that makes you much more likely to encounter the odd corner
of an operating system than higher level development. For all practical
purposes, Linux has succeeded as an operating system. Billions of web queries
are being served from Linux servers every day, and it's running on [billions] of
mobile devices.

But of course an operating system ultimately rooted in the 70s will have
accumulated its fair share of legacy. Of course, not all problems are rooted
in this: we still are perfectly capable of making new and exciting mistakes.

# Everything-ish is a file-ish
File descriptors are one of the most poorly named concepts, as they don't
necessarily represent (or describe) a file. Think of them more as pointers to
resources in the kernel (which are conveniently referenced-counted), and things
will start making more sense. You can use file descriptors to refer to many
things, among them

* [Files and Directories](http://man7.org/linux/man-pages/man2/open.2.html)
* [Sockets](http://man7.org/linux/man-pages/man2/socket.2.html)
* [Pipes](http://man7.org/linux/man-pages/man2/pipe.2.html)
* [Timers](http://man7.org/linux/man-pages/man2/timerfd_create.2.html)
* [Incoming signals](http://man7.org/linux/man-pages/man2/signalfd.2.html)
* [Anonymous memory](http://man7.org/linux/man-pages/man2/memfd_create.2.html)
* [Events](http://man7.org/linux/man-pages/man2/eventfd.2.html)
* [Processes](http://man7.org/linux/man-pages/man2/pidfd_open.2.html)
* [Performance counters](http://man7.org/linux/man-pages/man2/perf_event_open.2.html)
* Sadly, [not locks](https://lwn.net/Articles/280960/)

Many of these things are not files in the either the sense that you can use
them to store data, nor in the sense that they have a path on the file system.
That all of these things are file descriptors is actually kind of cool: you can
wait on many of these things, and asynchronous I/O APIs like `epoll` operate on
file descriptors. This means that you can wait for data to become available on
some socket *or* for some timer to run out *or* for a POSIX signal to arrive.
No threading nonsense involved.

On the other hand, a file descriptor to an application developer is just an
integer. From an API perspective, now you end up with all of these things just
being integers, and all being called fd, and many functions operating on fds,
all of them working on a different subset of them. For instance, what would it
even mean to write to a timerfd? This gets even worse when talking about errors.
Manpages will have a list of possible errnos, but with file descriptor
functions, usually all bets are off (short of delving into the kernel source)
of trying to understand which ones can happen for your particular case. If
your FD is special enough (say, some `/sys` file), you might even get errors
that are *not* listed on the manpage.

# "Everything is bytes\n\0" {#quiz}
Linux has around 300 system calls that you can use in your programs. This is
sometimes cited as an impressively small API surface for a kernel,
but that's only telling half of the truth. There are many, many files in the
`/proc` and `/sys` virtual file-systems that you can read from to get
information or write to to change some behaviour. You can use them to read a
process' command line, to get its CPU use, its memory use, to get or change the
maximum number of file-descriptors open, set counters on breakpoints, etc.
Long story short, you can do many things. If you are writing systems software,
you will be using those quite a lot.

Most of those produce (or consume) information in ASCII, so if you want to use
them programatically, you'll have to parse the output. If you want to do that,
it helps to know the format well.

Pop quiz! No cheating.[^2]

1. Does `/proc/<pid>/cmdline` have a trailing NUL-byte?
2. Does `/proc/<pid>/status` use tabs or spaces?
3. Does `/proc/<pid>/status` have a trailing newline?
4. NUL-byte?
6. Does `/proc/<pid>/oom_score` (a single value) have a trailing newline?
7. Does `/proc/<pid>/wchan` (a single value) have a trailing newline?

[Go to results](#quiz-results).

So that was fun! If you haven't looked at the results yet, please do take a
second, even if you haven't tried to answer yourself. There are multiple problems
highlighted: most of `/proc` files are ad-hoc text formats (at least *most* of
them do use newlines, it appears). Another one is that they are extremely poorly
documented. A quick look at [`man 5 proc`](
http://man7.org/linux/man-pages/man5/proc.5.html) would not go far in answering
the questions in this quiz. You have to look at the files and work out the
format from this. Of course, if you read this from a program, the kernel
rendering text only for your program to parse it back again is wasteful in both
computer and human resources. This being ad-hoc formats, one has to write a
parser for each and every one of them.

For some high-volume formats like ftrace, parsing from the text format is
actually prohibitively expensive. This is why there is also a (undocumented)
de-facto binary interface. With some configuration in, yes, you guessed it,
a custom text format.

# Manything is a race
How would you implement `killall`? Conceptually, it's fairly simple: you go
through all processes, and kill the ones matching. Having some function
`matches` to determine whether a PID matches the given specification, you'd
write something along the lines of this:

{% highlight c++ %}
std::vector<pid_t> pids = get_all_pids();
for (pid_t pid : pids) {
  if (matches(pid)) {
    kill(pid);
  }
}
{% endhighlight %}

Cool. That seems to work (given some imagination). But if you are a diligent
programmer, maybe having done some multi-threaded programming, something might
look a bit off. All we pass around is an integer `pid`, so how do we know it
refers to the same process in `match(pid)` and in `kill(pid)`? Processes can
go away, and their PIDs will be re-used. In that case, we could potentially
match PID 1234 when it is `retriable_nonsense_job`, but kill
`super_important_process` later.

So how do you write a *correct* `killall`? Turns out, on Linux it's impossible
before 5.1, on other Unices it might still be. All the major operating systems'
implementations are not fundamentally different to the pseudocode above[^1].
On Linux 5.1, you can `opendir` `/proc/\<pid\>`, then use that as a handle to both
read the command line (and other attributes), and `pidfd_send_signal`.

This might remind you of some code you might have written. Something like

{% highlight c++ %}
pid_t pid = fork();
if (pid < 0) {
  oh no;
  abort();
}
if (pid == 0) {
  something in child;
} else {
  int status;
  YOUR_EINTR_WRAPPER(waitpid(pid, &status, 0));
}
{% endhighlight %}

Could PID-reuse mean you could end up waiting on something else? Something
that might never exit, in the worst case? Thankfully, not.
The child process will actually stick around as a *zombie*, even
if it is finished, until you waitpid for it. As you can only wait on your
children, no race can occur like this. Of course, the other thing that can
happen is that the parent dies before it has chance to waitpid. In that case,
the problem is also averted through some gymnastics: the child process gets
*orphaned* and reparented to `init`, which will wait on it to prevent the
zombie from sticking around. *Phew*.

# For fork's sake

[`fork`](http://man7.org/linux/man-pages/man2/fork.2.html) is one of the
fundamental building blocks of Unix systems. It goes way back to the old days of
Unix and is used to create a new process. That process is a copy of the
original one. From an API perspective, to the child it will appear that `fork`
returned 0, while to the parent the PID of the child. Kind of magical if you
think of it.[^exhalation]

Memory does not actually get copied just yet, but only when either the parent
or the child attempt to change it (this is called copy-on-write). File
descriptors will get duplicated (convenient that they are already referenced
counted). Problems already start popping up here: you need to be sure to close
the ones you don't actually need in the child, or you might hold on to them
forever. This gets even worse if you `exec` another program, which (in general)
won't know or care about your file descriptors. To solve the latter problem,
`O_CLOEXEC` (close on exec) was invented. Of course that's not the default, so
people often manually iterate over all FDs (via `/proc`), and then close them
all.

But then came threads, and we went from bad to worse. Because it's unclear what
to do with all these other threads (it's not like they have a call to fork where
they could take different branches), they just get terminated in whatever state
they are when fork gets called. Remember we copied the whole memory state? That
might contain a lock (say, in `malloc`) that was being held by one of those
threads. The next time anything calls `malloc`, it will be stuck forever,
waiting on the now terminated thread. Your child process is toast. Your program
might even be single threaded, but some library you pulled might have created a
thread, that might have happened to do an allocation when you called fork.

They came up with `pthread_atfork` hooks that get called before the fork, and
then afterwards in the parent on the child. That was intended to allow library
developers to make their libraries safe in the presence of forks, but even the
manpage concedes that this "is generally too difficult to be practicable."

But there's more! Even though the memory isn't actually copied, forking is
still an expensive operation for the kernel. It has to copy all sorts of
structures like page tables and so on. That's a waste if all we are going
to do is exec a new program, replacing the page tables once again. So
they came up with `vfork`, which is like fork except that you pinkie-promise
not to call anything except `exec` or `_exit` afterwards. This raises
questions about the API design: if all you are allowed to call is `exec`, why
not create a function that is the combination of vfork an exec? With all the
different exec variations I can see how that would be slightly painful, but at
least not something that makes it super easy to shoot your foot. In fact,
`posix_spawn` is exactly that and was standardised in 2001.

But going further down the route of forking, modern Linux also has something
else up its sleeve: [`clone`]. That let's you mix-and-match what your new
thread/process should share and what it shouldn't. It let's you create
something that has its own process id, but is a thread for all other intents
and purposes. Good luck.
glibc does not provide a syscall wrapper for this, so you'll have to be extra
dedicated to use this.[^wrapper]

Ultimately, things do not compose in a way that make it possible to locally
reason about a program. Yes, if you hold fork and threads exactly right (for
instance by managing all the threads, and having a way to signal them to go
to a safe state, and then forking) you *can* write a correct program. But that
requires you to globally about your program, which only works if you control
every bit of your program, and even then is very hard.

# ENOSYS
Writing cross-platform software is hard. Most software is written for Linux,
and if it happens to run on other Unices, that is a happy accident. For systems
software, that is usually a lost cause, as it needs more powerful APIs than
POSIX provides. So we'll settle for Linux. Most of these APIs were introduced
at some point, so you'd usually target Linux of some version or newer -- this
is a generally unavoidable thing to do. But even then Linux has compile-time
configuration flags that will enable or disable some system calls, or some
arguments to system calls. And some depend on the target architecture. It's a
bit like duck typing: you call it and hope it works.

> ENOSYS The membarrier() system call is not implemented by this
>       kernel.
>
> EINVAL cmd is invalid, or flags is nonzero, or the
>        MEMBARRIER_CMD_GLOBAL command is disabled because the
>        nohz_full CPU parameter has been set, or the
>        MEMBARRIER_CMD_PRIVATE_EXPEDITED_SYNC_CORE and
>        MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED_SYNC_CORE commands
>        are not implemented by the architecture.
>
> -- [`man 2 membarrier`](http://man7.org/linux/man-pages/man2/membarrier.2.html)

Any software that uses one of those system calls is inherently unportable even
between machines of the *same kernel version*. You will notice this
incompatibility at runtime, whenever you attempt to call that syscall.
Some more sanity could be achieved if there was a way to check for this without
actually calling the syscall. Then some static analysis could figure out which
syscalls you use, and then generate a stanza to check whether they exist that
you can run at first startup. Everything would be like `./configure`. What's
not to love?

# Standards
> WE DO NOT BREAK USERSPACE!
>
> -- Linus (2012)

C++ has [The Standard](https://timsong-cpp.github.io/cppwp/n3337/), which, even
being aspirational[^aspirational] at times, gives people a way to abstractly
reason about their programs. Unix has POSIX, but that is a very low
common denominator. Linux-specific details have no standard whatsoever. In the
end, the kernel behaviour is whatever is in the kernel source, and the
kernel promises to not break userspace[^nobreak]. For libc, it's the wild west
without any proper specification once you go beyond what's specified in C and
POSIX.

With the proliferation on non-Glibc based Linux systems (musl on Alpine Linux,
Bionic on Android) this makes interoperability more painful than it has to be.
But more than that: it makes system programming *very hard*. It is often
impossible to work in all situations client code could construct via creative
use of the available syscalls. Ultimately, looking at kernel and userspace in
isolation is not the right approach. In an ideal world, applications should not
need to care about the kernel.

# Errno

Pop quiz again! How did I get this error?

{% highlight sh %}
$ ./a.out 
bash: ./a.out: No such file or directory
{% endhighlight %}

If you guessed that `a.out` didn't exist, that would be one way of getting this
error. But in my case what does not exist is the linker library that is
specified in the ELF file[^link-error]. That's one of the joys of `errno`:
for more complicated operations like this, it is unclear what file the `ENOENT`
error refers to. Some of the socket errnos on the other hand are very specific.
Other syscalls just give you `EINVAL`, and you need to figure out what exactly
is wrong. If all you get is a single integer, I don't think there's a way to
really strike that balance, as you have to encode both what you want humans to
see, but also what the calling code might deal with. For `EINVAL`, a human
would probably want to know which of the arguments was wrong, while the
calling code can't do anything with that information.

# Conclusion
I could go on, but I think I'll wrap it up here. People can only take so much.
[^followup]


Systems programming is exciting, and if you've read this far you'd probably be
inclined to agree. I've shown some of the not-so-nice corners I've come across.
Some are inevitable in an operating system of this size and age. But they also
are a valuable lesson in system design. Generalising a bit

* *Data should be machine-readable*: otherwise, people will have to build ad-hoc
parsers. Those will contain bugs.
* *APIs should allow to locally reason about code*: it is impossible to reason
about all your dependencies in big software systems. Apart from that, it is
very hard to reason about large state spaces.
* *Errors matter:* good error reporting gives a good developer experience.
* *Consistency matters:* having too many knobs fragments your system.
* *Behaviour should be specified:* This
  creates a contract between application programmers and operating system
  programmers. People will violate that contract, but no one knows what to
  expect if there isn't any.

# Quiz results {#quiz-results}
1. Maybe.[^maybe]
2. Tabs.[^status-tabs]
3. Yes.
4. No.
6. Yes.
7. No.

Of course, those were kind of trick questions. No one would ever have gotten
the first one right (what kind of answer is that even?). And the last two don't
make any sense either, because they are both single values.

[Go back up](#quiz)


# Endnotes

[^maybe]: Depends on whether your kernel has [this commit](
          https://github.com/torvalds/linux/commit/f5b65348fd77839b50e79bc0a5e536832ea52d8d).

[^status-tabs]: The [manpage](http://man7.org/linux/man-pages/man5/proc.5.html) uses spaces.

[^1]: [GNU/Linux], [FreeBSD], or [macOS].

[GNU/Linux]: https://github.com/acg/psmisc/blob/master/src/killall.c 

[FreeBSD]: http://sources.freebsd.org/RELENG_7/src/usr.bin/killall/killall.c

[macOS]: https://github.com/apple-open-source/macos/blob/master/shell_cmds/killall/killall.c

[^2]: Or cheat, I'm a sign, not a cop.

[^link-error]: This has actually happened to me before. But if you know where
               to look ([man execve](http://man7.org/linux/man-pages/man2/execve.2.html))
               it's at least documented.

[`clone`]: http://man7.org/linux/man-pages/man2/clone.2.html

[^wrapper]: I'm not a fan of syscalls without wrappers, as that encourages
            people to use raw syscalls, which then constrains libc
            implementors. clone is actually a good example: Bionic (Android's
            libc) has a cache for the TID of a thread. It needs to invalidate
            that cache upon calls to fork / clone / etc. It can only do this
            when people use the syscall wrapper. People don't always.
            
[^exhalation]: **Spoilers for Ted Chiang's story "Anxiety Is the Dizziness of
              Freedom"**. It's like the premise of that story. 
              (Assuming single-threaded) fork brings forth a new
              process that is the same, except for one int, which is used to
              make a choice. All other differences derive from that.

[^aspirational]: Some of the more low-level details leave a lot to be desired.
                 Many things are impossible in a standards compliant way, or
                 are underspecified, so people end up doing what works in
                 practice.


[^nobreak]: This, of course, leads to the question of what constitutes a
            feature, and what constitutes a bug. And what level of quality you
            expect from user-space. For instance, adding a new line to a proc
            file would break applications that do not properly look at key,
            value pairs, but rather hard-code the line they expect the value
            to be at. It's generally accepted that writing code like this is
            asking to be broken.

[billions]: https://twitter.com/Android/status/1125822326183014401

[^followup]: I do have opinions on more things, including but not limited to:
             `perf_event_open`, signals, `sendmsg`, session leaders,
             controlling terminals, `SIGPIPE`, ...
