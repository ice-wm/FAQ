---
title: IceWM FAQ and Howto
---

Last modified 2020/03/21

### What is IceWM?

IceWM is a **window manager** for the **X11** window system.
It is designed to be **small, fast, lightweight**, and to
emulate the **look and feel of Motif, OS/2 and Windows**.
It is **very configurable**: it provides a **customizable look**
with a relatively **consistent feel**.

Now that you know what IceWM is and are still reading on you are
obviously interested in using it. To use a program you will first
need to have it. The obvious question is:

### Where to get it?

See [![here](/images/logom.jpg "ice-wm.org")](https://ice-wm.org "ice-wm.org").

### Which operating systems?

IceWM runs under:

- **Linux**
- **NetBSD**
- **FreeBSD**
- **OpenBSD**

### How to install from RPM?

See [repology](https://repology.org/project/icewm/versions)
or [pkgs](https://pkgs.org/download/icewm).

### Compile from source?

IceWM provides **autoconf** and **CMake**.
See [README](https://github.com/ice-wm/icewm/blob/master/README.md) or
[INSTALL-cmakebuild](https://github.com/ice-wm/icewm/blob/master/INSTALL-cmakebuild.md).

### Default window manager?

Under GNOME you can run
[icewm-set-gnomewm](https://ice-wm.org/man/icewm-set-gnomewm)
to make IceWM your default window manager.

Some display managers examine `~/.dmrc`:

```desktop
    [Desktop]
    Session=icewm-session
    Language=en_US.utf8
```

Make sure that `icewm-session.desktop` can be found.
Use `locate icewm-session.desktop`.
It may need to be placed in `/usr/share/xsessions/`.

Check `/var/lib/AccountsService/users/$USER` for your default session.
This may look like:

```desktop
    [User]
    XSession=icewm-session
    SystemAccount=false
```

Another way to set this is:

```bash
   $ myid=$(id -u -r)
   $ gdbus call --system --dest org.freedesktop.Accounts \
        --object-path /org/freedesktop/Accounts/User${myid} \
        --method org.freedesktop.Accounts.User.SetXSession icewm-session
```

If you use `startx` to run X11 then create a `~/.xinitrc`:

```bash
    #!/bin/bash
    icewm-session >~/.xinitrc.log 2>&1
```

Do `chmod +x ~/.xinitrc`. If you use Xdm then link:
`ln ~/.xinitrc ~/.xsession`.

The WDM display manager examines `~/.wmrc`.
Do `echo icewm-session >~/.wmrc`.

Any startup commands for your IceWM session go into the
[icewm startup file](https://ice-wm.org/man/icewm-startup).

### Configuration

When you are runnign IceWM you can switch themes via the
**Start Menu button** in the bottom left corner.
Some options can be changed on the fly via the **Preferences Menu**.
Don't forget to select **Save Modifications**. Most options
require a restart of icewm to be effective via
**Logout -> Restart IceWM**.

Configuring IceWM usually requires editing text config files.
See the [icewm](https://ice-wm.org/man/icewm) and
[preferences](https://ice-wm.org/man/icewm-preferences) man pages.

You can customize IceWM by editing the following configuration files:

- [menu](https://ice-wm.org/man/icewm-menu)
    Controls the contents of the `start` menu.
- [preferences](https://ice-wm.org/man/icewm-preferences)
    Controls the general behavior of IceWM.
- [keys](https://ice-wm.org/man/icewm-keys)
    Controls which additional key combos are available to users.
- [toolbar](https://ice-wm.org/man/icewm-toolbar)
    Controls the row of launcher icons on the taskbar and has the
    same syntax as the menu file.
- [winoptions](https://ice-wm.org/man/icewm-winoptions)
    Controls the behavior of individual applications (as identified
    by the names of their respective windows).
- [startup](https://ice-wm.org/man/icewm-startup)
    Script or command (must be executable) executed by `icewm-session` on startup.
- [shutdown](https://ice-wm.org/man/icewm-shutdown)
    Script or command (must be executable) executed by `icewm-session` on shutdown.
- [theme](https://ice-wm.org/man/icewm-theme)
    IceWM theme path/name.
- [prefoverride](https://ice-wm.org/man/icewm-prefoverride)
    To override theme preferences.
- [focus\_mode](https://ice-wm.org/man/icewm-focus_mode)
    The IceWM focus model.

#### menu


The `menu` file controls the contents in your Start menu.
It has the following syntax:

```
    prog Program Icon app -with -options
```

`prog` is a keyword, telling IceWM that it's a program entry. Other keywords
are `separator` to draw a separator and

```
    menu Xyz folder_icon {
      prog ...
    }
```

to open a new sub menu called Xyz. `Program` is
the name which will be shown in the menu. Enclose it in apostrophes if you
need more than one word here. `Icon` will be used as the menu
entry's icon, if a corresponding image is found in IceWM's IconSearchPath.
And finally `app -with -options` is what's going to be started if
a user chooses this entry.

Note that the menu only shows entries which are found in your PATH, IceWM is
clever enough to omit non-usable entries.

There are also two advanced options `runonce` and
`menuprogreload "title" icon_name timeout program_exec`

`runonce` is used to start application only once - if its already running do not start it upon
clicking. Runonce needs some other options - see manual.
`menuprogreload` is used to created dynamic menus:
timeout is integer value, it specifies minimum time
interval (in seconds) between menu reloading. Zero value means
updating menus every time when user click it.

### What are focus models?


To answer this question it is a good idea to first take a look at the
four general focus models that are implemented by IceWM:


- `ClickToRaise`
    When a window is clicked, it is raised and activated. This is the
    behavior of Win95 and OS/2.
- `ClickToFocus`
    A Window is raised and focused when titlebar or frame border is
    clicked and it is focused but not raised when the window interior
    is clicked.
- `PointerFocus`
    When the mouse is moved, focus is set to window the mouse is
    pointing at. It should be possible to change the focus with the
    keyboard when the mouse is not moved.
- `ExplicitFocus`
    When a window is clicked, it is activated but not raised. New
    windows do not automatically get the focus unless they are
    transient windows for the active window.


*A window is activated, is focused, gets the focus, ...*
means that input (e. g. keystrokes) now are sent to that window.

IceWM implements these focus models:

1. click : changing input focus requires to click a window with
   the left mouse button.

2. sloppy : set input focus merely by moving the mouse pointer
   over a window.

3. explicit : no focus change or window raise unless you force it.

4. strict : similar to sloppy, but keep focus with last window
   even if new applications become mapped.

5. quiet : like sloppy, but without flashing if something wants focus.

6. custom : Is configured by the following ten options
   in the preferences file: "ClickToFocus", "FocusOnAppRaise",
   "RequestFocusOnAppRaise", "RaiseOnFocus", "RaiseOnClickClient",
   "FocusChangesWorkspace", "FocusOnMap", "FocusOnMapTransient",
   "FocusOnMapTransientActive", "MapInactiveOnTop"

**In short:** The focus model controls what you have to do to
make a window pop up and to have it listen to what you type.

### Configure mouse buttons


`UseRootButtons` and `ButtonRaiseMask` are so called **bitmask options**.

This concept is e.g. used by `chmod` where `4` stands for read access,
`2` for write access and `1` for execute (or change directory)
access and you add up the relevant numbers to control the file access.

As far as `UseRootButtons` and
`ButtonRaiseMask` are concerned,
`1` stands for the first mouse button,
`2` for the second one and `4`
for the third one. The following list shows which number stands for
which combination of mouse buttons:

```
    ---------------------------------
     Value   Stands for
    ---------------------------------
       0     No mouse button at all
       1     Button 1
       2     Button 2
       3     Buttons 1 and 2
       4     Buttons 3
       5     Buttons 1 and 3
       6     Buttons 2 and 3
       7     All three mouse buttons
    ---------------------------------
```

Any value greater than seven has the same effect as seven.
`UseRootButtons` controls which buttons call up a
menu when clicked on an unoccupied region of the desktop.
`ButtonRaiseMask` determines which buttons will
raise a window when clicked on that window's title bar.

### Bind menus to buttons


There is an option for each of the root menus which controls which
button is bound to that menu.

```
    -----------------------------------------
     Option Name            Controls
    -----------------------------------------
     DesktopWinMenuButton   Window menu
     DesktopWinListButton   Window list
     DesktopMenuButton      Application menu
    -----------------------------------------
```


The value of each option determines the button to which the
corresponding menu is bound according to the following scheme:

```
    -----------------------------
     Value   Stands for
    -----------------------------
       0     No mouse button
       1     Left mouse button
       2     Right mouse button
       3     Middle mouse button
      4-6    Other buttons
    -----------------------------
```


### Monitor mailboxes?


No problem. See
[mailbox monitoring](https://ice-wm.org/manual/#mailbox-monitoring-updated-2018-03-04).

### Disable the Alt keys?


To send all Alt key combinations to an application,
you can use a [window option](https://ice-wm.org/man/icewm-winoptions)

```
window_class.fullKeys: 1
```

The [preference](https://ice-wm.org/man/icewm-preferences)
you are looking for is `ClientWindowMouseActions=0`.
This disables Alt+mouse drag to move window for all IceWM handled windows.


### Control Applications


This section is about how you can make windows appear on a certain
workspace, have them displayed without a border or titlebar, or put them
above or under other windows. This is accomplished using 
[winoptions](https://ice-wm.org/man/icewm-winoptions).

Assigning a particular option (icon, default layer, default
workspace, etc.) to a given application or application window can be
done as follows:

First, you should acquire the `WM_CLASS` descriptor
using `xprop`:

```
    xprop | grep WM_CLASS
```

in a terminal.
The output of `xprop` for Mozilla Firefox is

```
    WM_CLASS(STRING) = "Navigator", "Firefox"
```

The first double quoted name is the window name
and the second is the window class.
You can then add the desired options to your
`winoptions` file. Entries in that file have one
of the following formats:

```
    name.class.option: value
    class.option:      value
    name.option:       value
```

To assign the icons `navigator_*.xpm` to the
Netscape Navigator window, use this option:

```
    Navigator.Netscape.icon: navigator
```

The other options work according to roughly the same pattern.
The list of winoptions you can find in the
[man page](https://ice-wm.org/man/icewm-winoptions)
about Window Options.


### Keep window on top?


There are two slightly different ways to do this. Use whatever suits your
need. Option one: the window always stays on top of any other windows. Set
the following winoption `name.class.layer: onTop`.
Option two: the window sits in a rectangular zone of the desktop where no
other windows can be placed: Use the doNotCover winoption:
`name.class.doNotCover: 1`. By the way, this is how the taskbar or
the GNOME panel work. It's a good idea to use this on gkrellm, your icq
client, or other monitoring tools you'd always like to have in view.

### Iconify or maximize?


There may be programs that you either want to start up iconified or
maximized. For this there are winoptions: `startMinimized`,
`startMaximized`, `startFullscreen`, etc.

### Map to a workspace?


Either use `winoptions` and define

```
xmms.workspace: 7
Mozilla.workspace: 9
```

This allways starts xmms on workspace 7 and Mozilla on workspace 9, keep in
mind, IceWM starts counting at 0. IceWM will switch to the nominated
workspace on every start of these programs.

Or you can use `icesh`:

```
icesh -class xeyes setWorkspace 0
```

This move xeyes to workspace 0.


### Basic keyboard shortcuts

It should be possible to control everything by keyboard. Here we show some
of the not so obvious ways to achieve important window managing tasks only
with keystrokes.


```
Alt+Tab = Switches between the open windows
Alt+F4 = Closes a window
Alt+F9 = Minimizes a window
Alt+F10 = Maximizes a window
Alt+F12 = Rolls the window up
Alt+Shift+F10 = Maximizes the window vertically
Ctrl+Alt+arrow left = Changes workspaces from 1-12
Ctrl+Alt+arrow right = Changes workspaces from 12-1
Ctrl+Alt+Esc = Opens the  window list
Ctrl+Esc = Opens the  menu
```


### Switch Desktop by key


You are accustomed to a window manager that allows you to switch
between virtual desktops using your keyboard? IceWM allows for this,
too.

Before I describe how to switch between virtual desktops I want to
describe how to control their number. Imagine that your
`~/.icewm/preferences` has a row reading

```
    WorkspaceNames="1","2","3","4","5","6","7","8","9","0"
```

This setting results in ten virtual desktops and ten buttons in your
taskbar looking like this:

```
    +---+---+---+---+---+---+---+---+---+---+
    | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 |
    +---+---+---+---+---+---+---+---+---+---+
```

If you name less desktops you obtain less if you name more you get
more.

For understanding how switching virtual desktops works in IceWM you
should imagine that the buttons represent your virtual desktops and
that these desktops are arranged in one long row.

You can imagine two ways of switching between desktops:


- Switching to desktop number seven
- Switching to the desktop on the left/right of the present one


IceWM has both ways:


- To switch to desktop number *n* you simply press `Ctrl+Alt+n`.
- To switch one desktop to the left you press `Ctrl+Alt+Left`.
- To switch one desktop to the right you press `Ctrl+Alt+Right`.


If you are using `Ctrl+Alt+Right` on the
rightmost desktop you switch to the leftmost desktop.
From here, `Ctrl+Alt+Left` brings you back to the
rightmost desktop.

*What if you have more than ten virtual desktops?* In this
case `Ctrl+Alt+n` will only work for the first ten
desktops while switching to the left or right still works for all
desktops.

**Note:** To switch desktops when moving mouse on desktop edges use preference:

```
    EdgeSwitch=1
```

then you can change workspaces by moving your cursor to
the left/right edges of your screen.


### Move windows to desktops


In the previous section I explained how to switch between desktops.
If you didn't already read it you should do it now because moving the
active window to another desktop works almost the same like switching
to a certain desktop. All you have to do is press the
`Shift` while switching to the desktop:


-  To move a window to desktop number *n* press `Ctrl+Alt+Shift+n`.
-  To move a window one desktop to the left press `Ctrl+Alt+Shift+Left`.
-  To move a window one desktop to the right press `Ctrl+Alt+Shift+Right`.


### Command Line Interface


You should run IceWM with `TaskBarDoubleHeigth=1`,
because that will enable the *Address Bar*
(see *What is the blank bar in the task bar good for?*).
This is especially useful if you rather frequently need to access
man pages and don't want to have `xman` hang around all the time.

If you enter `man perl` in the *Address Bar* and press
`Ctrl+Enter` a terminal will pop up displaying the
Perl man page. If you press `q` not only the
man page no longer is displayed but the XTerm will terminate, too.

This is just one example of how to use the CLI. You can use
it to issue any other command as well. A problem that might occur is
that the XTerm will terminate before you had time to read the output
of a command (it terminates as soon as the command is done).
The terminal may support a **hold** resource or `-hold` option,
which will keep the terminal open until it is closed by you.


### Can I use Win(95) keys?


Sure you can. In `.xinitrc`

```
    clear mod4
    keycode 64 = Alt_L
    keycode 113 = Alt_R
    keycode 115 = Meta_L
    keycode 116 = Meta_R
    add Mod4 = Meta_L Meta_R
```

in `.Xmodmap` there is:

```
    add Mod1 = Alt_L
    add Mod2 = Mode_switch
    keycode 117 = Menu
```

and in `~/.icewm/preferences`

```
    Win95Keys=1 # was 0
```

On a free workspace the right Win95 opens the list of Workspaces.

Now also in OpenOffice I can use the right menu key to open the menus in the
OOo taskbar with the letters for the shortcut I can switch to the desired
menu without needing to leave the keyboard,  my preferred way of working
on the pc.


### How to install Themes


IceWM can be customized using a great variety of themes.
You can download them from
[https://www.box-look.org/](https://www.box-look.org/browse/cat/142/ord/latest/).
To install themes simply unpack them into your `~/.icewm/themes/` directory.

### Which image formats?


IceWM supports JPEG, PNG and XPM. Support for SVG is optional.

### Setting the background


If you provide the appropriate options in your
`preferences` file and start `icewmbg`, IceWM will set the
background color or the background image for you. You can use

```
    DesktopBackgroundColor="color"
```

to set a background color and

```
    DesktopBackgroundImage="image"
```

to set a background image. To keep IceWM from setting a background
color/image you simply set *both* options to an empty string:

```
    DesktopBackgroundColor=""
    DesktopBackgroundImage=""
```

**Hints:**

1.
    Commenting out
    `DesktopBackgroundColor="color"`
    and `DesktopBackgroundImage="image"`
    does not have the intended effect.
2.
    IMHO using a background image (especially a huge one) isn't that
    good an idea. It awfully slows down the X windowing system.


To distinguish between filling whole desktop with image or to place it self
standing in the middle you can use

```
    DesktopBackgroundCenter=""
```

DesktopBackgroundCenter is used to tell IceWM how you want
your wallpaper placed on the screen.
If set to 1 your picture will be centered on screen.
As a result of that, you will only have one picture
in the middle of your desktop.
If set to 0 your picture file will fill the whole screen.
That is a good thing if you are using a pattern thing
to cover the whole desktop.


### Setting the clock format


Setting up the look of the task bar clock of IceWM as well as the
format of the associated tooltip is rather easy. IceWM uses the same
format as the Unix standard function `strftime` so
when in doubt you can always refer to

```
    man 3 strftime
```

To set the clock format you use

```
    TimeFormat="<format string>"
```

and for the clock tooltip format you use

```
    DateFormat="<format string>"
```


Ordinary characters placed in the format string are printed without
conversion (if possible, see below). Conversion specifiers are
introduced by a percent character `%`, and are
replaced by a corresponding string.

**Important Note:** While `DateFormat` and
`TimeFormat` both support all the format
descriptors the latter only has full support if used with

```
    TaskBarClockLeds=0
```

(which is set equal 1 by default).

The reason for this is that there are no icons to display the name of
a month, day, or time zone. To be more precise there are only icons
for

1. digits (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
2. colon, dot, slash, and space
3. A, P, and M (for AM and PM)


Format descriptors which may only be in `TimeFormat` if
`TaskBarClockLeds=0` (in general or depending on the locale)
are labeled as **restricted** in the following table.
It shows the replacement for all format descriptors available.

The values in parentheses show what the different format specifiers
display for

`YYYY/MM/DD HH:MM:SS TimeZone = 1999/09/04 19:09:22 UTC`

on my machine with hardware clock and Linux running UTC, local being
"C" (i.e.  no internationalization at all):


- `%a` (Sat) restricted
    The abbreviated weekday name according to the current locale.
- `%A` (Saturday) restricted
    The full weekday name according to the current locale.
- `%b` (Sep) restricted
    The abbreviated month name according to the current locale.
- `%B` (September) restricted
    The full month name according to the current locale.
- `%c` (Sat Sep 04 19:09:22 1999) restricted
    The preferred date and time representation for the current
    locale.
- `%d` (04)
    The day of the month as a decimal number (range 01 to 31).
- `%H` (19)
    The hour as a decimal number using a 24-hour clock (range 00 to
    23).
- `%I` (07)
    The hour as a decimal number using a 12-hour clock (range 01 to
    12).
- `%j` (247)
    The day of the year as a decimal number (range 001 to 366).
- `%m` (09)
    The month as a decimal number (range 01 to 12).
- `%M` (09)
    The minute as a decimal number.
- `%p` (PM) restricted
    Either "am" or "pm" according to the given
    time value, or the corresponding strings for the current locale.
- `%S` (22)
    The second as a decimal number.
- `%U` (35)
    The week number of the current year as a decimal number, starting
    with the first Sunday as the first day of the first week.
- `%W` (35)
    The week number of the current year as a decimal number, starting
    with the first Monday as the first day of the first week.
- `%w` (06)
    The day of the week as a decimal, Sunday being 0.
- `%x` (09/04/99) restricted
    The preferred date representation for the current locale without
    the time.
- `%X` (19:09:22) restricted
    The preferred time representation for the current locale without
    the date.
- `%y` (99)
    The year as a decimal number without a century (range 00 to 99).
- `%Y` (1999)
    The year as a decimal number including the century.
- `%Z` (UTC) restricted
    The time zone or its name or its abbreviation.
- `%%` restricted
    A literal "%" character.


### How to add icons?


An icon for an application can be specified by a name,
filename, full path or path prefix in `winoptions`.
To locate an icon which is specified by name or filename,
IceWM looks at the value of IconPath in `preferences`.
This is colon-separated list of directories.
A directory is subjected to tilde expansion and expansion
of at most one leading environment variable like `$HOME`.
If the icon is still not found sofar, then IceWM looks
for icons in `$ICEWM_PRIVCFG/icons/` (or `~/.config/icewm/icons/`,
or `~/.icewm/icons/`), then at theme icons,
then at CFGDIR/icons, then at LIBDIR/icons.
Here CFGDIR and LIBDIR are defined at compile time
and can be queried by `icewm --directories`.

### How to make themes?


There is documentation on [themes](https://ice-wm.org/themes/)
written by MJ Ray and update by Adam Pribyl.


### What is the Logout Command?


For most users, nothing. The Logout and Cancel commands were meant for GNOME
integration as alternative commands that would be run when users
initiated a logout or logout cancel. Since GNOME did not seem to
incorporate this feature, they generally go unused.

### A blank field in taskbar?


If you are running IceWM with the
`TaskBarDoubleHeight` option set, a blank field in
the task bar occurs. It is a command line interface.

In this field you can enter commands to start programs. If you click
inside the field and enter `xclock` the X clock is
started.

If you click on it and simply press `Ctrl+Enter`
an XTerm is being started.

If you enter a non-X command and press
`Ctrl+Enter` an that command is being executed in
an XTerm.

### Stop grabbing my keystrokes


What if you are running an application and need to use a keystroke
that is grabbed by IceWM?

Marko suggests the following workaround:


1. Activate scroll lock
2. Do problematic key stroke
3. Deactivate scroll lock


He advises that this will only work if
`ScrollLock` is set up as a modifier.

Here is how to use the X11 `xmodmap` utility to setup `ScrollLock` as
a modifier (from Marco Molteni):
-  check which modifiers are free:

```bash
    $ xmodmap -pm

    xmodmap:  up to 2 keys per modifier, (keycodes in parentheses):

    shift       Shift_L (0x32),  Shift_R (0x3e)
    lock        Caps_Lock (0x42)
    control     Control_L (0x25),  Control_R (0x6d)
    mod1        Alt_L (0x40),  Alt_R (0x71)
    mod2        Num_Lock (0x4d)
    mod3
    mod4        Super_L (0x73),  Super_R (0x74)
    mod5
```

-  in this example `mod3` is free, so we bind the `ScrollLock` key to it:

```bash
    $ xmodmap -e "add mod3 = Scroll_Lock"
```

   This invocation of `xmodmap` could be put in the script that starts the
   window manager, `~/.xinit` or `~/.xsession`, or
   in [startup](https://ice-wm.org/man/icewm-startup)


### Setting the lock command

To set the lock command, add

```
    LockCommand="xlock -mode blank"
```

to your `$HOME/.icewm/preferences` and xlock will run in blank mode.

In case you prefer xscreensaver

```
    LockCommand="xscreensaver 2>/dev/null; sleep 1; xscreensaver-command -lock"
```

### How to lock the screen


Screen locking is something you should do whenever you leave your
machine (even at home and even for only a few seconds - just imagine
a cat pushing the enter button at the wrong moment). It should be a
habit like logging out root as soon as possible.

### ... by keyboard


With IceWM screen locking is very easy: If you press

```
    Ctrl+Alt+Del
```

a menu pops up offering you the following tasks:


- Loc*k* Workstation
- *S*leep mode
- *C*ancel
- *L*ogout
- Re*b*oot
- Shut*d*own
- *W*indow List
- *R*estart icewm
- *A*bout


The letters that are emphasized are underlined in the dialog.
The meaning of this emphasis is that you may e. g. press
`k` to lock your workstation.

Another possibility is to use the arrow keys to navigate and
then press `Enter` to activate the task.

### ... by mouse


If you prefer to use your mouse to lock the screen you may add the
following entry to your `$HOME/.icewm/toolbar`

```
    prog    xlock   xlock   xlock
```

You could as well add that line to `~/.icewm/menu` or
`~/.icewm/programs`, but that's not a good idea:
Screen locking is often done in a hurry and if you have to scan
through a menu this will increase the chance that you will not lock
your machine at all.

### ... using a lock command


How to define a different lock command is described in
section "Setting the lock command"

### Support session management?


From 1.2.13 IceWM has some basic session management to manage all its parts.
But this is where the more complicated desktop environments like
GNOME, KDE or xfce join the game. IceWM still is mainly a window manager...
but of course you can always start your favorite apps upon X start-up/login
using the `.xinitrc` or `.xsession` files. Or use IceWM as the
window manager instead of the default GNOME/KDE window manager.

### Can I have icons on the desktop?


Sure, but not from IceWM. Again, this is desktop environment work, but
usually done by the respective file managers, since they already know about
MIME types, file endings and such. IceWM users usually use
`idesk`, `dfm`, `rox`, `kfm` or `gmc`.

### My background is ignored?

Usually this is because it's the wrong image format. It can happen when
IceWM is compiled only with libXpm.
With imlib, IceWM is able to read most of the often used image
formats like png, gif, jpeg, instead of just xpm images with libXpm. Another
reason can be, that the theme defines another image or color.

### Can I have bigger icons?

From IceWM 1.2.14 it is possible to specify size of icons in IceWM preferences.
There are four relevant options:

```
    MenuIconSize=16
    SmallIconSize=16
    LargeIconSize=32
    HugeIconSize=48
```

These values are default but you can change them to whatever you want.
`MenuIconSize` specifies size of icons in menu. Three other are used for
any other icon in IceWM. E.g. `SmallIconSize` is used in taskbar,
application frames and window list. `LargeIconSize` is used in quickswitch.

You have to take in mind that when you change size of `SmallIconsSize`
then all above described parts will have icons of different size, but taskbar
and frames will not change their high accordingly! Also when you specify the size
that is not available, then icons will be resize - this can cause some disturbance
mainly when you are using xpm icons.

There is a trick to increase size of taskbar however.
Taskbar height is sized according
size of start button. E.g. for linux if your `linux.xpm` in taskbar
folder is 50x32 then your taskbar will be 32 pixels high.

To change the height of frames you have to make theme with higher frames.


### How to translate IceWM?

The best option is to join
[openSUSE Weblate](https://l10n.opensuse.org/projects/icewm/icewm-1-4-branch/).
First register an account either
[here](https://l10n.opensuse.org/accounts/register/) or
[there](https://idp-portal.suse.com/univention/self-service/#page=createaccount)
then improve the translation for your language.


The other option is to create a copy of `icewm.pot` and rename it to
`cs.po` or whatever is right for your language.
Then you have to translate the file using any of the tools for
gettext file transaltion, e.g. kbabel, or you can edit it by hand.
After translation you can send it to icewm-devel list or post it
as patch in patch tracker.


If you want to test file yourself you can add this file
into `po` directory under IceWM sources and then configure IceWM
(`./configure`) and type `make` in `po` directory.
This creates .mo file, which you can either copy to locale locations
(e.g. /usr/local/share/locale/cs/LC\_MESSAGES) or you can do `make install`.

### How to use Xrandr?

IceWM supports since a few versions the xrandr feature of X11. This can
very easily be used to define a menu item on your toolbar to change the
display resolution, provided that you run recent enough versions of both
X11 and IceWM that supports xrandr. You can run `xrandr -q` to see the
resolutions supported using your present X configuration (maximum
resolution and color depth). You can edit this menu fragment when you
have checked which resolutions work and then you can put it into your
[toolbar](https://ice-wm.org/man/icewm-toolbar).

```
    # IceWM toolbar menu to change the display resolution.
    # This needs xrandr support from both X11 and Icewm.
    #
    # Xrandr is considered an experimental feature, so your screen may go
    # blank if you have a problem with some resolution setting.
    # It is a good idea to close your other windows before testing.
    #
    # Check your own resolutions with xrandr -q and modify accordingly.
    # This example assumes a default resolution of 1280x1024.
    #


    menu Resolution redhat-system-settings {
       prog 1280x1024 1280x1024 xrandr -s 0
       prog 1152x864  1152x864  xrandr -s 2
       prog 1024x768  1024x768  xrandr -s 3
       prog  800x600   800x600  xrandr -s 4
       prog  640x480   640x480  xrandr -s 5
    }
```

The redhat-system-settings is a bitmap I picked up from my Fedora Core 3
box, you can put there whatever you want of course.



### Example: configuration A-Z


This is a sample of possible configurations you could do to have IceWM
running with all you need. Following applies for RedHat(9).
Placement of files can be bit different.

### X window login

To have possibility to switch to IceWM  in GDM greeter
(after start to runlevel 5 = Xwindow),
then you need to do following things:


- Add to `/etc/X11/gdm/Sessions/` (gdm is default greeter)
  file `IceWM` with content:

```bash
    #!/bin/bash
    exec /etc/X11/xdm/Xsession icewm
```

- Modify `/etc/X11/xdm/Xsession` to understand what "icewm"
  is (this may not be necessary).

- Add to `/usr/share/apps/switchdesk/` file `Xclients.icewm` with content:

```bash
    #!/bin/bash
    exec /usr/local/bin/icewm-session
```


### IceWM configuration


To configure all of IceWM options go to sections about configuration.
Generally all you need to customize IceWM globaly, is to edit `/usr/local/share/icewm/preferences` etc.


### Icons on desktop


Usually people want to have icons on desktop.
One of most simple applications that can
satisfy this need is `idesk` (see Tools to find it).
Configuration of idesk is almost as easy as configuration
of IceWM, but has one disadvantage:
idesk does not have in version 0.3.x global configuration file.
Therefore each user needs to have proper configuration
file in his/her home.

To configure idesk you need to:

- Add `~/.ideskrc` file with content like this

```
    table Config
      FontName: Helvetica
      FontSize: 9
      FontColor: #ffffff
      PaddingX: 35
      PaddingY: 25
      Locked: true
      HighContrast: false
      Transparency: 50
      Shadow: true
      ShadowColor: #000000
      ShadowX: 1
      ShadowY: 1
      Bold: false
    end
```

- Add `~/.idesktop` directory

- Add `whatever.lnk` files into it, with content like this

```
    table Icon
      Caption: Mozilla
      Command: mozilla
      Icon: /usr/share/pixmaps/mozilla-icon.png
      X: 22
      Y: 13
    end
```

- Do not forget you need to start idesk at the beginning of the session.
  Best to achieve this is using your `~/.icewm/startup` file (for details see Configuration section).
  In case of idesk you can add line:

```bash
    idesk > /dev/null & # start idesk - desktop icon manager
```


### IceWM ignores my colors

Some users wonder why the colors specified in their preference files
seem to have no effect upon the actual appearance of things. The
reason is that these settings may be overridden by settings in the
theme file.

The theme file can control all of the options
controlled by the `preferences` file, but usually
theme authors are decent confine their meddling to superficial
aspects of window manager behavior and leave control over most
important behaviors to the user.

If this wasn't the reason: If you are running X in 8-bit mode then it
is possible that the specified color simply isn't available.

You don't know if X is running in 8-bit mode? Run

```
    xwininfo | grep Depth
```

in an XTerm and click on the root window (the desktop). If this
command displays

```
    Depth: n
```

you are running X in n-bit mode (n typically is 8, 16, 24 or 32).

### Programs are missing from menus


A very annoying problem are programs you added to the `menu` file but
that are missing in the corresponding menus. That isn't really a bug
of IceWM. The point of view of IceWM is that it makes no sense to
display a program that *is not present*.

The crucial point is the meaning of *to be present*.  It does not
mean *to be installed*, but *to be found using the present path*.
Do `echo $PATH` or `which program` to find if a program is in PATH.

To fix the problem you have at least three possibilities:


1.  You give the full path and not only the program name itself.
2.  You set the PATH in your `.xinitrc`, `.xsession` or `.Xclients`.
3.  You set the PATH in [env](https://ice-wm.org/man/icewm-env).


### IceWM maximizes windows over the GNOME panel


This used to be a really annoying problem, but seems to be gone with newer
versions of IceWM and GNOME. If it still happens on your machine try to set

```
    Panel.doNotCover: 1
```

in your `winoptions` file.

### Screen locking doesn't work


The reason for this is that the standard lock command
(`xscreensaver-command` or `xlock`) could not be found by IceWM.
See "Setting the lock command" for details on setting
a different lock command.

### Background does not show up


IceWM is divided in few separated parts. One of them is `icewmbg`.
This part takes care of bacground setup. Therefore if you want IceWM to
take care of desktop background you have to start `icewmbg` at
IceWM startup. The proper way is to start "icewm-session" in your
X startup instead of just icewm.
See "Configuration".


### Font settings are ignored


IceWM uses two ways of font handling - corefonts OR fonts provided by xfreetype library.


These fonts can be specified in `preferences` or theme `default.theme`.

For X server provided fonts (configure --enable-corefonts option) the definition looks like this:

```
    ActiveButtonFontName = "-artwiz-snap-regular-r-normal-sans-10-*-*-*-*-*-*-*"
```

For Xft (xfreetype) library (used by default, disable using option --disable-xfreetype),
then specification is like this:

```
    ActiveButtonFontNameXft = "Snap:size=10,sans-serif:size=12:bold"
```


To provide correct fonts to Xft you have to specify them in `/etc/fonts/fonts.conf`.
X server font are either provided by X server itself e.g. `/etc/X11/XF86Config` - Section "Files",
or by XFS (X Font Server) defined in. `/etc/X11/fs/config`.


### How can I make the title bar of an XTerm span the whole screen when maximized?

Set `ConsiderSizeHintsMaximized=0`.


### How can I disable the Super keys, which open the WindowList or Menu?

Set `Win95Keys=0`.


### How can IceWM load new keybindings from .icewm/keys without a restart?

Do `icesh keys`.


### License


This document is released under the terms of
the GNU Library General Public License.

### Authors

- Adam Pribyl
- Markus Ackermann
- Josef 'Jupp' Schugt
