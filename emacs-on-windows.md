# How I Use Emacs on Windows

Because my job traps me on Windows 90% of the time, I've had to adapt my Emacs
configuration to get everything working well. Who knows if this will be helpful
to anyone else, but I figured I'd describe it.

Everyone does things differently, and since this file is titled "How I Use
Emacs On Windows", I'll be describing how *I* do things rather than some
idealized best practice.

This should all work with the default distribution of Emacs, using tools that
you install natively, or through a manager/environment like MSYS, Chocolatey,
or (I think?) Cygwin. No guarantees on that last one, since I haven't used it
in years.

If you wanna use WSL, you're on your own :)

## How I Organize Things

I've been carrying the same `home` folder with me for >20 years. It's organized
the way you might expect:

    E:\Home ->
                \Applications
                \Archive
                \Inbox
                \Media
                \Office
                \Org
                \Site
                \Work
                \.emacs.d
                \.tmp

As well as any other configuraton files or folders, which are typically hidden.
Everything lives in Home, and it's backed up all over the place. Parts of this
folder are selectively synced between machines, as appropriate. I use Nextcloud
for this, but you can use anything you like.

# Setting System Variables

There are a few system variables I set. In Windows 10, you can set these by
right-clicking on the Start menu, picking "System" from the list, clicking
"Advanced System Settings" on the left side of the System window, clicking the
"Advanced" tab, and then the "Environment Variables..." button. They really
buried that thing! If you have trouble finding it, just search the internet a
bit, and you'll find instructions to get there on your version of Windows.

In the Environment Variables table, I create:

- `HOME` and set it to `E:\Home`

I then modify `path` to include anything I want Emacs (and Windows generally)
to be able to find, such as:

- `e:\Home\Applications\Emacs\bin\`
- `e:\Home\Applications\Everything\`
- `e:\Home\Applications\aspell\`
- `e:\Home\Applications\msys2\bin`
- `e:\Home\Applications\msys2\mingw64\bin`
- `e:\Home\Applications\msys2\mingw32\bin`

You can add whatever makes sense to you -- the important thing to keep in mind
is, a lot of open source tools don't automatically update Windows' `path`
variable, so when I install anything from the internet, I make
a habit of checking to make sure it got added to the path correctly.

## What Setting These Variables Accomplishes

By setting these variables, you can define where Emacs thinks `Home` is for
you -- and because `Home` is where Emacs looks for your init, you can now just
put all of your configuration stuff into your home folder, as in:

- `e:\Home\.emacs.d\`
- `e:\Home\.gnus`
- `e:\Home\.authinfo`

And so on. This is vastly easier than having to custom set the location for
each file in your init.

This also makes the file-finding and completion in Emacs work more sanely. You
can, for example, find file, hit `~\Org\somefile.org` and it'll work as
expected.

And, of course, you can use `(getenv "home")` in your config and have that
do something useful.

# Daemon-Client Setup in Windows

Okay, so you've got Emacs looking in the right place for your init, and your
system knows where to find all the binaries you need Emacs to be able to use.
Now let's talk about getting a Daemon working in Windows.

It's actually pretty easy! We're going to use the built-in shortcut
functionality of windows to automatically start a Daemon on boot, and create a
shortcut that will either connect to that Daemon, or start a new instance if no
Daemon is running.

**NB:** For all of these, once they're created, I go back in and set the "Start
in:" location to `E:\Home\`

## Creating a Daemon

**Step 1:** Create a shortcut with the following target:

`E:\Home\Applications\Emacs\bin\runemacs.exe --daemon`

And give it a name like `GNU Emacs Daemon`

**Step 2:** Copy the shortcut to the Start Menu

`C:\Users\[USERNAME]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\`

**Step 3 [Optional]:** To start at boot, create a copy of the shortcut in the startup folder

`C:\Users\[USERNAME]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

## Creating a Client

**Step 1:** Create a shortcut with the following target:

`E:\Home\Applications\Emacs\bin\emacsclientw.exe -n -c -a "E:\Home\Applications\Emacs\bin\runemacs.exe"`

This shortcut first tries to connect using `emacsclient` and if that fails,
falls back to `runemacs`.

Give the shortcut a name like `GNU Emacs`

**Step 2:** Copy the shortcut to the Start Menu

