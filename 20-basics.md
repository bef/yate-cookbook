---
layout: page
title: Yate Basics
permalink: /basics/
---

This chapter will provide a basic understanding of how to install, configure and debug Yate. Please feel free to skip to the next chapter if your Yate installation is already up and running properly.

## Documentation

Where can I get more information?

* [http://docs.yate.ro/](http://docs.yate.ro/) - up-to-date documentation wiki with lots of articles
* [http://yate.null.ro/](http://yate.null.ro/) - slightly out-of-date documentation wiki, but still useful
* Source Code / RTFS - The source code is well structured C++ and very easy to read. Give it a try.
* [http://yate.null.ro/pmwiki/index.php?n=Main.MailList](http://yate.null.ro/pmwiki/index.php?n=Main.MailList) - the Yate mailing list and its archive

## Quick Installation

You may choose to install Yate from your operating system's package repository, e.g. on Debian Linux:

    # apt-get install yate

However, keep in mind that your OS may not include the latest Yate version, which can be problematic in regard to security and feature set. On the other hand, packages are easier to maintain.

### Install from Source

1. Download latest version from [http://yate.null.ro/](http://yate.null.ro/) and unpack the archive as usual, e.g.

        $ cd /usr/local/src
        $ wget http://yate.null.ro/tarballs/yate5/yate-XXXXX.tar.gz ## replace XXXXX with actual version number
        $ tar zxvf yate-XXXXX.tar.gz
        $ cd yate

    *Note:* If you are upgrading, be aware that the archive contains the directory `yate/`, which may already exist. --> `cp yate yate-xxx` first.

2. Install dependencies, e.g. on Debian/Ubuntu:

        # apt-get install build-essential zlib1g-dev libssl-dev libgsm1-dev pkg-config speex
        # apt-get install libmysqlclient-dev ## for mysql support if needed

3. Configure with flags, e.g. with custom install path, without Postgres support and with SSE2:

        $ ./configure --prefix=/home/poc/yate --without-libpq --enable-sse2

    Or simply run the default configure:

        $ ./configure

    Personally I like to keep this line and additional installation instructions and notes close to the actual installation in a file like `../COMPILE_YATE`. This way I know exactly how to re-compile or upgrade the software several months later for this particular installation.

4. Compile + Install

        $ make
        $ sudo make install

    Don't worry. There is a working `make uninstall`, which deletes all but configuration files.

After installation, files can be found in the usual places. Assuming an installation prefix of `/usr/local`:

* `/usr/local/etc/yate`: configuration files
* `/usr/local/bin/yate*`: executable files
* `/usr/local/share/yate/*`: support files: scripts, sounds, UI files
* `/usr/local/man/man8/yate*`: manual pages
* `/usr/local/share/doc/yate-*`: developer documentation
* `/usr/local/lib/yate/*`: yate modules
* `/usr/local/lib/libyate*`: shared libraries
* `/usr/local/include/yate`: include files
* depending on your configuration there may be other support files

### Upgrade from Source

1. Stop Yate, e.g. `killall yate`
2. `make uninstall` in the old Yate source directory: This will remove version specific libraries
3. `make install` in the new Yate source directory: This will install the new Yate version and not harm any existing configuration files.
4. Update configuration if needed, e.g. new modules.
5. Start Yate


### Configuration

Yate configuration files are like INI files with `[sections]` and `key=value` pairs. For syntax highlighting in VIM just type `:setf dosini` and `:syntax on` or extend your `.vimrc` like so:

```vim
" yate -> dosini syntax
au BufNewFile,BufRead */etc/yate/*.conf setf dosini

" syntax highlighting
syn on
```

Config files reside in `/usr/local/etc/yate/` by default. Let's start with `yate.conf`: It is a good idea only to load those modules actually needed for your setup. Otherwise it may lead to unwanted features or protocols being enabled, which in turn broadens the attack surface from a security point of view.

The following example shows a usable `yate.conf` restricted to load only a few modules on startup:

```INI
[general]
; modload: boolean: Should a module be loaded by default if there is no
;  reference to it in the [modules] section
modload=disable

;...

[modules]
dumbchan.yate=true
lateroute.yate=true
regfile.yate=true
accfile.yate=true
rmanager.yate=true
wavefile.yate=true
pbx.yate=true
tonegen.yate=true
callfork.yate=true
ysipchan.yate=true
extmodule.yate=true
yrtpchan.yate=true
openssl.yate=true
regexroute.yate=true
;ilbcwebrtc.yate=true
;cdrbuild.yate=true
;cdrfile.yate=true
;javascript.yate=true
;...
```

I like to add all available modules commented out to the [modules] section to be able to toggle modules with minimal effort. Try the following command:

    find /usr/local/lib/yate -name \*.yate |xargs -n 1 basename |xargs -n 1 -I '{}' echo ";{}=true"

As a rule: If you don't know the module, best leave it commented out. See also [the Yate docu Wiki](http://docs.yate.ro/wiki/Modules) for a list of modules.

*Next step:* Configure each module loaded on startup.


## Startup

Yate can run as unprivileged system user. I fact, creating a dedicated user and group just for running Yate is a security recommendation. So:

    # groupadd yate
    # useradd -g yate yate
    # mkdir /home/yate
    # chown yate:yate /home/yate
    # su - yate
    $


There are several ways to start Yate:

* For debugging right after compilation: There is a script in the source directory `./run` to start and debug Yate on Linux systems.

* For debugging and first setup after installation: Just type `yate` or `yate -v` or even `yate -Do -vv` (`-Do` means color output; `-vv` means 2x verbose).

    The following error may occur:

        yate: error while loading shared libraries: libyate.so.4.3.0: cannot open shared object file: No such file or directory

    If so, try `LD_LIBRARY_PATH=/usr/local/lib yate` or add the correct library path to your dynamic loader search path, e.g. `/etc/ld.so.conf` and run `ldconfig`. On MacOSX the dynamic linker uses the environment variable `DYLD_LIBRARY_PATH`.

    Also quite useful in this context is [SCREEN](http://www.gnu.org/software/screen/) or any other terminal multiplexer program.

* For production: The source package comes with a few sample init scripts:

        ./packing/deb/yate.init
        ./packing/portage/yate.init
        ./packing/rpm/yate.init


After successful startup you can see where Yate is listening for connections:

    $ sudo netstat -lntup |grep yate

On my test VM the output looks like this:

```
tcp    0    0 127.0.0.1:5060        0.0.0.0:*       LISTEN     28833/yate
tcp    0    0 172.16.229.129:5061   0.0.0.0:*       LISTEN     28833/yate
tcp    0    0 127.0.0.1:5038        0.0.0.0:*       LISTEN     28833/yate
tcp    0    0 127.0.0.1:5039        0.0.0.0:*       LISTEN     28833/yate
udp    0    0 172.16.229.129:5060   0.0.0.0:*                  28833/yate
udp    0    0 0.0.0.0:4569          0.0.0.0:*                  28833/yate
```

Port 5060 is for SIP. Port 2061 is for SIPS (SIP over TLS). Port 4569 is IAX. Port 5038 is the rmanager and port 5039 is an extmodule listener.


## Debugging

### Restarting Yate


* Full restart: `/etc/init.d/yate restart`
    or equivalently Ctrl-C, Cursor-UP, RETURN.

* Reload Yate: Send SIGQUIT to the Yate process:

        killall -SIGQUIT yate

    Or press `Ctrl-\` on the running foreground process - however this may detach shell scripts if Yate was started using a custom startup script.
    
    Or press `Ctrl-\` in rmanager.


### rmanager

The rmanager is a useful Yate console. A most common configuration is done like so in `rmanager.conf`:

```INI
[general]
; Each section creates a connection listener in the Remote Manager.
; An empty (all defaults) general section is assumed only in server mode if the
;  configuration file is missing.

; port: int: TCP Port to listen on, 0 to disable the listener
port=5038

; addr: ipaddress: IP address to bind to
addr=127.0.0.1

;...
; color: bool: Enable colorization debug as soon as connecting
;  This setting is ignored if telnet negotiation is disabled
color=yes
;...
```

Then, connect using telnet or netcat:

```
$ telnet localhost 5038
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
eventphone YATE 4.3.0-1 on devvm.
```

Try `help` and `status`. For more debugging output, try this (for verbose SIP debugging):

    debug on
    debug level 10
    debug sip level 10


Possible debug levels can be found in `yateclass.h` line 225++:


```
enum DebugLevel {
    DebugFail = 0,
    DebugTest = 1,
    DebugGoOn = 2,
    DebugConf = 3,
    DebugStub = 4,
    DebugWarn = 5,
    DebugMild = 6,
    DebugCall = 7,
    DebugNote = 8,
    DebugInfo = 9,
    DebugAll = 10
};
```


### IP level SIP debugging

Usual candidates would be

    # ngrep -l -W byline port 5060

    # tcpdump -ln -i eth0 -A port 5060



## Concepts

###Engine / Message Queue

The core of Yate - the engine - is a message queue. Each module can subscribe to a type of message and either process or reject the message. In addition, Yate modules can also observe the message queue without actually handling messages (watch), which is similar to a [PubSub pattern](https://en.wikipedia.org/wiki/Publish/subscribe).

When subscribing to a message type, the module must provide a priority. A message is then passed to each matching module in the order of subscribed priority.

Messages are human readable. Try this in rmanager:

```
machine on
Machine mode: on
%%<message::false:engine.timer::time=1361116546:nodename=devvm:handlers=register%z50,mysqldb%z90,openssl%z90,zlibcompress%z90,mux%z90,dumb%z90,conf%z90,stun%z90,iax%z90,analyzer%z90,tone%z90,users%z90,register%z90,monitoring%z90,tonedetect%z90,yrtp%z90,sip%z90,wave%z90,callfork%z90,pbx%z90,sip_cnam_lnp%z90,regfile%z100
...
^C
```

The format is as follows:

    %%<message:<id>:<processed>:[<name>]:<retvalue>[:<key>=<value>...]

The message dumped here is of type `engine.timer` and was sent from *Engine to Application*. There is no `id`. The message has not been processed by any module (processed=false). The return value is empty. At the end there are multiple key/value pairs, notably *handlers=...*, which lists all modules subscribed to this message type and their priority.

The complete message format documentation can be found [here](http://yate.null.ro/docs/extmodule.html) as well as in the source package under `docs/extmodule.html`.

A common message flow for a Call from a user to an IVR would look like this:

    call.preroute
    chan.startup
    call.route
    call.update
    call.execute
    chan.rtp status=created
    chan.connected
    ...
    chan.rtp status=terminated
    chan.hangup
    chan.disconnected


A list of standard messages can be found in the [documentation](http://docs.yate.ro/wiki/Standard_Messages). Custom modules may invent new message types and use the Yate engine to do IPC (inter-process communication).


### Extmodule

Participating in the Yate engine's message queue without writing another module in C++ (which would be simple enough) is done via the *extmodule* module. `extmodule.conf`:

```INI
[general]
scripts_dir=/usr/local/share/yate/scripts/
priority=100
timeout=10000
timebomb=true
;...

[listener tcp5039]
type=tcp
addr=127.0.0.1
port=5039
```
This configuration opens a TCP socket on port 5039. It is also possible to call scripts, that communicate via stdio, like so: `regexroute.conf`:

    ^77$=external/nodata/foo.tcl

This would execute the script `/usr/local/share/yate/scripts/foo.tcl` and interact using the extmodule protocol.

*Note*: Some examples in this document are written in Tcl using the [YGI library](https://github.com/bef/yate-tcl) to interface with Yate. Libraries in other languages - PHP, Python, Perl - are available as well. Yate also comes with its own JavaScript interpreter in its own module *javascript.yate*.

A simple use case for extmodule is YGI's message printer `dumpmsgs.tcl`:

```
---------> new message: user.register  true 
|             number 3333
|            sip_uri sip:devvm
|         sip_callid 2088059556@devvm
|           username 3333
|              realm devvm
|       ip_transport TLS
|            newcall false
|             domain devvm
|             device YATE/4.0.1
|             driver sip
|               data sip/sip:3333@172.16.229.1:62928
|            ip_host 172.16.229.1
|            ip_port 62928
|            expires 600
|             sip_to sip:3333@172.16.229.1:62928
|      connection_id tls:172.16.229.129:5061-172.16.229.1:62928
| connection_reliable true
|       route_params oconnection_id
|     oconnection_id tls:172.16.229.129:5061-172.16.229.1:62928
|           handlers monitoring:1,register:50
...
```

