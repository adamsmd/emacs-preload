# Implementation Details

## Starting a server (`emacs-preload start`)

The `emacs-preload start` command is responsible for preloading servers.  It
builds on `emacs --daemon`.

Starting an Emacs server causes a socket file to be created that `emacsclient`
can use to connect to that server.  These socket files are stored in
`SOCKET_DIR` (default: `/tmp/emacs$UID` where `$UID` is the current user id
number).

On top of this `emacs-preload` stores links to these socket files in
`LINK_DIR` (default: `~/.emacs/preload`).  This directory is automatically
created by `emacs-preload` subcommands that need it, or it can be created
manually with `emacs-preload init`.

The filenames in these two directories are made up of `PREFIX` (default:
`preload-`) followed by a sequence of randomly chosen characters.  Any file
that does not start with this is ignored.

Finally, when a new preload server is started, its elisp code is patched so it
behaves more like a traditional Emacs instance.  For example, so it does not
automatically exit when all buffers have be closed.  This can be disabled by
setting `PATCH_SERVER` to false.

## Connecting to a server (`emacs-preload connect`)

In order to connect to a server, `emacs-preload connect` chooses a link in
`LINK_DIR` and connects to the socket file it links to.  Once successfully
connected, it remove the link from `LINK_DIR`.  Thus, servers that have a link
in `LINK_DIR` are waiting for a client, and servers without a link in
`LINK_DIR` are already connected to a client.  (In both cases, the server
should still have a socket in `SOCKET_DIR`.)  Note that the code has to be
careful to avoid a race condition here.

## Starting and connecting (`emacs-preload run`)

The `emacs-preload run` command behaves like `emacs-preload connect` except
that it also runs `emacs-preload start` in a background process.  This ensures
that the pool of preload servers is automatically repopulated.

## Stopping a server (`emacs-preload stop`)

The `emacs-preload stop` command stops the servers with links in `LINK_DIR` by
simply sending a `(kill-emacs)` message to the server.  It does not affect
servers that already have clients and thus do not have links in `LINK_DIR`.

## Checking and showing the status of servers (`emacs-preload check` and `emacs-preload status`)

The `emacs-preload status` command prints the status of the preload
configuration, links, sockets, servers, and clients.  It is useful both for
debugging and if you want to better understand the intermals of
`emacs-preload`.

The `emacs-preload check` command is a version of `emacs-preload status` that
suppresses most information unless there is an error.  It is useful for
figuring out if something has gone wrong.

## When things go wrong (`emacs-preload cleanup`, `emacs-preload kill-orphans`, and `emacs-preload kill-all`)

There are three parts of this system that can break: the Emacs server, the
socket file for that server, and the link file for that server.  If any of
these are shutdown or removed when they shouldn't or otherwise becomes
corrupt, TODO

TODO: `emacs-preload cleanup`

TODO: `emacs-preload kill-orphans`

TODO: `emacs-preload kill-all`
