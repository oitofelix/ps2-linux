---
title: oitofelix - PlayStation 2 Linux
description:

tags: article, free software, GPG, SSH, PGP, cryptography
license: CC BY-SA 4.0
layout: oitofelix-homepage
base: http://oitofelix.github.io
base_local: http://localhost:4000
---
<div id="markdown" markdown="1">
## PlayStation 2 Linux

Between June and July 2012 I worked with the _PlayStation 2 Linux
developers_ of the [PSX Scene](http://psx-scene.com/forums/ps2-linux/)
community, collaborating and sharing experiences, in order to improve
the port of _Linux 2.6_ to the video game console _Sony PlayStation
2_.  I improved the _PS1 memory card support_, wrote _SBIOS_ and
_Linux_ Multi-tap pad drivers, and upgraded _kernelloader_ build
system's `png2rgb` image conversion tool to use _libpng_'s 1.5 _API_.
[_Mega Man_](http://psx-scene.com/forums/members/mega-man/) was the
leader developer of the porting project at that time.

__Table of contents__

1. [Initial experience report](#initial-experience-report)
2. [XDMCP tutorial](#xdmcp-tutorial)
3. [Development kernel report](#development-kernel-report)
4. [PS1 memory card write support patch](#ps1-memory-card-write-support-patch)
4. [PS1 memory card block usage reporting patch](#ps1-memory-card-block-usage-reporting-patch)
5. [`png2rgb` patch](#png2rgb-patch)
6. [The SBIOS size problem](#the-sbios-size-problem)
7. [SBIOS Multi-tap pad driver](#original-sbios-multi-tap-pad-driver)
8. [Linux Multi-tap pad driver](#linux-multi-tap-pad-driver)



### Initial experience report

Hello folks,

Congratulations on this amazing project --- I really appreciate it!
Here are some things I've experimented with so far:

1. Downloaded `kloader-2.6`, `linux-2.6_v8` kernel and _Debian 5.0
   mipsel port_ and installed them on my _PS2_ (model `SCPH-90001`);
2. Set up the _root file system_ and _swap_ over _NFS_;
3. Locally accessed the system using a _NTSC television_ as screen;
3. Fetched some packages using `apt-get`;
4. Mounted via _NFS_ my laptop's home directory within the _PS2_ for a
   very convenient way to share my personal data and settings;
5. Accessed my _PS2_ via RSH<sup>1</sup> and XDMCP<sup>2</sup>;

I'm very impressed by system's stability and usability --- things are
a little bit slow but the system is fairly usable.  I'm excited about
the project and I want to help!  I'll start by installing the
toolchain in order to compile the kernel's development version that
has the memory card support that I'm looking forward to see working.

Thank you _Mega Man_, you made me very happy!


__Footnotes__

1. I don't want the network overhead of SSH encryption.
2. In order to run X-based applications remotely, freeing the PS2 from
   the additional overhead of a local X server instance.





### XDMCP tutorial

Yes, I got my _PS2_ graphical desktop into notebook's screen.
Obviously the X server loads almost instantaneously because it's
running in my laptop.  What is somewhat slow is the execution of _X
clients_, because those ran in the _PS2_.  However, I can say for sure
that there is a significant performance boost by this remote access
method.  One thing I want to highlight is that my setup overload the
_PS2_'s network interface --- I think its limit speed is 1 Mb/s ---
because all file and swap memory operations go through it by _NFS_.  A
_fat PS2_ where the system and swap partitions are located within the
internal _HDD_ may achieve greater performance.  However, I cannot say
in advance if a _PS2_ running _GNU/Linux_ from an _USB mass storage
device_ is faster.

On my _PS2_ I'm running IceWM, which isnt't a full featured desktop
like XFCE --- rather it's just a nice window manager with taskbar and
the Win9x/(OS/2) look&feel.  I don't use heavy login managers like
_GDM_ or _KDM_; for _XDMCP_ session management the best choice seems
to be _XDM_.  Taking into account these considerations, here is the
time report that you requested: it takes around 5 seconds from the
moment that I start X server in the laptop to the moment _XDM_ shows
up with its login window.  After the login, it takes about 30 seconds
for _IceWM_ to load and be ready for use.

In the following lines I briefly describe how to make a working _XDMCP
session_ to remotely access your PS2.

1. First we set up the _PS2's X Display Manager_.  Two files require
   editing but one is optional.

   - __REQUIRED__ `/etc/x11/xdm/xdm-config`: this is the main(/meta)
     _XDM configuration file_.  In it you have to comment the
     following resource line:

     <pre>DisplayManager.requestPort: 0</pre>

     with an `!` (exclamation mark) so it looks like:

     <pre>! DisplayManager.requestPort: 0</pre>

     Often this is the last line of the resource file.  It tells _XDM_
     whether to enable remote _XDMCP session management_.

   - __REQUIRED__ `/etc/x11/xdm/Xaccess`: this file list the hosts
     from which the X server is allowed to request a _XDMCP
     sessision_.  You have to append to it the _IP_ of the _X server
     host_.  If the host and your PS2 are linked trough a trusted
     network, in place you can use just a `*` (asterisk) in order to
     match any host.  You can also use host names if you have set up
     the file `/etc/hosts` properly in both machines.

   - __OPTIONAL__ `/etc/x11/xdm/Xservers`: This file list the remote
     _X server displays_ that don't support the _XDMCP protocol_ and
     the local ones which are able to connect to _XDM_.  If you don't
     want to start a PS2 local X server automatically at start-up,
     comment the line that reads (it's usually the last line)

     <pre>:0 local /usr/bin/X :0 vt7 -nolisten tcp</pre>

     so it reads

     <pre># :0 local /usr/bin/X :0 vt7 -nolisten tcp</pre>

	 Of course, in this case the `#` (hash mark) indicates a comment
     rather than a root prompt.

2. The next step is to tell XDM to reread its configuration files:

   <pre># /etc/init.d/xdm reload</pre>

3. The last step is to run the _X server_ in _query mode_.  For that
   end type the following on the host computer:

   <pre># X :DISPLAY_NUMBER -query PS2_IP</pre>

   Where _DISPLAY_NUMBER_ are PS2_IP are meta-variables whose value
   depends on your setup.  Here you also can use the PS2 host name in
   place of its IP in case you have set up the `/etc/hosts` file.

If everything went well you will see in your computer display a _XDM
login window_.  Remember that _XDM_ usually runs the shell script
`~/.xsession` in order set up the user's environment, so the last
executable line of this file should look like:

<pre>exec MY_WINDOW_MANAGER</pre>

In my case MY_WINDOW_MANAGER is `icewm-session`.  If that file isn't
proper configured, nothing seems to happen and the login screen is
showed again after each login attempt.

If the wind has blown in the right direction, now you have a faster X
environment --- even better than mine if you have a fat _PS2_.

__Good luck!__





### Development kernel report

I've fetched the latest kernel development version (13/06/2012), then
built and used it.  Here is my report.


- __Framebuffer support__: the PS2 Linux framebuffer driver is great.
  I admire a lot the hack that has been made by _Mega Man_.
  Nevertheless, there is an annoying problem that I've noticed: if you
  have an application that mix the graphical and textual use of the
  _framebuffer_, the text disappears.  For instance take `w3m` with
  _framebuffer_ image support.  When you execute it, everything is
  fine until the first image loads; after that only images are visible
  and the text flicks in the background for each key press.  I hope
  there is a solution for this.
- __Memory Card Support__: the memory card support is a big step and
  it's working very well --- nice work, _wisi_ and _Mega Man_!
  However I noticed the following issues:

  - PS1 memory card reading is properly handled, but writing isn't.
    It's possible to remove files but it's impossible to create any;
    all attempts results in a `ps2mclib_SetDir() failed` error.  It's
    also impossible to create directories, but I think that's a _PS1
    memory card file system_'s limitation.

  - PS1 memory cards, unlikely PS2 memory cards, aren't listed by `df`
    command.

  PS2 memory card support is working nice, except for moving a file
  within a single memory card --- which raises an error.  For
  instance, it is __not__ possible to rename a file.

  A memory card driver's remarkable property is the ability to mount
  memory card devices even when there isn't memory card in the
  respective slot, without raising an error.  After a memory card is
  mounted, one can remove and insert memory cards freely without
  worrying to dismount and remount --- everything works automagically
  --- just don't forget to do a `sync` before removing a memory card
  that you have written to.


- __Sound support__: When I first heard my PS2 playing via OSS, it was
  amazing!  As noted in _kloader_'s `README` file, I needed two RTE
  modules: `libsd.irx` and `sdrdrv.irx`.

  `mplayer` played _WAV_ and _FLAC_ files without any problems.  But
  it didn't play _MP3_ nor _OGG_.  An _OGG_ file apparently have
  started to play but I only heard a tick and it hanged up.  A _MP3_
  file took more than one hour in `mplayer`'s `open_codec` function
  until I killed it.  Also, I have not been able to play midi with
  _Timidity+_; it seems to hang up like `mplayer` with _MP3_ files.


- __Miscelaneous kernel modules__: Furthermore, I have played a little
  bit with some kernel modules that aren't pre-compiled or
  pre-configured to build in the vanilla development version ---
  modules like `option` and `sr_mod`.

  I have been able to use an external DVD Reader (with adittional
  external power supply) and a 3G modem.  By the way, I'm posting this
  using my _PS2_, which has a 3G modem attached to it.


I'm very happy with the results so far.  It's been fun and I've
acquired valuable knowledge.

_Mega Man_, if there is something with which I could help, just say.

__Happy PS2 GNU/Linux hacking!__

> Your tests are interesting.  I assume that the PS1 memory cards have
> the same problems in Linux 2.4.  I never tested PS1 memory cards.
> It is good to know what is working and what is not working.  First I
> want to get the CDVD driver working and I want to clean up all
> patches, so it is easier to put it into higher kernel versions.





### PS1 memory card write support patch

I got _PS1 memory card_ writing working.  Now it's possible to create
and copy files to _PS1 memory cards_.  I've made numerous tests and
_Linux_, _ULaunchELF_ and _PS2 native browser_ successfully read the
written files.  [The patch is attached](ps1-mc-write-support.patch).

> Do you know what the error code `-4` from `mcSetFileInfo()` means?
> I found a code position where `-4` is mapped to `-ENOENT` which
> means "no such file or directory".  Can you say what is happening?

According to the file `TGE/sbios/mc.c`, function `sbcall_mcgetdir`,
line 801, `-4` is mapped to "dirname error".  I think that's due to
_PS1 memory card_'s lack of support for directories.  I can't say for
sure --- maybe only Sony knows.  Nevertheless, apparently that error
code may be safely ignored in this case.

> Thanks for the patch.  I added it to the CVS.





### PS1 memory card block usage reporting patch

I also managed to make the memory card driver report block usage for
PS1 memory cards.  Now `df` is listing PS1 memory cards properly.
[The patch is attached](ps1-mc-block-usage-report.patch).

> Thanks for the patch.  I added it to the CVS.





### `png2rgb` patch

I was building _Kernelloader_ from CVS when I noticed that `png2rgb`
doesn't build against the fairly recent `libpng` version `1.5.10`.  So
I've adapted it to the new API and built and tested it with
`kernelloader` standard image files and some custom images ---
everything is working fine.
[The patch is attached](png2rgb-libpng-1.5.10-api.patch).

> I will check the png2rgb patch on my "old" kubuntu which was
> released last month.




### The SBIOS size problem

The development version of the _Kernelloader_ failed to load _Linux_ with
the following message:

<pre>
Checking SBIOS...
Error Message:
The memory region end address 0x00010280 of SBIOS is after 0x00010000.
</pre>

Note that with the 2.6 version and the same configuration file it
works as expected.

The [attached patch](kloader-check-sbios-end-address.patch) corrects
the issue.  However I don't know enough about Kernelloader code to
ensure that it's reliable.

_Mega Man_, do you know where the problem is?

> The size problem of the _SBIOS_ on slim PSTwos was already fixed by
> optimizing for size:
> [Diff of /kernelloader/TGE/sbios/Makefile](http://kernelloader.cvs.sourceforge.net/viewvc/kernelloader/kernelloader/TGE/sbios/Makefile?r1=1.12&r2=1.13).
> Your patch can make it unstable, because the Linux kernel uses the
> memory at `0x10000`. The _SBIOS_ must fit into the area `0x1000` to
> `0x10000`.  Over time the _SBIOS_ was extended and got larger than
> this area.  Now the size is checked in the linker script when
> linking _SBIOS_. It will not build if the _SBIOS_ is too large.





### SBIOS Multi-tap pad driver

I've written a Multi-tap pad driver for PS2 Linux. I've had to write
the _SBIOS RPC interface_ and the _Linux interface_ both from scratch.
I didn't create a _Linux module_ by itself, instead I put the
Multi-tap handling into the _ps2pad module_, since the pad and
multi-tap are inherently correlated.  I tested it with the _free IRX
Multitap_ module _freemtap_.  It's working very nicely!  The joystick
driver interface is interacting properly as well.  Now it's possible
to use up to 8 controllers with your PS2 GNU/Linux system.  It's
interesting to note that _RTE_ and _Sony's PS2 Linux_ doesn't have
such support.

However, when I first built the _TGE SBIOS_ with the new Multi-tap
interface, I had to optimize it for size because with `-O2`
optimization flag it didn't build due to the lack of space.
Nevertheless, I did it and successfully made everything work. After
that I cleaned my code in order to make it easy to read and more
robust --- with safer error handling.  Can you guess what happened
then?  Yeah!? :-( The binary object file increased sightly and it
didn't link: `ld` is claiming full memory use in section `.bss`.  If
one edits the link script, to enlarge the memory space, it's possible
to build, but `Kloader` refuses to load the _SBIOS_ with a well know
message of another post.  It's possible to force it to load the large
_SBIOS_ file, but it'll get corrupted by Linux that would most likely
intersect its end.

The clear fact is that we need more space to _SBIOS_.  I'm also
planning to write a multitap memory card driver and a remote control
driver.  Such things aren't present in the _Sony's RTE_, so we must
not be constrained by their design.  We need more space because our
_TGE_ is becoming by far more functional.  I also noticed that much
code in _TGE_ need to be revised and improved to make sure that it's
consistent with the _RPC specification_ and that might mean a larger
binary.  We need at least `128 KiB` available to _SBIOS_.

So, _Mega Man_, what do you think?

> I want that it stays compatible.  The last time I tested the kernel
> with the _RTE loader_ from _Sony_, it was working.  This is possible
> when keeping the limited memory range and putting the additonal code
> into the kernel.  There is much more memory available. This way it
> will also work with _Sony's Linux Toolkit_.  It is also possible to
> load new _IRX modules_ via the _Linux kernel_ (_Linux firmware
> interface_). There can be also an auto-detection of the module
> version like `/kernelloader/TGE/sbios/mc.c` (see
> `smod_get_mod_by_name()`). An example for the different interface is
> the smaprpc driver:
> `/linux/linux-2.6.35.4-mipsel-ps2/drivers/ps2/smaprpc.c`.  It uses
> the `ps2sif_bindrpc` and `ps2sif_callrpc` for communication.  This
> should be enough for most drivers.  It would be good if the Linux
> interface would stay compatible and you just extend it, for example:
> The joystick driver should still work, the `/proc/ps2pad` and the
> name of the device nodes and the _IOCTLs_ on it.  For testing
> purpose it is possible to extend the memory region, but please don't
> use it for the final version.
>
> This is defined in the following files:
>
> <pre>
> linux-2.6.35.4-mipsel-ps2/arch/mips/Makefile:669:load-$(CONFIG_SONY_PS2)        += 0x80010000
> linux-2.6.35.4-mipsel-ps2/arch/mips/ps2/setup.c:156:    add_memory_region(0x00010000, 0x01ff0000, BOOT_MEM_RAM);
> kernelloader/loader/loader.c:1946:                   if (check_sections("SBIOS", sbios, sbios_size, 0x1000, 0x10000, NULL) != 0) {
> kernelloader/loader/loader.c:1979:           if (sbios_size < (0x10000 - ((int) strlen(ps2_console_type)) - 1)) {
> kernelloader/loader/loader.c:2021:           if (check_sections("kernel", buffer, kernel_size, 0x10000, lowestAddress, &highest) != 0) {
> kernelloader/TGE/sbios/linkfile:6:   mem(RWX) : ORIGIN = 0x80001000, LENGTH = 0xF000
> </pre>
>
> The `F000` and the `10000` needs to be increased.  The string for
> storing the PS2 model number is stored behind _SBIOS_, but before
> `0x10000`.  There should be an additional space of 32 Byte between
> the end address and the `0x10000`.  I will make the size in the file
> `kernelloader/TGE/sbios/linkfile` smaller for this reason.





### Linux Multi-tap pad driver

A few days ago I moved the _Multi-tap pad SBIOS driver_ into the
kernel as _Mega Man_ have suggested.  Unfortunately, since then I have
fought to make it work with no success.

All goes fine when one initializes the _RPC servers_, like

{% highlight c %}
ps2sif_bindrpc (&cdataPortOpen, MTAPSERVER_PORT_OPEN, SIF_RPCM_NOWAIT,
                ps2mtap_rpcend_notify, (void *) &compl);
{% endhighlight %}

but when I do my first call to open a port like this

{% highlight c %}
ps2sif_callrpc (&cdataPortOpen, RPC_NUMBER, SIF_RPCM_NOWAIT,
                (void *) ps2mtapRpcBuffer, SEND_SIZE,
                (void *) ps2mtapRpcBuffer, RECEIVE_SIZE,
                (ps2sif_endfunc_t) ps2mtap_rpcend_notify,
                (void *) &compl);
{% endhighlight %}

the function return `0` (success) but there isn't a reply in
`ps2mtapRpcBuffer[REPLY]` and the port actually doesn't open.  I've
carefully read all function definitions involved in that call and I'm
pretty sure that it was proceeding like before with the _SBIOS native
interface_.  I have read the _smap kernel driver_ also and I can't
figure out what I'm doing wrong.  Maybe there is a kernel side
procedure to initialize a memory region (`ps2mtapRpcBuffer`) to
communicate with the _IRX module_, or something like that, but I can't
find it on the _smap driver_.

I know that it is very difficult for you to help me since you don't
have the source code at hand, but maybe I'm missing something so
trivial that you can help me.  If you need some additional information
I will be happy to provide it.  Any clues?

> `ps2mtapRpcBuffer` needs to be at least 16 Byte aligned.

If something is 64 bytes aligned then it's 16 bytes aligned, right?
I've declared `ps2mtapRpcBuffer` like this:

{% highlight c %}
#define ALIGNMENT 64
static u32 ps2mtapRpcBuffer[RPC_BUFFER_SIZE] __attribute__((aligned (ALIGNMENT)));
{% endhighlight %}


> You need to use the functions including "complete" in the function
> name like in the example driver.

Being `compl` defined as

{% highlight c %}
struct completion compl;
{% endhighlight %}

Like in the example device driver, for each `ps2sif_callrpc` call I
call the completion functions in the following order:

{% highlight c %}
init_completion (&compl);

ps2sif_callrpc (&cdataPortOpen, RPC_NUMBER, SIF_RPCM_NOWAIT,
                (void *) ps2mtapRpcBuffer, SEND_SIZE,
                (void *) ps2mtapRpcBuffer, RECEIVE_SIZE,
                (ps2sif_endfunc_t) ps2mtap_rpcend_notify,
                (void *) &compl);

wait_for_completion (&compl);
{% endhighlight %}

Where `ps2mtap_rpcend_notify` has the standard definition:

{% highlight c %}
static void
ps2mtap_rpcend_notify (void *arg)
{
  complete ((struct completion *) arg);
  return;
}
{% endhighlight %}

> You may need to use `dma_cache_inv()` if you transfer additional
> stuff which is not `ps2mtapRpcBuffer`.

I don't transfer anything else.

_Mega Man_, thanks for those hints. Any other clues?

> Are you sure that `init_completion()` is called directly before each
> `ps2sif_callrpc()` call?

Yes, I am.

> If you resuse the same `compl` from `ps2sif_bindrpc()`, you can get
> an early wait queue wake up.

I don't reuse it.

> Try to add a `msleep()` after `ps2sif_callrpc()` and add `printk`
> after and in `ps2mtap_rpcend_notify()` with counter.

I did that, but it still doesn't work.

> If this is not the problem, you need to post your complete code
> change here (in a testable way).

I'm posting [the source code here](mtap-driver.patch) --- it's a patch
to apply against the Linux source tree.  It adds a module called
`ps2mtap`, by modifying the files `drivers/ps2/Kconfig` and
`drivers/ps2/Makefile`, and creating `drivers/ps2/mtap.c` as well.
Build it as a module and don't forget to load `freemtap.irx` via
_Kloader_ as module1.irx.

> The RPC communication is working. The open call returns `-4`. I
> assume this means that no multitap is connected because I don't have
> one.

Very strange.  Every time I get `0` in `ps2mtapRpcBuffer[REPLY]` as a
reply to the opening call --- with or without a multitap plugged in.
What can it be?

In my source code analysis, I've found the function that should be
responsible for that reply.  It's in the file `freemtap.c` from the
`freemtap.irx`'s source code, namely `mtapPortOpen`.

Assuming my reasoning is sound and that the _RPC_ is working right, it
happens that it's impossible to obtain `0` in
`ps2mtapRpcBuffer[REPLY]`.  Therefore, if my reasoning is sound, the
_RPC_ isn't working right for me.  Please follow the reasoning...

Given the function:

{% highlight c %}
s32 mtapPortOpen (u32 port)
{
  if (port < 4)                               **C1**
    {
      state_open[port] = 1;
      s32 res = get_slot_number(port, 10);

      if (res < 0)                            **C2**
        {
          state_getcon[port] = 0;
          state_slots[port] = 1;
          return res;                         **R1**
        }
      else
        {
          state_getcon[port] = 1;
          state_slots[port] = res;
          return 1;                           **R2**
        }
     }

     return 0;                                **R3**
}
{% endhighlight %}

Assume that `port == 0 || port == 1` evaluates to `1` (the only cases
that the kernel driver cares about) and the function's return value is
`0`.

The function's exit points are three, namely _R1_, _R2_ and _R3_.  So,
it's clear that the function only can return `res` variable's value,
`1` or `0`.  But we know that it returns `0` by hypothesis, thus it
exits in point _R1_ or _R3_.

Assume that it returns in point _R1_.  Therefore the conditional _C2_
is satisfied, i.e., the `res` variable holds a negative value.  That's
an absurd because it must hold the return value `0` by hypothesis.

So the exit point must be _R3_.  Therefore the conditional _C1_
doesn't get satisfied, otherwise the return points would be _R1_ or
_R2_.  Thus `port == 0 || port == 1` evaluates to 0, contradicting our
hypothesis.

The absurd came from the assumption that it was possible to `port == 0
|| port == 1` evaluate to `1` while the function's return value was
`0`.  As I get `0` every time that I try to open port `0` or `1`, I
can only conclude that function never gets called or its value isn't
preserved and therefore the _RPC_ isn't working right for me.  Does
anything come to mind?

> Maybe it is related to the slim PSTwo.  I used the fat PS2 for
> testing.  I disabled the pad driver in the Linux kernel and used the
> freesio, freepad and freemtap modules.

I did the same.

> Before calling `ps2sif_callrpc()` you can add:
> `ps2mtapRpcBuffer[REPLY] = 0x55`; Then you can check if it is set to
> `0` or if `0x55` is unchanged.

I did that and got `0x55`.  So, it seems that fat and slim _PS2_
behave differently.  Strangely when I implemented this driver at
_SBIOS_ level it worked...  What changed?  Is there another
suggestion?

> You need to debug it with printf in the _SBIOS_.  On default this
> will be redirected to `printk` calls in kernel.  You should disable
> all other stuff, to reduce the amount of unrelated prints on the
> screen; e.g. Don't load other modules and disable all other _PS2_
> drivers including USB.  For testing you need only the kernel.  You
> need to add `printf` to the interrupt function.  You need to enable
> the `printf` in function `_end_request()` in
> [sifrpc.c](http://kernelloader.cvs.sourceforge.net/viewvc/kernelloader/kernelloader/TGE/sbios/sifrpc.c?revision=1.18&view=markup).
> You should also try to load _kernelloader_ directly without much
> stuff before. Starting _uLauchELF_ can be already too much.


</div>
