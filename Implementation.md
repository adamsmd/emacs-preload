# Implementation Details

## Starting a server (`emacs-preload start`)

The `emacs-preload start` command is responsible for preloading servers.  It
builds on `emacs --daemon`.

Starting an Emacs server causes a socket file to be created that `emacsclient`
can use to connect to that server.  These socket files are stored in
`SOCKET_DIR` (default: `/tmp/emacs$UID` where `$UID` is the current user id
number).

On top of this `emacs-preload` stores links to these socket files in
`LINK_DIR` (default: `~/.emacs/emacs-preload`).  This directory is
automatically created by `emacs-preload` subcommands that need it, or it can
be created manually with `emacs-preload init`.

The filenames in these two directories are made up of `PREFIX` (default:
`emacs-preload-`) followed by a sequence of randomly chosen characters.  Any
file that does not start with this is ignored.

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

## Restarting servers (`emacs-preload restart`)

The `emacs-preload restart` command uses `emacs-preload stop` to stop the
servers with links in `LINK_DIR`.  It then uses `emacs-preload start` to start
the same number of new servers.  This is useful in development and testing to
make sure no servers with old code are running.

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
corrupt, the system ends up in an inconsistent state.

You can check if this has happened with `emacs-preload check`.  If it has,
your first recourse is `emacs-preload cleanup`.  It removes links, sockets, or
servers that are in broken states.  However, it careful to never remove these
for servers that have connected clients.  It leaves alone servers for which it
cannot confirm there are no clients.  Thus it is always safe to run but
sometimes cannot cleanup all servers.  Note that to shutdown a server
`eamcs-preload cleanup` sends `(kill-emacs)` to the server, whereas
`emacs-preload kill-orphans` and `emacs-preload kill-all` both send `SIGTERM`
and `SIGKILL` signals.

In order to determine how many clients a server has, `emacs-preload cleanup`
uses `emacsclient --eval` to query the server.  Thus, if the socket to connect
to the server does not exist or cannot be used to connect.  Dealing with this
is where `emacs-preload kill-orphans` and `emacs-preload kill-all` come in.
They are unsafe in that they may shutdown servers with connected clients, but
they can also shutdown servers that `emacs-preload cleanup` cannot.

The `emacs-preload kill-orphans` command shutsdown any preload server that
cannot be connected to, whereas the `emacs-preload kill-all` command shutsdown
all preload servers.

Both these commands determine which processes are preload servers by looking
in the output of `ps` for an `emacs` command started with a `--daemon` flag
pointing to a socket in `SOCKET_DIR` that has a filename starting with
`PREFIX` (default `emacs-preload-`).

Both these commands shutdown servers by first sending a `SIGTERM` signal and
then `KILL_DELAY` seconds later a `SIGKILL` signal.

In general you should run `emacs-preload check` before running either
`emacs-preload kill-orphans` or `emacs-preload kill-all` to double check which
servers would be killed.

**Warning:** The `emacs-preload kill-all` kills all preload servers so any
`emacs-preload run` or `emacs-preload connect` that are open will shutdown,
and any data in unsaved buffers may be lost. Use this command with caution.
