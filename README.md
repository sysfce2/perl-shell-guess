# Shell::Guess

make an educated guess about the shell in use

# SYNOPSIS

guessing shell which called the Perl script:

    use Shell::Guess;
    my $shell = Shell::Guess->running_shell;
    if($shell->is_c) {
      print "setenv FOO bar\n";
    } elsif($shell->is_bourne) {
      print "export FOO=bar\n";
    }

guessing the current user's login shell:

    use Shell::Guess;
    my $shell = Shell::Guess->login_shell;
    print $shell->name, "\n";

guessing an arbitrary user's login shell:

    use Shell::Guess;
    my $shell = Shell::Guess->login_shell('bob');
    print $shell->name, "\n";

# DESCRIPTION

Shell::Guess makes a reasonably aggressive attempt to determine the 
shell being employed by the user, either the shell that executed the 
perl script directly (the "running" shell), or the users' login shell 
(the "login" shell).  It does this by a variety of means available to 
it, depending on the platform that it is running on.

- getpwent

    On UNIXy systems with getpwent, that can be used to determine the login
    shell.

- dscl

    Under Mac OS X getpwent will typically not provide any useful information,
    so the dscl command is used instead.

- proc file systems

    On UNIXy systems with a proc filesystems (such as Linux), Shell::Guess 
    will attempt to use that to determine the running shell.

- ps

    On UNIXy systems without a proc filesystem, Shell::Guess will use the
    ps command to determine the running shell.

