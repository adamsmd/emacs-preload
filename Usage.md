# Usage

Usage: `emacs-preload [options] <subcommand> ...`

TODO description

TODO: TL;DR basic usage

TODO: Environment variables (help subcommand?) Booleans # True values are 'y',
'yes', 't', 'true', 'on', and '1'; # false values are 'n', 'no', 'f', 'false',
'off', and '0'

TODO: flags must be before sub-command

# TODO: The default creates a frame, but this can (?) be overrided by passing
`-nw`

# TODO: document bool parsing

# The most common usage is simply: #   emacs-preload connect ARGS ... # where
ARGS are arguments to be passed to `emacsclient` (e.g., files to edit).

# Data structures: #  ~/.emacs.d/preload/preload-XXXX #
/tmp/emacs1000/preload-XXXX #  emacs server

# Technical details: # Emacs deamons put a socket file in
`/tmp/emacs$UID/$NAME` # This script puts in ~/.emacs.d/preload/ symlinks to
the socket fild of the servers it starts. # When this script connects to a
server, it removes that symlink # Thus all the symlinks in ~/.emacs.d/preload/
represent unused servers that we can connect to. # # When the number of
symlinks in ~/.emacs.d/preload/ is low, # this script launches more servers in
the background.

# # There are a few ways things can break: #  (1) The server could shut down #
(2) The socket file could be removed #  (3) The symlinks in
~/.emacs.d/preload/ could be removed # # Also, any combination of the above
three could happen.

## About 150ms delay if not already have these
##EMACS_PRELOAD_LINK_DIR=$HOME/.emacs.d/preload
##EMACS_PRELOAD_SOCKET_DIR=/tmp/emacs$UID ##    "~/.emacs.d/") ##
(format "%s/emacs%d" (or (getenv "TMPDIR") "/tmp") (user-uid)))

TODO: help should support printing results of --help TODO: include readme

##Running `emacs-preload check` ##prints the current configuration under
"Configuration variables".  You can



optional arguments:
----|----
-h, --help  | Show this help message and exit
-v, --version  | Show program's version number and exit
--dry-run  | Print commands instead of executing them (default False)
--no-dry-run  | Disable --dry-run
--quiet  | Supress info messages and startup messages from emacs servers (default False)
--no-quiet  | Disable --quiet
--query-emacs  | Call emacs to determine LINK_DIR and SOCKET_DIR. Off by default for performance. (default False)
--no-query-emacs  | Disable --query-emacs
--size SIZE | Number of servers to keep running (see `start` and `stop`) (default 7)
--emacs FILE | The emacs binary to use (default emacs)
--emacsclient FILE | The emacsclient binary to use (default emacsclient)
--ps FILE | The ps binary to use for process listing (default ps)
--patch-server  | Patch the emacs server to remove the quirks of server mode (default True)
--no-patch-server  | Disable --patch-server
--start-delay FLOAT | Seconds to wait between starting servers (default 0.1)
--stop-delay FLOAT | Seconds to wait between stopping servers (default 0.1)
--kill-delay FLOAT | Second to wait between SIGTERM and SIGKILL when killing servers (default 1.0)
--connect-delay FLOAT | Seconds between attempts to connect to a server (default 0.1)
--connect-attempts INT | Maximum number of attempts to connect to a server (default 100)
--connect-timeout FLOAT | Maximum seconds to keep attempting to connect to a server (default 10.0)
--prefix STR | Prefix to use for server names, links, and sockets (default preload-)
--link-dir DIR | Directory in which to place links (default /home/adamsmd/.emacs.d/preload)
--socket-dir DIR | The directory in which emacs stores its sockets (default /tmp/emacs1000)

subcommands:
----|----
run | Main `emacs-preload` subcommand.  Connect to a preload server and in the background start new servers.
check | Check for errors
start | Start preload servers
stop | Stop preload servers
status | Print preload status
init | Create the directory holding links to preload servers
connect | Connect to a preload server
cleanup | Cleanup broken servers
kill-orphans | Kill orphaned servers
kill-all | Kill all servers
export-wiki | Generate wiki version of all usage messages
help | Prints help for subcommands


## run

Usage: `emacs-preload [options] run`

