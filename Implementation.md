# Implementation Details

## Starting a server (`emacs-preload start`)

The `emacs-preload start` command is responsible for preloading servers.  It
builds on `emacs --daemon` and `emacsclient`.


calling `emacs --daemon`
(which creates the socket file) and 

Starting an Emacs server causes a socket file to be created that `emacsclient`
can use to connect to that server.  These socket files are stored in
`SOCKET_DIR` (default: `/tmp/emacs$UID` where `$UID` is the current user id
number).

On top of this `emacs-preload` stores links to these socket files in
`LINK_DIR` (default: `~/.emacs/preload`).  TODO: init.
creating a link file pointed to that
socket.

The filnames in these two directories are made up of `PREFIX` (default:
`preload-`) followed by a sequence of randomly chosen characters.

Finally, 
start (also patches emacs see below)

## Checking and showing the status of servers (`emacs-preload check` and `emacs-preload status`)

check:

status:

## Connecting to a server (`emacs-preload connect` and `emacs-preload run`)

In order to connect to a server, `emacs-preload`
When this script connects to a server, it removes that symlink
Thus all the symlinks in ~/.emacs.d/preload/ represent unused servers that we can connect to.

run:
When the number of symlinks in ~/.emacs.d/preload/ is low,
this script launches more servers in the background.

## Stopping a server (`emacs-preload stop`)

stop

patch emacs

## When things go wrong (`emacs-preload cleanup`, `emacs-preload kill-orphans`, and `emacs-preload kill-all`)

There are a few ways things can break:
 (1) The server could shut down
 (2) The socket file could be removed
 (3) The symlinks in ~/.emacs.d/preload/ could be removed

Also, any combination of the above three could happen.

cleanup/kill-orphans/kill-all

