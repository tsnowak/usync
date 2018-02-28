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

To install `unison-2.40` on ubuntu we need to reach back into the bowels of the
ubuntu package archive.
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
make UISTYLE=text
```
This should work, but honestly I haven't tested it :D .

Whew, that was annoying, but hey, we're making progress despite having to drag
you silly Mac users along!

## Linux and WSL from Source

If you're a hipster and want to install from `unison-2.40` from source on
Linux~~/WSL~~ and not WSL, just follow the Mac OSX instructions except
`sudo apt-get install ocaml`.

If you want to install `ocaml` from source, or if you want to figure out why
unison is failing to build on WSL more power to you, but you're on your own from
here on out buckaroo.

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

# References
- [The Unison Manual](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html)
