# Emacs Preload: Fast Emacs Startup by Preloading Emacs Servers

This program is a drop-in replacement for Emacs that reduces start
times by preloading Emacs server instances in the background.

## Examples

If you do not care about the details, just use `emacs-preload run`
instead of `emacs`, and the tool will take care of the test.

The first run after a boot will be as slow as a normal `emacs`
startup as there are no servers preloaded yet.  After that, the
pool is automatically repopulated, and later runs should be much
faster.

You can avoid this initial slowness by populating the server pool
in advance with `emacs-preload start`.  In fact, `run` populates
the pool by running `start` in the background.

Any argument that can be passed to `emacsclient` is accepted.
Common use cases include:

- `emacs-preload run`

- `emacs-preload run -nw`

- `emacs-preload run file.txt`

An invocation of the form `emacs-preload run <args ...>`
corresponds to the following where `<name>` is internally
determined by `emacs-preload`:

- `emacsclient --quiet --socket-name <name> --create-frame <args ...>`

## Download

This project is hosted at
<https://bitbucket.org/adamsmd/emacs-preload>.

You can download the latest release from
<https://bitbucket.org/adamsmd/emacs-preload/addon/com.releasebucket/releases>.

Or you can clone the repository with either of the following.

```hg clone https://bitbucket.org/adamsmd/emacs-preload```

```hg clone ssh://hg@bitbucket.org/adamsmd/emacs-preload```

## Installation

This program is a self contained python script.  Everything in the
`emacs-preload` file.  Just copy it to wherever you want it installed and mark
it as executable (e.g., `chmod a+x emacs-preload`).

### Dependencies

Dependencies have intentionally been kept minimal.  They are:

- Python 3.5 or higher (`python3`)
- Emacs (`emacs`)
- Emacs client (`emacsclient`)
- `ps` (the process listing command usually installed on Unix variants
  including Linux and MacOS)

The paths to the last three can be configured if they are in a non-standard
location.

### Porability

The implementation uses Unix-specific features of Python.  It should run fine
under both Linux and MacOS.

There is no Windows support, but volunteers are welcome to add it.

### Configuration

While the default configuration should work for most cases, there are several
settings you can adjust to fit your needs.  For details, see `emacs-preload
--help` or [Usage.md](Usage.md) in the repository.

## Documentation

Documentation is available from `emacs-preload help` or [Usage.md](Usage.md)
in the repository.

## License

GPL version 3.0.  For details, see [LICENSE.txt](LICENSE.txt) in the
repository or online at <https://www.gnu.org/licenses/gpl.txt>

## Reporting bugs

Submit bugs to the issue tracker at
<https://bitbucket.org/adamsmd/emacs-preload/issues>.

My spare time is very limited and this is not my primary project, so bug
reports that include pull requests are more likely to be fixed.

Please include the output of `emacs-preload status` in your bug report.

If you cannot run `emacs-preload status`, include the versions of the
following:

- Emacs preload: `emacs-preload --version` (include the full version and SHA-1 hash)
- Python: `python3 --version`
- Emacs: `emacs --version`
- Emacs client: `emacsclient --version`

If you cannot run `emacs-preload status`, you can find the version number in
the `__version__` variables set at the top of the `emacs-preload` script, and
you can compute the SHA-1 hash manually with `openssl dgst -sha1
emacs-preload`.

## Contributing

For development guidelines, see [Development.md](Development.md) in the
repository.

## Frequently Asked Questions (FAQ)

### Why not `emacsclient`?

Internally this tool is built on `emacsclient`.  The difference is that the
usual `emacsclient` setup has multiple clients all connecting to the same
Emacs server, and thus buffers are shared between all of them.

In contract, `emacs-preload` manages a pool of servers, and each client
connects to a different server.  Thus buffers are not shared between them.

### Why isn't this project updated frequently?

Unfortunately, my time for side projects like this is very limited.  That
said, if an enterprising developer wants to take on maintenance or further
development, I would be happy to accept pull requests.

### How fast is the startup?

On my machine (Ubuntu 16.10 on a 2.6GHz Intel Core i7-6700HQ), I get the
following times.  All numbers measured in seconds as reported by `bash`'s
`time` built-in.

command                                           | real   | sys    | user
--------------------------------------------------|--------|--------|-------
`emacs-preload init`                              | 0.0933 | 0.0076 | 0.0836
`emacs-preload --query-emacs init`                | 0.2336 | 0.0328 | 0.1948
`emacs-preload connect --eval '(kill-emacs)' -nw` | 0.1682 | 0.0124 | 0.0708
`emacs-preload connect --eval '(kill-emacs)'`     | 0.9800 | 0.0088 | 0.0728
`vim -c q`                                        | 0.1395 | 0.0048 | 0.0328
`gvim -c q`                                       | 0.3329 | 0.0108 | 0.0552

Points of note:

1. These times are independent of normal `emacs` startup time.  The size and
   complexity of your `.emacs` does not affect it.

2. The script itself has low overhead as seen with `emacs-preload init`.
   These times are mostly the cost of starting `emacsclient`.

3. The `--query-emacs` option significantly increases this overhead.  It is
   recommended to leave it off (as it is by default) unless you really need
   it.

4. Terminal mode (i.e., with `-nw`) is almost as fast as `vim`.

5. GUI mode (i.e., without `-nw`) is 2.5 times slower than `gvim`.

6. While it would be nice to further improve these times, `emacs-preload`
   achieves reasonably short startup times.  Further improvements would have
   to be to `emacsclient` itself.

### How does it work?

See [Implementation.md](Implementation.md) in the repository.