Main `emacs-preload` subcommand. Uses `connect` to connect to an available
preload server, while using `start` in the background to launch new servers.

NOTE: Depending on how the `connect` and `start` processes are scheduled, the
number of servers left running may be one fewer than specified.

Arguments | 
----------|-
`args` | Arguments to `emacsclient`.  Not parsed by `emacs-preload`.  Passed unchanged to `emacsclient`.

## check

Usage: `emacs-preload [options] check`

Print preload configuration and check for errors in links, sockets, servers,
or clients.

## start

Usage: `emacs-preload [options] start`

Start new preload servers until at least `size` servers are running.

Arguments | 
----------|-
`size` | Target number of servers to have running.  Defaults to `SIZE` if omitted. Relative when starts with `+` or `-`.  Absolute otherwise.

## stop

Usage: `emacs-preload [options] stop`

Stop running preload servers until at most `size` servers are running.

Arguments | 
----------|-
`size` | Target number of servers to have running.  Defaults to zero if omitted. Relative when starts with `+` or `-`.  Absolute otherwise.

## status

Usage: `emacs-preload [options] status`

Print status of preload configuration, links, sockets, servers, and clients.

## init

Usage: `emacs-preload [options] init`

Create the directory specified by `LINK_DIR`, which is where links to preload
servers are kept.

Rarely needs to be called manually as it is automatically called by the
subcommands that need the `LINK_DIR` directory.

## connect

Usage: `emacs-preload [options] connect`

Connect to an available preload server.  Picks a waiting server from those in
`LINK_DIR`, removes it from `LINK_DIR`, and connects to it with `emacsclient`.

Arguments | 
----------|-
`args` | Arguments to `emacsclient`.  Not parsed by `emacs-preload`.  Passed unchanged to `emacsclient`.

## cleanup

Usage: `emacs-preload [options] cleanup`

Cleanup broken servers.

Normally emacs-preload cleans up after itself automatically.  If it doesn't,
you can use this command to manually cleanup.  We are working to reduce how
often this command is needed.

This command is always safe.  It will not close servers with connected clients
or fresh servers waiting for a client.  Because this subcommand is guaranteed
to be safe, it leaves alone servers for which it cannot confirm that there are
no clients.  Thus you may need to use kill-orphans or kill-all subcommands to
kill such servers.

Removes links in `LINK_DIR` for unreachable servers and servers that already
have clients.  Removes sockets in `SOCKET_DIR` that do not connect to a
server.  Sends `(kill-emacs)` to servers that have no client and are not in
`LINK_DIR`.

## kill-orphans

Usage: `emacs-preload [options] kill-orphans`

Kills all orphan preload servers, which are those for which there are no
sockets in `SOCKET_DIR`.

Normally emacs-preload cleans up after itself automatically.  If it doesn't,
you can use this command to manually cleanup.  We are working to reduce how
often this command is needed.

WARNING: Disconnects any `emacsclient` connected to orphan preload servers.
This is not part of `stop` or `cleanup` since there is no way to tell if the
servers skipped by `cleanup` have connected clients.  This command should only
be used when `cleanup` to fail to shutdown a server.

## kill-all

Usage: `emacs-preload [options] kill-all`

Kills all preload servers.

Normally emacs-preload cleans up after itself automatically.  If it doesn't,
you can use this command to manually cleanup.  We are working to reduce how
often this command is needed.

WARNING: Disconnects any `emacsclient` connected to  preload servers.  This is
not part of `stop` or `cleanup` since there is no way to tell if the servers
skipped by `cleanup` have connected clients.  This command should only be used
when `cleanup` and `kill-orphans` to fail to shutdown a server.

## export-wiki

Usage: `emacs-preload [options] export-wiki`

Create BitBucket formatted wiki files for all the usage messages

Arguments | 
----------|-
`file` | The file in which to put the usage messages

## help

Usage: `emacs-preload [options] help`

None

Arguments | 
----------|-
`subcommand` | The subcommand for which to print help (choose from `run`, `check`, `start`, `stop`, `status`, `init`, `connect`, `cleanup`, `kill-orphans`, `kill-all`, `export-wiki`, or `help`)