`C:\Users\[USERNAME]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\`

### The One True Emacs Shortcut

This is your primary shortcut for interacting with Emacs. I have this pinned to
the Start Menu, and the Taskbar.

## Creating a Safe Start Shortcut

I also created a shortcut labeled `GNU Emacs Safe Start`, which has the target:

`E:\Home\Applications\Emacs\bin\runemacs.exe -Q`

So I can load Emacs without my init as needed.

## Creating a Debug Shortcut

And finally I created a shortcut labeled `GNU Emacs Debug`, which has the
target:

`E:\Home\Applications\Emacs\bin\runemacs.exe --debug-init`

Which allows me to run the `debug-init` stuff easier, when needed.

# Opening Filetypes with Emacs by Default

Let's say I want to open all `.el` files with Emacs by default, and I want it
to do so by connecting to my Daemon (or starting a new instance if there's no
Daemon running).

**Step 1:** Right-click on a file of the type you want Emacs to open

**Step 2:** Hover over "Open with.." and pick "Choose another app"

**Step 3:** Ignore all the options and look for "More Apps"

**Step 4:** *This is the kicker.* Don't find the `.exe` for Emacs, go find the
`GNU Emacs` shortcut in your Start Menu, and use that.

Now when you open the file it'll do so using your shortcut, which makes this
all work as expected.

# Universal "Summon Emacs" and "Org Capture" Keyboard Shortcuts

It's not enough to just have Emacs running -- I like being able to call Emacs
with a keyboard shortcut from anywhere. To do that, I again use (abuse?) the
built-in shortcut functionality in Windows.

**NB:** Keyboard shortcuts only work for shortcuts in the Start Menu
and on the desktop.

Remember those shortcuts we created? Go find the `GNU Emacs` shortcut in the
Start Menu, go to Properties, and look in the "Shortcut" tab.

In the bottom cluster of options, there's a field labeled "Shortcut key". Click
once into the box, then hit the desired key combination. I use `Ctrl + Alt +
F1` but you can use whatever. Apply, and close. Now hit your keyboard
combination, and the shorcut should launch.

The first time I set this up, Windows had trouble remembering I'd set the
shortcut, but it worked correctly after a reboot.

## Create an Org Capture Keyboard Shortcut

If you want a universal keyboard shortcut to capture something in Org, you'll
need to make one more shortcut as before, with the target:

`E:\Home\Applications\Emacs\bin\emacsclientw.exe -n -c -e "(progn (raise-frame)(org-capture))"`

Then, as above, save the shortcut into the Start Menu, and add the keyboard
shortcut.

# Other Stuff

Digging through my init, I noticed a couple odds and ends that may be helpful.

## Basic Windows Settings

I have the following in my init, which sets a few variables on Windows
machines. I tried to gather a couple things here, and set them in one place,
rather than scattering them throughout my init.

This sets up spellchecking with Aspell, and fixes Ghostscript so I can view
PDFs, among other things.

    (when (eq system-type 'windows-nt)
      (setq w32-allow-system-shell t) ; enables cmd.exe as shell
      (setq save-interprogram-paste-before-kill 1 ; stop killing my clipboard, plz
            ; ghostscript on windows
            ; see https://www.emacswiki.org/emacs/docviewmode for details
            doc-view-ghostscript-program "~/Applications/msys2/mingw64/bin/gswin32c.exe"
            ; set curl location
            request-curl "~/Applications/msys2/mingw64/bin/curl.exe")
      (cond ((executable-find "aspell") ; spell-checking
             (setq ispell-program-name "aspell")
             (setq ispell-extra-args '("--sug-mode=ultra" "--lang=en_us")))))

But you can always target code only at Windows machines by adding `when`
statement:

    (when (eq system-type 'windows-nt) ... )

## Drive Letters and Dired

I've got `dired+` installed, which allows me to switch local drives easier in
`dired` sessions.

You can also change drives when finding files by just typing, for example, `C:`
instead of a file name -- in `counsel`, at least, it works as desired.

## Info files from MSYS

You can help Emacs find your Msys `info` files by adding the following to your
init:

    ;; Info-mode (Info's gotta be capitalized in the variables!)
    (when (eq system-type 'windows-nt)
      (setq Info-additional-directory-list
            (quote
             ("~/Applications/msys2/usr/share/info" "~/Applications/msys2 /mingw64/share/info"))))

## Replacing `locate` with `everything` in `counsel`

The locate command is really great... if you're not on Windows. Since I use the
`ivy` \ `swiper` \ `counsel` stack, I've added the following to my init to use
the Windows search tool `everything` instead:

    (with-eval-after-load 'counsel
      (when (eq system-type 'windows-nt)
        (defun counsel-locate-cmd-es (input)
          "Return a shell command based on INPUT."
          (counsel-require-program "es.exe")
          (format "es.exe -r %s"
                  (counsel--elisp-to-pcre
                   (ivy--regex input t))))))

## Mail on Windows?

People love using Emacs for mail. Windows users cannot use any of the popular
tools like `mu4e`. What to do?

Use `gnus` as an IMAP client. It works fine. It's a pain in the ass to set up,
though, so it may not be worth it for you. If there's interest, I can explain
how I've got Gnus running to access my personal email and my corporate Office
365 email.

If there's a way to get the better mail tools working without WSL, I'd love to
hear about it.

# Stuff to Know

A couple things to know:

- When emacs can't find a tool you need, the tool is probably not in your `path`
- I've found Emacs on Windows to be pretty good about flipping the slashes on
  paths -- I can use `e:/Home/Applications/` in my variables and and Emacs doesn't bat
  an eye. In cases where you need to use Windows-style paths, you gotta escape
  the backslashes, as in: `E:\\Home\\Applications`
- Some packages need you to use the full path (i.e. `e:/Home/Org/...`) when
  customizing their variables rather than pinning them to `home` (i.e.
  `~/Erg/`. I know it's true of `image-dired` and a few other things.
- I've found I often need to add `.exe` to the names of commands to get Emacs
  to find them, as in `(counsel-require-program "es.exe")`

# Conclusion

And that's basically it. I've been very happy with Emacs on Windows, and don't
really feel like I'm missing out on any important functionality. It took a bit
of tweaking and reading docs or old Stack Exchange threads to figure out how to
get it all set up, but it was time well spent.
