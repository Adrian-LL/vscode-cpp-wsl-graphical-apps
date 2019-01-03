# Installing & Using Visual Studio Code for C++ ... - Part 3 - Windows + WSL + Graphical Apps Under WSL & the Conclusion

See also [Part 1](https://www.linkedin.com/pulse/installing-using-visual-studio-code-g-plus-other-tools-ludosan/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BGDGdSyRvTveQ7%2B5uucpnbw%3D%3D) and [Part 2](https://www.linkedin.com/pulse/installing-using-visual-studio-code-c-learning-part-ludosan/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3B4k7jTbiRRaeIOnDUj%2BmWnQ%3D%3D&) of this series.

# Running native Linux VSC in WSL (and some other applications) 

Now, the final complication -- that is running VSCode as graphical app in WSL.

Yes, this is possible, although not supported by Microsoft. Mainly because WSL implementation is not based on a kernel.
See more about internal workings of WSL at [Jessie Frazelle Blog (very nice and short explanation)](https://blog.jessfraz.com/post/windows-for-linux-nerds/) or at Microsoft (e.g. https://blogs.msdn.microsoft.com/wsl/2016/04/22/windows-subsystem-for-linux-overview/ or https://docs.microsoft.com/en-us/windows/wsl/about).

> So follow this only if you have too much time to spare or really want to learn something. Maybe in time this will get better. 

You can avoid all of these by installing a linux virtual (or real) machine :-)

## 1. Preparing a better console / terminal for WSL

The preparation to run graphical apps under WSL leads to a **collateral issue, about using the best console / terminal for WSL under Windows**.

There are a lot of discussions around about why the Microsoft native console is not enough -although is getting better end better (see here: https://blogs.msdn.microsoft.com/commandline/tag/wsl/ ).

### Problems I had using the standard console:
- not being able to use the mouse in mouse-aware programs (e.g. mc, vim, htop),
- impossibility to use history (with Up and Down arrows: e.g. in bpython or in virtual environments in Python, or in git),
- incomplete compatibility with curses library (clearing terminals) or xterm equivalence.
These were not so bad as they seem.

As an example, For example, try to do this (the actual command is `curl wttr.in`) :-)

![wttr](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/wttr.JPG)

This is also a matter of personal preference.

- One simple solution is **to run tmux under windows console**. Tmux is preinstalled in WSL, but it implies a lot of shortcuts to learn.
- **Conemu (and Cmder), hyper and other** seems either bloated, or too big or too buggy for what hey offer (and for my taste). For example there are matters regarding using the mouse, or the history (with Up and Down arrows or other key combinations).
- [**Qterminal** for Windows](https://github.com/kghost/qterminal/releases/) is working (although in beta state)
- The simplest seems to be [**Mobaxterm (version 11)**](https://mobaxterm.mobatek.net/). This also has a lot of other functions such as capabilities to be used as Putty replacement, a serial terminal, SSH, Rsh, RDP, VNC AWS S3 - if you know what are these, you probably don't need my article :).

> Just install Mobaxterm and it works (i.e. leave it with the default configuration). It includes the X Server needed to running graphical apps. Mobaxterm contains a Cygwin toolkit, but we will not use it. 
See the default Mobaxterm after installation.
 
![mobaxterm](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/mobaxterm.JPG)

As you can see, there is now a WSL-Ubuntu (if you have Ubuntu installed) in the Mobaxterm. 

Just start this terminal. Do NOT use "Start local terminal" (this is the local Cygwin implementation). 

Note the X server icon in the upper right. By default a X server is started. Multiple tabbed windows (i.e. shells can be opened). Different linux versions can be configured via the Settings menu.

![mobaxterm-tmux](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/mobaxterm-tmux.PNG)

Mobaxterm running tmux with 2-windows split, one running mc, and the other bash.
Mouse is working for resizing thanks to adding below in .tmux.conf (create it if you don't have it).
```bash
# .tmux.conf
# Enable mouse control (clickable windows, panes, resizable panes)
set -g mouse on
set -g default-terminal "screen-256color"
```
As the ls command looks a little ugly when listing Windows folders, some tweaks can be added to .bashrc (see code and comments below).
```bash
# Change ls colours
LS_COLORS="ow=01;36;40" && export LS_COLORS
 
# For switching to mounted home easily using a command such as cd m~
# Change the path below to wherever your Windows home is.
 
cd() {
    if [[ $@ == "m~" ]]; then
        command cd "/mnt/c/Users/adrian"
    else
        command cd "$@"
    fi
}
 
# Not needed if you use the default configuration of Mobaxterm.
# Change it and uncomment if really needed.
 
# export DISPLAY=:0
```
So I stick with Mobaxterm for now.

## 2. Preparing Windows and WSL for Graphical Apps

I assume that WSL Ubuntu 18.04 is installed (as in previous article).

For running graphical apps, an X Server is needed. I will assume that Mobaxterm is installed with default options to use both its console and its X Server capabilities.

Of course other X Servers can be used (such as [VCXsrv](https://sourceforge.net/projects/vcxsrv/), [Xming](https://sourceforge.net/projects/xming/) in the free category, or [X410](https://token2shell.com/x410/) in the paid apps), but in my tests I had the best results with Mobaxterm.

### Step 1: Install dependencies (libraries or additional programs needed). 
I tried to minimize the overhead, but be prepared for a lot of download and an increase of 1GB or more of your occupied disk space.
```bash
$ sudo apt install libgtk2.0-0
$ sudo apt install libxss1
$ sudo apt install libasound2
$ sudo apt install dbus-x11
```
Is is possible that `libgtk3.0` is also needed, so please install it:
```bash
$ sudo apt install libgtk3.0
```
or, in a single line
```bash
$ sudo apt install libgtk2.0-0 libxss1 libasound2 dbus-x11 libgtk3.0
```

Some remarks:
- `libasound2` is needed for VSC, although not really useful. Alternatively you can install the `alsa` package (also not really useful).
- `dbus-x11` is needed (in theory) for communication between programs, but I did not see any improvements with it. *It did not really worked for my environement*.

### Step 2: Install an X terminal. Not really needed, but just as a proof of concept.
```bash
$ sudo apt install xfce4-terminal
```
After installation, you can launch it with:
```bash
$ xfce4-terminal &
```
>Starting now, the missing features from WSL (as compared with a full, kernel-based installation of Linux) begin to appear.

See for example an error message in Mobaxterm windows. The terminal works well though.
![terminal](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/terminal.PNG)
 
### Step 3 - now for the tricky part(s) - VSC:

Add VSC repositories to Ubuntu apt (as they are not included by default):
```bash
$ cd ~
$ curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
$ sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
$ sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
```

Actually install VSC:
```bash
$ sudo apt update
$ sudo apt install code
```
and if you want to try the insiders versions:
```bash
$ sudo apt install code-insiders
```
It should be ready now? Unfortunately, not yet...

### Step 4 - changing the default VSC options:
Launch VSC with:
```bash
$ code
```
> Now the problems begin 

Since version 1.29, the default windows border style is "custom". You will get something as below, but a window impossible to resize and/or move. 
See the difference in the next image.

![custom](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/custom.PNG)

Modify from menu File -> Preferences -> Settings -> enter in the search bar "window.titleBarStyle" and change from "custom" to "native".

![window.titleBarStyle](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/window.titleBarStyle.png)

![window.titleBarStyle-native](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/window.titleBarStyle-native.png)
 
Code will ask to restart. And, voila, a window with borders, and that can be resized and/or moved.

![native](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/native.PNG) 

> Please note that the steps above are not needed on a full Linux installation (i.e. not WSL) 

Another default option is when launching a terminal, the terminal.integrated.rendererType - by default this is set to "auto". Choose "dom".

![dom](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/dom.PNG)

### Step 5: install extension(s) for C++. 
If you try to install via the interface, it will not work (sometimes, very rarely, it does). So, manual process, for example as below. Ensure that code (or code-insiders) is NOT running, otherwise some other errors occur.
```bash
~$ wget https://github.com/Microsoft/vscode-cpptools/releases/download/0.20.1/cpptools-linux.vsix
~$ code --install-extension cpptools-linux.vsix 
Extension 'cpptools-linux.vsix' was successfully installed!
~$
```

Instead of `wget` command, the extension can be downloaded from Windows, with a browser from Marketplace and saved somewhere where it can be accessed.
> Same operations for code-insiders if installed.

#### Optional steps
**These were not really working (or actually working in less than 20% cases...). But just in case you want to experiment...**

#### Optional 1 - configuring dbus
`dbus` is needed for inter-process communications. By default it uses unix sockets (that are not implemented in WSL). So, we may try to replace with tcp.
But first it need to be installed (which was done in Step 1 above, with `sudo apt install dbus-x11`) and configured.

a) `.bashrc` changes - add the following to your `.bashrc`

```bash
# X Server
export LIBGL_ALWAYS_INDIRECT=1
export NO_AT_BRIDGE=1
# Note - the following is not needed when using Mobaxterm
# *and* the session was launched from it

export DISPLAY=0:0
# Setup a D-Bus instance that will be shared by all X-Window apps. NOTE - apparently the command does not work, use the next.

# be sure that you are in the sudoers file
# pidof dbus-launch 1> /dev/null || dbus-launch --exit-with-x11 1> /dev/null 2> /dev/null

# The following seems to be better
sudo /etc/init.d/dbus start &> /dev/null
```

b) Grant our user access without password to the `dbus` service

Run the following command:
```bash
~$ sudo visudo -f /etc/sudoers.d/dbus
```
Then paste the following text inside the Nano editor that will launch (Replace `your_username` with your linux username):
```bash
your_username ALL = (root) NOPASSWD: /etc/init.d/dbus
```
Press CTRL+O to save the file, then press Enter to confirm. Finally, press CTRL+X to close the Nano editor.

Now close you Linux terminal and open a new one.

c) Prepare dbus to tcp use instead of sockets (to add pain to injury, we use vim to edit - you can change with nano if you want).
```bash
$ cd /usr/share/dbus-1 && sudo vim session.conf
```
(When session.conf opens in vi editor)
Press i to enter insert mode and add the following (this is to preserve the original command)
```html
<!-- <listen>unix:tmpdir=/tmp</listen> || Original Command --> 
```
and add:
```html
<listen>tcp:host=localhost,bind=0.0.0.0,port=0</listen>

<!-- note that <EXTERNAL> is used wiht Unix socket and for windows should probably be commented out -->
<!-- check the instructions in the session.conf file -->

<auth>EXTERNAL</auth>
<auth>DBUS_COOKIE_SHA1</auth>
<auth>ANONYMOUS</auth> 
```
pres ESC key when done and :wq to save and exit.
Restart the Linux console and pray that it will work.

#### Optional 2 - installing a browser 
(actually this should not be optional as VSC need browser for help functions and for downloads). This is not really working.
```bash
~$ sudo apt install firefox
```
#### Bonus - install sublime-text (this seems to work better)
```bash
~$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -

~$ echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list

~$ sudo apt-get update
~$ sudo apt-get install sublime-text
```
If everything goes well, just run it:
```bash
~$ subl .
```
![sublime](https://raw.githubusercontent.com/Adrian-LL/vscode-cpp-setting-trials/master/images/sublime.JPG)


## (Unfortunate) Conclusions
Keeping into account where we started - wanted to learn /use C++ via console programs; wanted to have a helpful editor (with autocompletion and help if possible); wanted to install / configure as little as possible, the result is that I clearly overcomplicated, but also learned a lot in the process...

- **The easiest solution is still that presented in Part 1 of this series**. (Second is that Part 2, and in the 3rd and almost unusable place, the graphical apps under WSL).
- **Running VSC in WSL is not really usable** (and too complicated). Sometimes it even has a bad redrawing and the solution is to kill it.
- **Running (complex) graphical apps under WSL is still a fad for now**. There are some that work (such as sublime-text). 
- Power (good CPU) and memory is needed for some speed.
- **The C/ C++ debugging does not really work as expected**. The best usage is still from Part 1 (Windows only - except possibly when using Microsoft native tools - compiler and debugger). 
- Keep in mind that VSC is still an editor with IDE capabilities, so if you really need it, you should use a full-fledged one.
- **`lldb` (of `llvm` fame) does not work at all under WSL** (at least for C++) as it seems to use some semaphores from the kernel. It may work in the future.