- [Win32::Process::Info](https://metacpan.org/pod/Win32::Process::Info) and [Win32::Process::List](https://metacpan.org/pod/Win32::Process::List)

    On Windows if these modules are installed they will be used to determine
    the running shell.  This method can differentiate between PowerShell,
    `command.com` and `cmd.exe`.

- ComSpec

    If the above method is inconclusive, the ComSpec environment variable
    will be consulted to differentiate between `command.com` or `cmd.exe`
    (PowerShell cannot be detected in this manner).

- reasonable defaults

    If the running or login shell cannot be otherwise determined, a reasonable
    default for your platform will be used as a fallback.  Under OpenVMS this is
    dcl, Windows 95/98 and MS-DOS this is command.com and Windows NT/2000/XP/Vista/7
    this is cmd.exe.  UNIXy platforms fallback to bourne shell.

The intended use of this module is to enable a Perl developer to write 
a script that generates shell configurations for the calling shell so they
can be imported back into the calling shell using `eval` and backticks
or `source`.  For example, if your script looks like this:

    #!/usr/bin/perl
    use Shell::Guess;
    my $shell = Shell::Guess->running_shell;
    if($shell->is_bourne) {
      print "export FOO=bar\n";
    } else($shell->is_c) {
      print "setenv FOO bar\n";
    } else {
      die "I don't support ", $shell->name, " shell";
    }

You can then import FOO into your bash or c shell like this:

    % eval `perl script.pl`

or, you can write the output to a configuration file and source it:

    % perl script.pl > foo.sh
    % source foo.sh

[Shell::Config::Generate](https://metacpan.org/pod/Shell::Config::Generate) provides a portable interface for generating
such shell configurations, and is designed to work with this module.

# CLASS METHODS

These class methods return an instance of Shell::Guess, which can then be 
interrogated by the instance methods in the next section below.

## Shell::Guess->running\_shell

Returns an instance of Shell::Guess based on the shell which directly
started the current Perl script.  If the running shell cannot be determined,
it will return the login shell.

## Shell::Guess->login\_shell( \[ $username \] )

Returns an instance of Shell::Guess for the given user.  If no username is specified then
the current user will be used.  If no shell can be guessed then a reasonable fallback
will be chosen based on your platform.

## Shell::Guess->bash\_shell

Returns an instance of Shell::Guess for bash.

The following instance methods will return:

- $shell->name = bash
- $shell->is\_bash = 1
- $shell->is\_bourne = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/bash

All other instance methods will return false

## Shell::Guess->bourne\_shell

Returns an instance of Shell::Guess for the bourne shell.

The following instance methods will return:

- $shell->name = bourne
- $shell->is\_bourne = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/sh

All other instance methods will return false

## Shell::Guess->c\_shell

Returns an instance of Shell::Guess for c shell.

The following instance methods will return:

- $shell->name = c
- $shell->is\_c = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/csh

All other instance methods will return false

## Shell::Guess->cmd\_shell

Returns an instance of Shell::Guess for the Windows NT cmd shell (cmd.exe).

The following instance methods will return:

- $shell->name = cmd
- $shell->is\_cmd = 1
- $shell->is\_win32 = 1
- $shell->default\_location = C:\\Windows\\system32\\cmd.exe

All other instance methods will return false

## Shell::Guess->command\_shell

Returns an instance of Shell::Guess for the Windows 95 command shell (command.com).

The following instance methods will return:

- $shell->name = command
- $shell->is\_command = 1
- $shell->is\_win32 = 1
- $shell->default\_location = C:\\Windows\\system32\\command.com

All other instance methods will return false

## Shell::Guess->dcl\_shell

Returns an instance of Shell::Guess for the OpenVMS dcl shell.

The following instance methods will return:

- $shell->name = dcl
- $shell->is\_dcl = 1
- $shell->is\_vms = 1

All other instance methods will return false

## Shell::Guess->fish\_shell

Returns an instance of Shell::Guess for the fish shell.

The following instance methods will return:

- $shell->name = fish
- $shell->is\_fish = 1
- $shell->is\_unix = 1

## Shell::Guess->korn\_shell

Returns an instance of Shell::Guess for the korn shell.

The following instance methods will return:

- $shell->name = korn
- $shell->is\_korn = 1
- $shell->is\_bourne = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/ksh

All other instance methods will return false

## Shell::Guess->power\_shell

Returns an instance of Shell::Guess for Windows PowerShell.

The following instance methods will return:

- $shell->name = power
- $shell->is\_power = 1
- $shell->is\_win32 = 1

All other instance methods will return false

## Shell::Guess->tc\_shell

Returns an instance of Shell::Guess for tcsh.

The following instance methods will return:

- $shell->name = tc
- $shell->is\_tc = 1
- $shell->is\_c = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/tcsh

All other instance methods will return false

## Shell::Guess->z\_shell

Returns an instance of Shell::Guess for zsh.

The following instance methods will return:

- $shell->name = z
- $shell->is\_z = 1
- $shell->is\_bourne = 1
- $shell->is\_unix = 1
- $shell->default\_location = /bin/zsh

All other instance methods will return false

# INSTANCE METHODS

The normal way to call these is by calling them on the result of either
_running\_shell_ or _login\_shell_, but they can also be called as class
methods, in which case the currently running shell will be used, so

    Shell::Guess->is_bourne

is the same as

    Shell::Guess->running_shell->is_bourne

## $shell->is\_bash

Returns true if the shell is bash.

## $shell->is\_bourne

Returns true if the shell is the bourne shell, or a shell which supports bourne syntax (e.g. bash or korn).

## $shell->is\_c

Returns true if the shell is csh, or a shell which supports csh syntax (e.g. tcsh).

## $shell->is\_cmd

Returns true if the shell is the Windows command.com shell.

## $shell->is\_command

Returns true if the shell is the Windows cmd.com shell.

## $shell->is\_dcl

Returns true if the shell is the OpenVMS dcl shell.

## $shell->is\_fish

Returns true if the shell is Fish shell.

## $shell->is\_korn

Returns true if the shell is the korn shell.

## $shell->is\_power

Returns true if the shell is Windows PowerShell.

## $shell->is\_tc

Returns true if the shell is tcsh.

## $shell->is\_unix

Returns true if the shell is traditionally a UNIX shell (e.g. bourne, bash, korn)

## $shell->is\_vms

Returns true if the shell is traditionally an OpenVMS shell (e.g. dcl)

## $shell->is\_win32

Returns true if the shell is traditionally a Windows shell (command.com, cmd.exe)

## $shell->is\_z

Returns true if the shell is zsh

## $shell->name

Returns the name of the shell.

## $shell->default\_location

The usual location for this shell, for example /bin/sh for bourne shell
and /bin/csh for c shell.  May not be defined for all shells.

# CAVEATS

Shell::Guess shouldn't ever die or crash, instead it will attempt to make a guess or use a fallback 
about either the login or running shell even on unsupported operating systems.  The fallback is the 
most common shell on the particular platform that you are using, so on UNIXy platforms the fallback 
is bourne, and on OpenVMS the fallback is dcl.

These are the operating systems that have been tested in development and are most likely to guess
reliably.

- Linux
- Cygwin
- FreeBSD
- Mac OS X
- Windows (Strawberry Perl)
- MS-DOS (djgpp)
- OpenVMS

    Always detected as dcl (a more nuanced view of OpenVMS is probably possible, patches welcome).

UNIXy platforms without a proc filesystem will use [Unix::Process](https://metacpan.org/pod/Unix::Process) if installed, which will execute 
ps to determine the running shell.

It is pretty easy to fool the ->running\_shell method by using fork, or if your Perl script
is not otherwise being directly executed by the shell.

Patches are welcome to make other platforms work more reliably.

# AUTHOR

Graham Ollis <plicease@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2012 by Graham Ollis.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
