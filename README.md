This is a script, based upon
[unison](https://www.cis.upenn.edu/~bcpierce/unison/), for syncing folders
between computers. If you attach it to a cronjob or a system startup file, it
can function similar to dropbox.

Below I run through how to install and use the script. Enjoy and don't hesitate
to let me know how I can make it better!

# Installing Unison

In my code, I use `unison-2.40`. At the time it was the only version that I could
acquire across platforms. That might have changed, but for the purposes of this
tutorial we're going to stick with it!

## Ubuntu Linux or WSL

To install `unison-2.40` binaries on Ubuntu we need to reach back into the
bowels of the Ubuntu package manager archive.
```
wget http://mirrors.kernel.org/ubuntu/pool/universe/u/unison/unison_2.40.102-2ubuntu1_amd64.deb
```
We can then install the resulting `.deb` file, and check that it works via:
```
sudo dpkg -i unison_2.40.102-2ubuntu1_amd64.deb
rm unison_2.40.102-2ubuntu1_amd64.deb
which unison
```
which should return `/usr/bin/unison`. If so, splendid!

Note for Windows we've decided to only speak in terms of WSL (because it makes
life oh so much simpler). But, if you're a hipster you may love CYGWIN. You
can find some CYGWIN install instructions on
[the unison website](https://www.cis.upenn.edu/~bcpierce/unison/download.html).

## Mac OSX

For those Mac OSX users, unison binaries ~~are also available through Homebrew~~
are no longer available through Homebrew. You're now going to have to compile
the source yourself (serves you right!).
```
git clone git@github.com:bcpierce00/unison.git
cd unison
git checkout unison-2.40
brew install ocaml
```
Then, according to this
[somewhat obscure github question](https://github.com/bcpierce00/unison/issues/18)
 we need to go to line 45 of `unison/src/mkProjectInfo.ml` and change
```
$Rev $
```
to
```
$Rev: 600$
```
Don't ask me why, I have no bloody idea. But, now we can proceed to compiling
our beloved unison!

```
make UISTYLE=text
which unison
```
Here we tell it to make the text only version (non-GUI) and then check that
it exists.

Whew, that was annoying, but hey, we're making progress despite having to drag
you silly Mac users along!

## Linux and WSL from Source

If you're a hipster and want to install from `unison-2.40` from source on
Linux/WSL, just follow the Mac OSX instructions except
`sudo apt-get install ocaml` instead of `brew install ocaml`.

If you want to install `ocaml` from source more power to you, but you're
on your own from here on out buckaroo...

# PRF Files

The `.prf` files are used to specify the folders and file paths to be synced,
and also to specify options pertaining to the sync.

## Root Directories

In my sample `work.prf` file you will first see:
```
# local, current computer root dir
root = /home/mylocalname/
# server root dir
root = ssh://myremotename@myserver//path/to/storage/area
```
This specifies the root file paths of the sync. This means that when unison
syncs, it will sync the folders (`Documents`, `Work`, and `Teaching`
specified later on) between `/home/mylocalname` and remotely on
`myremotename@myserver` at `/path/to/storage/area`.

## Ignores

There are also some files that we really don't want to waste our time syncing.
These are specified by ignore directives such as:
```
ignore = BelowPath .git
ignore = Name *.so
ignore = Name *.o
```
Here we say, "Don't sync anything below a `.git` folder, and don't sync
anything of file type `.so` or `.o`." There are many types of files that you might
want to ignore, and you can always add them here.

## Further Options

Here we specify a few different options:

`perms=0`: Don't transfer file permissions.

`auto = true`: Automatically accept non-conflicting changes.

`silent = true`: Print nothing except error messages.

`times = true`: Sync modification time stamps.

`log = true`: Keep a log file of updates.

`logfile = /home/mylocalname/.unison/unison.log`: Specified the location and
name of the log file.

## Paths to Sync

Finally, here is where we actually specify which folders below `root` that we
actually want to sync.
```
path = Documents
path = Work
path = Teaching
```

# Using Usync

## Move to bin

To most easily use `usync`, I'd recommend copying it to your `/usr/local/bin`.
```
sudo cp usync /usr/local/bin
sudo chmod +x /usr/local/bin/usync
```

## Usage

```
Usage:
[-d]            Used to sync the Default file work.prf.
[-f] [FILENAME] Used to sync via a specified PRF file.
[-h]            Used to print flag options.
```

At the top of the usync file you will see the DEFAULT variable. You may change
this to specify the `.prf` file used by default `[-d]`.

With the `-f` option you can specify the `.prf` from which to sync on the
fly.

## Attaching to a cronjob

Run the following and select your favorite editor (3 m'lord!).
`crontab -e`

Then just add a line for your usync. For example, let's say we want it to sync
our files everyday at 5AM using the `work.prf` file.
```
0 5 * * * usync -f work
```
Blamo! It's done, and your computer will now sync your stuff to your server
every day at 5AM.

# To Infinity, and Beyond!

## Hold Version

Odds are you'll probably want to keep using unison into the future, but if you
do an `apt-get upgrade` on Ubuntu Linux/WSL unison will get upgraded to the
latest version, which will break the system (unison doesn't work with  
different versions in your sync cluster). To remedy this, we'll want to put
unison on hold
```
sudo apt mark hold unison
```
If you installed unison from source, no package manager is managing it, so you
don't need to worry about this.

## SSH Username/Computer Persistence

If you're kinda new to Unix, you might be annoyed by having to enter your
password each time unison syncs. Well that's too bad and you'll just have to get
over it you lazy... just kidding.

I've fortunately made some handy-dandy scripts to help make managing ssh
configurations easier. ~~I haven't uploaded those yet, but soon!~~ You can find these scripts [here](https://github.com/tsnowak/ssh-tools).

Now some might ask, "Ted, why do I need ssh management tools, I have two hands
and a brain full of unbridled knowledge and zeal?" Well let me tell ya'. If you
work on a lot of computers, it can become tedious to make an entry in the
`.ssh/config` across all previous machines for each new machine. Additionally,
you'll need to make ssh keys for each machine, and put them in the
`authorized_keys` file on every other existing machine. Anyways, for the
algorithmically minded I think the complexity becomes `O(n^5)` and that's just
too much `n` for me, you know what I'm sayin'? I'm more of an `O(n)` or `O(n^2)`
 on a good day kinda guy.

For now, until I get that code uploaded check out these docs:

* [Digital Ocean SSH Config Tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client)

# References
For further information:

* [The Unison Manual](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html)
