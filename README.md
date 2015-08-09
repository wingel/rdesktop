I've been using [rdesktop](http://www.rdesktop.org/) for quite some
time.  It's a nice little program that can connect to a Windows
machine using the remote desktop (RDP) protocol so that I can view the
Windows screen on my Linux machine.

I've mostly been running rdesktop it's own workspace covering the
whole screen.  For anyone looking over my shoulder it would look like
I'm running Windows, there are no visible traces of the MATE (Gnome)
desktop.  I turn on keyboard "grab" in rdesktop, which means that if I
press any of the key combinations that are normally handled by the
Marco (metacity) window manager, they are sent to the Windows machine
instead: Alt-TAB switches windows on the Windows machine instead of
switching away from rdesktop.

To be able to switch to different desktops it has to be possible to
"ungrab" the keyboard somehow.  Normally that is done by moving the
moue outside of the rdesktop window, but when rdeskop is covering the
whole screen that won't work any more.  Instead I added a patch to
rdesktop that adds an option "-U" which lets me press Ctrl-Alt to
"ungrab" the keyboard.  That means that I can press Ctrl-Alt, release
the keys and then press Ctrl-Alt-Down to move away from the desktop.
It sounds complicated but works really well in practice.

This command line is what I used.  "-D" to remove all window
decorations such as the title bar and borders, "-U" to turn on my
"ungrab" patch and "-g 100%x100%" to make the rdesktop window cover
the whole screen.

    rdesktop -D -U -g 100%x100% hostname

All this worked really well for a long time.  Until something changed
in the window manager which made the Gnome panels move to the top
which in turn means the Windows system tray would be hidden below the
Gnome panels.  I didn't have time to figure how to fix this at the
time, instead I started using rdesktop in "workarea" mode where the
rdesktop window is sized to the area between the panels instead:

    rdesktop -D -U -g workarea hostname

It was really ugly with two panels, both the MATE and the Windows one.
It also confused some Windows programs that assumed that the screen
must be at least 1024x768 pixels large and on my laptop I would end up
with a windows screen which was about 730 pixels high and buttons
would end up hidden below the bottom of the screen.  But it had to
make do.

Using the "-f" flag to get fullscreen mode in rdesktop didn't work
either.  If I use that the window is ignored by the window manager and
shows up on top of all other windows on all workspaces.  Maybe good
for a computer in kiosk mode, not as useful when I want a convenient
way of switching between the workspace with the Windows desktop and
the MATE desktop.  The reason rdesktop behaves this way is that it
turns on the "override redirect" flag on its window.  Override
redirect tells the Window manager to totally ignore the window and is
intended for short lived windows such as menus or tool-tips.

The proper way to do it, which I just figured out, is to stop using
override redirect and set the window property
"_NET_WM_STATE_FULLSCREEN" instead.  This hint tells the window
manager be aware of the window and do the proper thing when switching
workspaces, but to otherwise leave the window alone.  I've made an
ugly patch to rdesktop which probably breaks other things, but it
works well enough for my use case.  So I can now run rdeskop in full
screen mode and still have proper workspace switching:

    rdesktop -D -U -f hostname

That's one small itch scratched.  Only a couple of hundred left to go.

The maintainer of rdesktop didn't want to take my ungrab patch when I
mailed it to him a long time ago and considering that the fullscreen
patch is a hack which probably breaks other things I definitely
wouldn't expect him to take that one.  But on the off chance that
anyone would find my modified version of rdesktop useful, the source
can be found on [github](https://github.com/wingel/rdesktop).
