# Using a Raspberry Pi as a TACACS Server
* [Installing TACACS+ on Raspberry Pi](#installing-tacacs--on-raspberry-pi)
* [Create a TACACS+ configuration file](#create-a-tacacs--configuration-file)
* [Configure a network device for TACACS](#configure-a-network-device-for-tacacs)
* [Setting `tac_plus` up as a systemd service](#setting--tac-plus--up-as-a-systemd-service)
* [Final Thoughts](#final-thoughts)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

An important part of becoming a network engineer is understanding and becoming comfortable with AAA services to ensure only the right people connect to and interact with your network.  TACACS is a commonly used device administration protocol for Authentication, Authorization, and Accounting.  

In an enterprise network, you'll likely see solutions like Cisco Identity Services Engine (ISE) used, however an ISE server can be a bit overkill for a basic networking lab.  Luckily Cisco provided the basic TACACS functionality in the form of a developer's kit at no cost.  And using it, the open source program tacacs+ (or `tac_plus`) was developed and is a great option for networking labs like ours.  

## Installing TACACS+ on Raspberry Pi 
TACACS+ (or tac_plus) is available for installation for many Linux distrubitons and platforms through package management tools like `apt` or `yum`.  And `tacacs+` was available for Raspberry Pi several years ago, however as of the writing of this doc it no longer is available for a simple `apt-get install tacacs+`.  

This means we will need to do a manual installation.  But don't fear, this isn't complicated and this guide will walk you through. 

> Note: This guide doesn't go into depth on installation from source concepts.

1. Install pre-reqs for compiling tacacs+

    ```bash
    sudo apt-get install bison flex libwrap0-dev 
    ```

1. Download tacacs+ source from [Shubbery Network](https://shrubbery.net/tac_plus/) download [site](https://shrubbery.net/pub/tac_plus/). 
    > The tacacs+ server is not updated regularly.  The "latest" version is 4.0.4.28 and is downloaded in this example. 

    ```bash
    # Create a "downloads" directory and move there
    mkdir ~/downloads
    cd ~/downloads 

    # Download the tacacs+ code 
    wget https://shrubbery.net/pub/tac_plus/tacacs-F4.0.4.28.tar.gz 

    # Untar the file and move into the directory 
    tar -xzf tacacs-F4.0.4.28.tar.gz
    cd tacacs-F4.0.4.28
    ```

1. Configure and install tacacs+

    ```bash
    ./configure 
    ```

    <details><summary>Output ./configure</summary>

    ```
    checking for a BSD-compatible install... /usr/bin/install -c
    checking whether build environment is sane... yes
    checking for a thread-safe mkdir -p... /usr/bin/mkdir -p
    checking for gawk... no
    checking for mawk... mawk
    checking whether make sets $(MAKE)... yes
    checking whether make supports nested variables... yes
    checking whether to enable maintainer-specific portions of Makefiles... no
    checking build system type... aarch64-unknown-linux-gnu
    checking host system type... aarch64-unknown-linux-gnu
    checking for gmake... /usr/bin/gmake
    checking whether /usr/bin/gmake sets $(MAKE)... yes
    checking whether to enable maintainer-specific portions of Makefiles... no
    checking how to print strings... printf
    checking for style of include used by /usr/bin/gmake... GNU
    checking for gcc... gcc
    checking whether the C compiler works... yes
    checking for C compiler default output file name... a.out
    checking for suffix of executables... 
    checking whether we are cross compiling... no
    checking for suffix of object files... o
    checking whether we are using the GNU C compiler... yes
    checking whether gcc accepts -g... yes
    checking for gcc option to accept ISO C89... none needed
    checking whether gcc understands -c and -o together... yes
    checking dependency style of gcc... gcc3
    checking for a sed that does not truncate output... /usr/bin/sed
    checking for grep that handles long lines and -e... /usr/bin/grep
    checking for egrep... /usr/bin/grep -E
    checking for fgrep... /usr/bin/grep -F
    checking for ld used by gcc... /usr/bin/ld
    checking if the linker (/usr/bin/ld) is GNU ld... yes
    checking for BSD- or MS-compatible name lister (nm)... /usr/bin/nm -B
    checking the name lister (/usr/bin/nm -B) interface... BSD nm
    checking whether ln -s works... yes
    checking the maximum length of command line arguments... 1572864
    checking whether the shell understands some XSI constructs... yes
    checking whether the shell understands "+="... yes
    checking how to convert aarch64-unknown-linux-gnu file names to aarch64-unknown-linux-gnu format... func_convert_file_noop
    checking how to convert aarch64-unknown-linux-gnu file names to toolchain format... func_convert_file_noop
    checking for /usr/bin/ld option to reload object files... -r
    checking for objdump... objdump
    checking how to recognize dependent libraries... pass_all
    checking for dlltool... no
    checking how to associate runtime and link libraries... printf %s\n
    checking for ar... ar
    checking for archiver @FILE support... @
    checking for strip... strip
    checking for ranlib... ranlib
    checking command to parse /usr/bin/nm -B output from gcc object... ok
    checking for sysroot... no
    checking for mt... mt
    checking if mt is a manifest tool... no
    checking how to run the C preprocessor... gcc -E
    checking for ANSI C header files... yes
    checking for sys/types.h... yes
    checking for sys/stat.h... yes
    checking for stdlib.h... yes
    checking for string.h... yes
    checking for memory.h... yes
    checking for strings.h... yes
    checking for inttypes.h... yes
    checking for stdint.h... yes
    checking for unistd.h... yes
    checking for dlfcn.h... yes
    checking for objdir... .libs
    checking if gcc supports -fno-rtti -fno-exceptions... no
    checking for gcc option to produce PIC... -fPIC -DPIC
    checking if gcc PIC flag -fPIC -DPIC works... yes
    checking if gcc static flag -static works... yes
    checking if gcc supports -c -o file.o... yes
    checking if gcc supports -c -o file.o... (cached) yes
    checking whether the gcc linker (/usr/bin/ld) supports shared libraries... yes
    checking whether -lc should be explicitly linked in... no
    checking dynamic linker characteristics... GNU/Linux ld.so
    checking how to hardcode library paths into programs... immediate
    checking for shl_load... no
    checking for shl_load in -ldld... no
    checking for dlopen... no
    checking for dlopen in -ldl... yes
    checking whether a program can dlopen itself... yes
    checking whether a statically linked program can dlopen itself... no
    checking whether stripping libraries is possible... yes
    checking if libtool supports shared libraries... yes
    checking whether to build shared libraries... yes
    checking whether to build static libraries... yes
    checking for the pthreads library -lpthreads... no
    checking whether pthreads work without any flags... no
    checking whether pthreads work with -Kthread... no
    checking whether pthreads work with -kthread... no
    checking for the pthreads library -llthread... no
    checking whether pthreads work with -pthread... yes
    checking for joinable pthread attribute... PTHREAD_CREATE_JOINABLE
    checking if more special flags are required for pthreads... no
    checking for gcc... (cached) gcc
    checking whether we are using the GNU C compiler... (cached) yes
    checking whether gcc accepts -g... (cached) yes
    checking for gcc option to accept ISO C89... (cached) none needed
    checking whether gcc understands -c and -o together... (cached) yes
    checking dependency style of gcc... (cached) gcc3
    checking how to run the C preprocessor... gcc -E
    checking for an ANSI C-conforming const... yes
    checking for inline... inline
    checking for preprocessor stringizing operator... yes
    checking for flex... flex
    checking lex output file root... lex.yy
    checking lex library... -lfl
    checking whether yytext is a pointer... yes
    checking for bison... bison -y
    checking whether yacc is bison in disguise... yes
    checking whether byte ordering is bigendian... no
    checking size of long int... 8
    checking whether to include symbols... no
    checking whether to set gcc warnings... no
    checking whether to use libwrap... yes
    checking whether to include skey support... no
    checking whether to include RSA SecurID support... no
    checking whether to setuid()... no
    checking whether to setgid()... no
    checking whether to include ACL support... yes
    checking whether to include user-enable support... yes
    checking whether to include maximum sessions (maxsess) support... no
    checking whether to include maxsess finger support... no
    checking whether to include ARAP DES support... no
    checking whether to include MSCHAP support... no
    checking whether to include MSCHAP DES support... no
    checking for alt pid file FQPN... /var/run/tac_plus.pid
    checking for alt accounting file FQPN... /var/log/tac_plus.acct
    checking for alt log file FQPN... /var/log/tac_plus.log
    checking for alt wholog file FQPN... /var/log/tacwho.log
    checking whether to profile... no
    checking for pam_start in -lpam... no
    checking for ANSI C header files... (cached) yes
    checking whether time.h and sys/time.h may both be included... yes
    checking crypt.h usability... yes
    checking crypt.h presence... yes
    checking for crypt.h... yes
    checking ctype.h usability... yes
    checking ctype.h presence... yes
    checking for ctype.h... yes
    checking errno.h usability... yes
    checking errno.h presence... yes
    checking for errno.h... yes
    checking fcntl.h usability... yes
    checking fcntl.h presence... yes
    checking for fcntl.h... yes
    checking malloc.h usability... yes
    checking malloc.h presence... yes
    checking for malloc.h... yes
    checking shadow.h usability... yes
    checking shadow.h presence... yes
    checking for shadow.h... yes
    checking for stdlib.h... (cached) yes
    checking for stdint.h... (cached) yes
    checking for string.h... (cached) yes
    checking for strings.h... (cached) yes
    checking sys/resource.h usability... yes
    checking sys/resource.h presence... yes
    checking for sys/resource.h... yes
    checking sys/socket.h usability... yes
    checking sys/socket.h presence... yes
    checking for sys/socket.h... yes
    checking for sys/types.h... (cached) yes
    checking sys/wait.h usability... yes
    checking sys/wait.h presence... yes
    checking for sys/wait.h... yes
    checking sysexits.h usability... yes
    checking sysexits.h presence... yes
    checking for sysexits.h... yes
    checking syslog.h usability... yes
    checking syslog.h presence... yes
    checking for syslog.h... yes
    checking termios.h usability... yes
    checking termios.h presence... yes
    checking for termios.h... yes
    checking for unistd.h... (cached) yes
    checking utmp.h usability... yes
    checking utmp.h presence... yes
    checking for utmp.h... yes
    checking utmpx.h usability... yes
    checking utmpx.h presence... yes
    checking for utmpx.h... yes
    checking wait.h usability... yes
    checking wait.h presence... yes
    checking for wait.h... yes
    checking return type of signal handlers... void
    checking for socklen_t... yes
    checking for pid_t... yes
    checking for getdtablesize... yes
    checking for memcpy... yes
    checking for memset... yes
    checking for random... yes
    checking for strchr... yes
    checking for strcspn... yes
    checking for strerror... yes
    checking for strrchr... yes
    checking for wait3... yes
    checking for wait4... yes
    checking for waitpid... yes
    checking whether setpgrp takes no argument... yes
    checking if waitpid takes a union wait... no
    checking if signals need to be re-armed... no
    checking if children need to be reaped... yes
    checking if children need to be reaped with SIG_IGN... yes
    checking for gnutar... no
    checking for gtar... no
    checking for tar... tar
    checking for perl5... no
    checking for perl... /usr/bin/perl
    checking that generated files are newer than configure... done
    configure: creating ./config.status
    config.status: creating Makefile
    config.status: creating version.h
    config.status: creating pathsl.h
    config.status: creating tac_plus.8
    config.status: creating tac_plus.conf.5
    config.status: creating config.h
    config.status: executing depfiles commands
    config.status: executing libtool commands
    ```

    </details>

    ```bash
    make install
    ```

    <details><summary>Output: make install</summary>

    ```
    /usr/bin/gmake  install-am
    gmake[1]: Entering directory '/home/pi/downloads/tacacs-F4.0.4.28'
    /bin/bash ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include -g -O2 -pthread    -g -O2 -pthread     -MT libtacacs_la-fdes.lo -MD -MP -MF .deps/libtacacs_la-fdes.Tpo -c -o libtacacs_la-fdes.lo `test -f 'fdes.c' || echo './'`fdes.c
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-fdes.lo -MD -MP -MF .deps/libtacacs_la-fdes.Tpo -c fdes.c  -fPIC -DPIC -o .libs/libtacacs_la-fdes.o
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-fdes.lo -MD -MP -MF .deps/libtacacs_la-fdes.Tpo -c fdes.c -o libtacacs_la-fdes.o >/dev/null 2>&1
    mv -f .deps/libtacacs_la-fdes.Tpo .deps/libtacacs_la-fdes.Plo
    /bin/bash ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include -g -O2 -pthread    -g -O2 -pthread     -MT libtacacs_la-maxsess.lo -MD -MP -MF .deps/libtacacs_la-maxsess.Tpo -c -o libtacacs_la-maxsess.lo `test -f 'maxsess.c' || echo './'`maxsess.c
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-maxsess.lo -MD -MP -MF .deps/libtacacs_la-maxsess.Tpo -c maxsess.c  -fPIC -DPIC -o .libs/libtacacs_la-maxsess.o
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-maxsess.lo -MD -MP -MF .deps/libtacacs_la-maxsess.Tpo -c maxsess.c -o libtacacs_la-maxsess.o >/dev/null 2>&1
    mv -f .deps/libtacacs_la-maxsess.Tpo .deps/libtacacs_la-maxsess.Plo
    /bin/bash ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include -g -O2 -pthread    -g -O2 -pthread     -MT libtacacs_la-md4.lo -MD -MP -MF .deps/libtacacs_la-md4.Tpo -c -o libtacacs_la-md4.lo `test -f 'md4.c' || echo './'`md4.c
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-md4.lo -MD -MP -MF .deps/libtacacs_la-md4.Tpo -c md4.c  -fPIC -DPIC -o .libs/libtacacs_la-md4.o
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-md4.lo -MD -MP -MF .deps/libtacacs_la-md4.Tpo -c md4.c -o libtacacs_la-md4.o >/dev/null 2>&1
    mv -f .deps/libtacacs_la-md4.Tpo .deps/libtacacs_la-md4.Plo
    /bin/bash ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include -g -O2 -pthread    -g -O2 -pthread     -MT libtacacs_la-md5.lo -MD -MP -MF .deps/libtacacs_la-md5.Tpo -c -o libtacacs_la-md5.lo `test -f 'md5.c' || echo './'`md5.c
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-md5.lo -MD -MP -MF .deps/libtacacs_la-md5.Tpo -c md5.c  -fPIC -DPIC -o .libs/libtacacs_la-md5.o
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-md5.lo -MD -MP -MF .deps/libtacacs_la-md5.Tpo -c md5.c -o libtacacs_la-md5.o >/dev/null 2>&1
    mv -f .deps/libtacacs_la-md5.Tpo .deps/libtacacs_la-md5.Plo
    /bin/bash ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include -g -O2 -pthread    -g -O2 -pthread     -MT libtacacs_la-packet.lo -MD -MP -MF .deps/libtacacs_la-packet.Tpo -c -o libtacacs_la-packet.lo `test -f 'packet.c' || echo './'`packet.c
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-packet.lo -MD -MP -MF .deps/libtacacs_la-packet.Tpo -c packet.c  -fPIC -DPIC -o .libs/libtacacs_la-packet.o
    libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/local/include -g -O2 -pthread -g -O2 -pthread -MT libtacacs_la-packet.lo -MD -MP -MF .deps/libtacacs_la-packet.Tpo -c packet.c -o libtacacs_la-packet.o >/dev/null 2>&1
    mv -f .deps/libtacacs_la-packet.Tpo .deps/libtacacs_la-packet.Plo
    /bin/bash ./libtool  --tag=CC   --mode=link gcc -g -O2 -pthread    -g -O2 -pthread     -version-info 1:0:0 -version-number 1:0:0 -L/usr/local/lib -L/lib   -L/usr/local/lib -L/lib -o libtacacs.la -rpath /usr/local/lib libtacacs_la-fdes.lo libtacacs_la-maxsess.lo libtacacs_la-md4.lo libtacacs_la-md5.lo libtacacs_la-packet.lo  -lnsl -lcrypt 
    libtool: link: gcc -shared  -fPIC -DPIC  .libs/libtacacs_la-fdes.o .libs/libtacacs_la-maxsess.o .libs/libtacacs_la-md4.o .libs/libtacacs_la-md5.o .libs/libtacacs_la-packet.o   -L/usr/local/lib -L/lib -lnsl -lcrypt  -O2 -pthread -O2 -pthread   -pthread -Wl,-soname -Wl,libtacacs.so.1 -o .libs/libtacacs.so.1.0.0
    libtool: link: (cd ".libs" && rm -f "libtacacs.so.1" && ln -s "libtacacs.so.1.0.0" "libtacacs.so.1")
    libtool: link: (cd ".libs" && rm -f "libtacacs.so" && ln -s "libtacacs.so.1.0.0" "libtacacs.so")
    libtool: link: ar cru .libs/libtacacs.a  libtacacs_la-fdes.o libtacacs_la-maxsess.o libtacacs_la-md4.o libtacacs_la-md5.o libtacacs_la-packet.o
    ar: `u' modifier ignored since `D' is the default (see `U')
    libtool: link: ranlib .libs/libtacacs.a
    libtool: link: ( cd ".libs" && rm -f "libtacacs.la" && ln -s "../libtacacs.la" "libtacacs.la" )
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT tac_pwd.o -MD -MP -MF .deps/tac_pwd.Tpo -c -o tac_pwd.o tac_pwd.c
    mv -f .deps/tac_pwd.Tpo .deps/tac_pwd.Po
    /bin/bash ./libtool  --tag=CC   --mode=link gcc  -g -O2 -pthread      -L/usr/local/lib -L/lib -o tac_pwd tac_pwd.o  -lnsl -lcrypt 
    libtool: link: gcc -g -O2 -pthread -o tac_pwd tac_pwd.o  -L/usr/local/lib -L/lib -lnsl -lcrypt -pthread
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT acct.o -MD -MP -MF .deps/acct.Tpo -c -o acct.o acct.c
    mv -f .deps/acct.Tpo .deps/acct.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT authen.o -MD -MP -MF .deps/authen.Tpo -c -o authen.o authen.c
    mv -f .deps/authen.Tpo .deps/authen.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT author.o -MD -MP -MF .deps/author.Tpo -c -o author.o author.c
    mv -f .deps/author.Tpo .deps/author.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT choose_authen.o -MD -MP -MF .deps/choose_authen.Tpo -c -o choose_authen.o choose_authen.c
    mv -f .deps/choose_authen.Tpo .deps/choose_authen.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT config.o -MD -MP -MF .deps/config.Tpo -c -o config.o config.c
    mv -f .deps/config.Tpo .deps/config.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT default_fn.o -MD -MP -MF .deps/default_fn.Tpo -c -o default_fn.o default_fn.c
    mv -f .deps/default_fn.Tpo .deps/default_fn.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT default_v0_fn.o -MD -MP -MF .deps/default_v0_fn.Tpo -c -o default_v0_fn.o default_v0_fn.c
    mv -f .deps/default_v0_fn.Tpo .deps/default_v0_fn.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT do_acct.o -MD -MP -MF .deps/do_acct.Tpo -c -o do_acct.o do_acct.c
    do_acct.c: In function ‘do_acct_syslog’:
    do_acct.c:184:6: warning: ‘strncat’ specified bound 4 equals source length [-Wstringop-overflow=]
    184 |      strncat(cmdbuf, "    ", 4);
        |      ^~~~~~~~~~~~~~~~~~~~~~~~~~
    mv -f .deps/do_acct.Tpo .deps/do_acct.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT do_author.o -MD -MP -MF .deps/do_author.Tpo -c -o do_author.o do_author.c
    mv -f .deps/do_author.Tpo .deps/do_author.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT dump.o -MD -MP -MF .deps/dump.Tpo -c -o dump.o dump.c
    mv -f .deps/dump.Tpo .deps/dump.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT enable.o -MD -MP -MF .deps/enable.Tpo -c -o enable.o enable.c
    mv -f .deps/enable.Tpo .deps/enable.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT encrypt.o -MD -MP -MF .deps/encrypt.Tpo -c -o encrypt.o encrypt.c
    mv -f .deps/encrypt.Tpo .deps/encrypt.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT expire.o -MD -MP -MF .deps/expire.Tpo -c -o expire.o expire.c
    mv -f .deps/expire.Tpo .deps/expire.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT hash.o -MD -MP -MF .deps/hash.Tpo -c -o hash.o hash.c
    mv -f .deps/hash.Tpo .deps/hash.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT maxsessint.o -MD -MP -MF .deps/maxsessint.Tpo -c -o maxsessint.o maxsessint.c
    mv -f .deps/maxsessint.Tpo .deps/maxsessint.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT parse.o -MD -MP -MF .deps/parse.Tpo -c -o parse.o parse.c
    mv -f .deps/parse.Tpo .deps/parse.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT programs.o -MD -MP -MF .deps/programs.Tpo -c -o programs.o programs.c
    mv -f .deps/programs.Tpo .deps/programs.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT pw.o -MD -MP -MF .deps/pw.Tpo -c -o pw.o pw.c
    mv -f .deps/pw.Tpo .deps/pw.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT pwlib.o -MD -MP -MF .deps/pwlib.Tpo -c -o pwlib.o pwlib.c
    mv -f .deps/pwlib.Tpo .deps/pwlib.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT report.o -MD -MP -MF .deps/report.Tpo -c -o report.o report.c
    mv -f .deps/report.Tpo .deps/report.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT sendauth.o -MD -MP -MF .deps/sendauth.Tpo -c -o sendauth.o sendauth.c
    mv -f .deps/sendauth.Tpo .deps/sendauth.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT sendpass.o -MD -MP -MF .deps/sendpass.Tpo -c -o sendpass.o sendpass.c
    mv -f .deps/sendpass.Tpo .deps/sendpass.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT tac_plus.o -MD -MP -MF .deps/tac_plus.Tpo -c -o tac_plus.o tac_plus.c
    mv -f .deps/tac_plus.Tpo .deps/tac_plus.Po
    gcc -DHAVE_CONFIG_H -I.   -I/usr/local/include  -g -O2 -pthread     -MT utils.o -MD -MP -MF .deps/utils.Tpo -c -o utils.o utils.c
    mv -f .deps/utils.Tpo .deps/utils.Po
    /bin/bash ./libtool  --tag=CC   --mode=link gcc  -g -O2 -pthread     -L. -L/usr/local/lib -L/lib -o tac_plus acct.o authen.o author.o choose_authen.o config.o default_fn.o default_v0_fn.o do_acct.o do_author.o dump.o enable.o encrypt.o expire.o hash.o maxsessint.o parse.o programs.o pw.o pwlib.o report.o sendauth.o sendpass.o tac_plus.o utils.o   -lwrap -ltacacs -lnsl -lcrypt 
    libtool: link: gcc -g -O2 -pthread -o .libs/tac_plus acct.o authen.o author.o choose_authen.o config.o default_fn.o default_v0_fn.o do_acct.o do_author.o dump.o enable.o encrypt.o expire.o hash.o maxsessint.o parse.o programs.o pw.o pwlib.o report.o sendauth.o sendpass.o tac_plus.o utils.o  -L. -L/usr/local/lib -L/lib -lwrap /home/pi/downloads/tacacs-F4.0.4.28/.libs/libtacacs.so -lnsl -lcrypt -pthread
    gmake[2]: Entering directory '/home/pi/downloads/tacacs-F4.0.4.28'
    /usr/bin/mkdir -p '/usr/local/lib'
    /bin/bash ./libtool   --mode=install /usr/bin/install -c   libtacacs.la '/usr/local/lib'
    libtool: install: /usr/bin/install -c .libs/libtacacs.so.1.0.0 /usr/local/lib/libtacacs.so.1.0.0
    libtool: install: (cd /usr/local/lib && { ln -s -f libtacacs.so.1.0.0 libtacacs.so.1 || { rm -f libtacacs.so.1 && ln -s libtacacs.so.1.0.0 libtacacs.so.1; }; })
    libtool: install: (cd /usr/local/lib && { ln -s -f libtacacs.so.1.0.0 libtacacs.so || { rm -f libtacacs.so && ln -s libtacacs.so.1.0.0 libtacacs.so; }; })
    libtool: install: /usr/bin/install -c .libs/libtacacs.lai /usr/local/lib/libtacacs.la
    libtool: install: /usr/bin/install -c .libs/libtacacs.a /usr/local/lib/libtacacs.a
    libtool: install: chmod 644 /usr/local/lib/libtacacs.a
    libtool: install: ranlib /usr/local/lib/libtacacs.a
    libtool: finish: PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/sbin" ldconfig -n /usr/local/lib
    ----------------------------------------------------------------------
    Libraries have been installed in:
    /usr/local/lib

    If you ever happen to want to link against installed libraries
    in a given directory, LIBDIR, you must either use libtool, and
    specify the full pathname of the library, or use the `-LLIBDIR'
    flag during linking and do at least one of the following:
    - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
        during execution
    - add LIBDIR to the `LD_RUN_PATH' environment variable
        during linking
    - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
    - have your system administrator add LIBDIR to `/etc/ld.so.conf'

    See any operating system documentation about shared libraries for
    more information, such as the ld(1) and ld.so(8) manual pages.
    ----------------------------------------------------------------------
    /usr/bin/mkdir -p '/usr/local/bin'
    /bin/bash ./libtool   --mode=install /usr/bin/install -c tac_pwd '/usr/local/bin'
    libtool: install: /usr/bin/install -c tac_pwd /usr/local/bin/tac_pwd
    /usr/bin/mkdir -p '/usr/local/sbin'
    /bin/bash ./libtool   --mode=install /usr/bin/install -c tac_plus '/usr/local/sbin'
    libtool: install: /usr/bin/install -c .libs/tac_plus /usr/local/sbin/tac_plus
    /usr/bin/mkdir -p '/usr/local/include'
    /usr/bin/install -c -m 644 tacacs.h '/usr/local/include'
    /usr/bin/mkdir -p '/usr/local/share/man/man5'
    /usr/bin/install -c -m 644 tac_plus.conf.5 '/usr/local/share/man/man5'
    /usr/bin/mkdir -p '/usr/local/share/man/man8'
    /usr/bin/install -c -m 644 tac_plus.8 tac_pwd.8 '/usr/local/share/man/man8'
    /usr/bin/mkdir -p '/usr/local/share/tacacs'
    /usr/bin/install -c -m 644 do_auth.py users_guide '/usr/local/share/tacacs'
    /usr/bin/mkdir -p '/usr/local/share/tacacs'
    /usr/bin/install -c tac_convert '/usr/local/share/tacacs'
    gmake[2]: Leaving directory '/home/pi/downloads/tacacs-F4.0.4.28'
    gmake[1]: Leaving directory '/home/pi/downloads/tacacs-F4.0.4.28'
    ```

    </details>

1. As a last step, rerun the dynamic library linker to pickup the new tacacs libraries.

    ```bash
    sudo ldconfig -v
    ```

## Create a TACACS+ configuration file
Before we can start our TACACS+ server (known from here on as `tac_plus` the name of the executable) we need to build the configuration file that will be used to receive and respond to all TACACS interactions from our network devices.  

> The configuration file shown here is a very basic one, that simply creates a single admin user.  For full details on configuring a robust TACACS policy, consult the manual page for `tac_plus` that was installed along with it. 
>
> ```bash
> man tac_plus
> man tac_plus.conf
> ```

1. Create the folder `/etc/tacacs` to hold all configuration details for `tac_plus`

    ```bash
    sudo mkdir /etc/tacacs
    ```

1. Name your configuration file `/etc/tacacs/tac_plus.conf` and start with this basic configuration. 

    ```bash
    # The TACASC shared-secret
    key = labtacacskey 

    # A full admin user called 'tacacsadmin'
    user = tacacsadmin {
        default service = permit 

        # Create a des encrypted password with the `tac_pwd` command  
        login = des Oy6FGC2LNE6ao

        # Set priv 15 for user 
        service = exec {
            priv-lvl = 15
        }
    }
    ```

1. Check the configuration file for errors by parsing. If everything is good, you should just see your configuration file printed back out to you. 

    ```bash
    tac_plus -C /etc/tacacs/tac_plus.conf -P
    ```

    <details><summary>Output from config parsing</summary>

    ```
    # The TACASC shared-secret
    key = labtacacskey 

    # A full admin user called 'tacacsadmin'
    user = tacacsadmin {
        default service = permit 

        # Create a des encrypted password with the `tac_pwd` command  
        login = des Oy6FGC2LNE6ao

        # Set priv 15 for user 
        service = exec {
            priv-lvl = 15
        }
    }
    ```

    </details>

1. Start-up `tac_plus` on the server.  In this command we provide the configuration file as well as enable debugging of all authorization and authentication messages to the log file `/var/log/tac_plus.log`.  This will make it easier to monitor status of TACACS interactions to the server. 

    ```bash
    sudo tac_plus -C /etc/tacacs/tac_plus.conf \
    -d 8 \
    -d 16 \
    -l /var/log/tac_plus.log 
    ```

    <details><summary>tac_plus debug options</summary>

    Here are the options available for debugging.  Just add another `-d` option with each type of debugging you want. 

    ```
    Value   Meaning
    2       configuration parsing debugging
    4       fork(1) debugging
    8       authorization debugging
    16      authentication debugging
    32      password file processing debugging
    64      accounting debugging
    128     config file parsing & lookup
    256     packet transmission/reception
    512     encryption/decryption
    1024    MD5 hash algorithm debugging
    2048    very low level encryption/decryption
    32768   max session debugging
    65536   lock debugging
    ```

    </details>

1. If/when you want to stop `tac_plus` when started like this, use `sudo killall tac_plus`

## Configure a network device for TACACS 
With our TACACS+ server up and operational, let's configure a newtork device to use it for authentication and authorization. 

> For this example I am using a Cisco Catalyst 3650 switch (WS-C3650-24TS-S).  

1. Before we setup TACACS authentication, let's make sure we keep the console without login in case something goes wrong. 

    ```
    aaa new-model
    !
    aaa authentication login CONSOLE none
    !
    line con 0
      login authentication CONSOLE
    ```

1. Next we'll configure the tacacs server, as well as the interface on our device to send the tacacs messages from. 
    > Configuring the source interface isn't strictly required, but I always like to be explicit

    ```
    ip tacacs source-interface Vlan192

    ! Providing the TACACS server address by DNS name is supported, 
    ! but IOS XE replaces the name with an IP before applying config
    tacacs server lab-server
      address ipv4 lab-server.lab.example
      key labtacacskey
    ```

1. We can test and verify that tacacs is working before configuring the switch to use it with this enable mode command. 

    ```
    ! The test command for user 'tacacsadmin' and password 'password'
    test aaa group tacacs+ tacacsadmin password legacy 

    ! Output
    Attempting authentication test to server-group tacacs+ using tacacs+
    User was successfully authenticated.
    ```

    > Success! 

1. If you tail the tac_plus.log on `lab-server` you should see the messages coming in. 

    ```bash
    tail -f /var/log/tac_plus.log

    # Ouput 
    Sun Feb 27 19:42:43 2022 [116628]: connect from 192.168.192.101 [192.168.192.101]
    Sun Feb 27 19:42:44 2022 [116628]: login query for 'tacacsadmin' port unknown-port from 192.168.192.101 accepted
    ```

1. Next up, let's enable the `aaa authentication` and `aaa authorization` on the switch. 

    ```
    aaa authentication login default group tacacs+ local
    aaa authorization exec default group tacacs+ local 
    ```

    > Because we configure `default`, we don't need to explicitly change set the `aaa group` on the `line` config

1. An attempt to ssh to the switch prompts for credentials.  Providing the TACACS username works. 

    ```
    ssh tacacsadmin@192.168.192.101
    Password: ************

    lab-switch#
    ```

1. And the log on `lab-server` shows the full AAA communication process including both the authentication and authorization happening. 

    ```
    Sun Feb 27 19:47:57 2022 [116906]: connect from 192.168.192.101 [192.168.192.101]
    Sun Feb 27 19:47:59 2022 [116906]: login query for 'tacacsadmin' port tty2 from 192.168.192.101 accepted
    Sun Feb 27 19:47:59 2022 [116907]: connect from 192.168.192.101 [192.168.192.101]
    Sun Feb 27 19:47:59 2022 [116907]: Start authorization request
    Sun Feb 27 19:47:59 2022 [116907]: do_author: user='tacacsadmin'
    Sun Feb 27 19:47:59 2022 [116907]: user 'tacacsadmin' found
    Sun Feb 27 19:47:59 2022 [116907]: exec authorization request for tacacsadmin
    Sun Feb 27 19:47:59 2022 [116907]: exec is explicitly permitted by line 12
    Sun Feb 27 19:47:59 2022 [116907]: nas:service=shell (passed thru)
    Sun Feb 27 19:47:59 2022 [116907]: nas:cmd* (passed thru)
    Sun Feb 27 19:47:59 2022 [116907]: nas:absent, server:priv-lvl=15 -> add priv-lvl=15 (k)
    Sun Feb 27 19:47:59 2022 [116907]: added 1 args
    Sun Feb 27 19:47:59 2022 [116907]: out_args[0] = service=shell input copy discarded
    Sun Feb 27 19:47:59 2022 [116907]: out_args[1] = cmd* input copy discarded
    Sun Feb 27 19:47:59 2022 [116907]: out_args[2] = priv-lvl=15 compacted to out_args[0]
    Sun Feb 27 19:47:59 2022 [116907]: 1 output args
    Sun Feb 27 19:47:59 2022 [116907]: authorization query for 'tacacsadmin' tty2 from 192.168.192.101 accepted
    ```

## Setting `tac_plus` up as a systemd service
Manually starting and stopping `tac_plus` isn't that complicated, but wouldn't it be great if it acted like the DNS and DHCP servers?  Starting with the RPi automatically, and using `systemctl` commands to manage it?  We can set that up by creating an systemd service file tac_plus.  

> A full discussion of systemd and services is beyond this guide. 

1. Create a file called `tac_plus.service` in the directory `/etc/systemd/system` with this contents.

    ```
    [Unit]
    Description=TACACS+ Server
    Documentation=man:tac_plus(8) man:tac_plus.conf(5)
    After=network.target

    [Service]
    Type=simple
    ExecStartPre=/usr/local/sbin/tac_plus -P -C /etc/tacacs/tac_plus.conf
    ExecStart=/usr/local/sbin/tac_plus -G -C /etc/tacacs/tac_plus.conf  -d 8 -d 16 -l /var/log/tac_plus.log
    ExecReload=/bin/sh -c "/usr/local/sbin/tac_plus -P -C /etc/tacacs/tac_plus.conf >/dev/null 2>&1" && /bin/kill -HUP $MAINPID
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```

1. After you create the file, make sure systemd takes note of the changes by reloading. 

    ```bash
    sudo systemctl daemon-reload
    ```

1. Before trying to start the service, make sure that there isn't an instance running already. 

    ```bash
    sudo killall tac_plus
    ```

1. Now startup the service. 

    ```bash
    sudo systemctl start tac_plus.service
    ```

    > You can check status of the service with `systemctl status tac_plus.service`

1. If you want to configure the new `tac_plus.service` to start automatically when the RPi starts. 

    ```bash
    sudo systemctl enable tac_plus
    ```

1. The log file is still created and maintained, but you can also use the journal to montior status.  

    ```bash
    journalctl -xef -u tac_plus
    ```

## Final Thoughts 
Now that we've got a TACACS server up and running in our lab, we can begin to explore all the ways TACACS can be used to secure access to our network devices.  You can create read-only users, custom command lists, and setup accounting for each command.  The details in the `tac_plus.conf` man page will show you how all this is setup, or a bit of searching online will find lots of examples from other engineers using `tac_plus` in their setups.  

And a bonus I learned after using `tac_plus` for awhile, I found I had a much better understanding of how TACACS worked than when I used one of the more polished commercial TACACS servers like Cisco ISE.  Not that I would want to give those up in the production networks.  