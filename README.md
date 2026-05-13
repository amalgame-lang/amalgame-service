# amalgame-service

Pure-Amalgame service facade for [Amalgame](https://github.com/amalgame-lang/Amalgame).
**SIGTERM/SIGINT** signal handler + `ShouldStop()` polling +
interruptible `Sleep()` — the three primitives every long-running
daemon needs.

Originally bundled in amc's `src/stdlib/service.am`; extracted into
this external package as part of the framework split (post-v0.7.5).

## Install

```bash
amc package add service                  # via the curated index
amc package add github.com/amalgame-lang/amalgame-service@v0.1.0
```

Requires **amc 0.7.7+**.

## Surface

```amalgame
import Amalgame.Service

class Program {
    public static int Main(List<string> args) {
        Service.Install()                       // register SIGTERM/SIGINT handler

        while (!Service.ShouldStop()) {
            // … do work …
            Service.Sleep(5000)                 // interruptible, returns
                                                // immediately on shutdown
        }
        return 0
    }
}
```

`Service.RequestStop()` is the programmatic shutdown trigger — same
flag the signal handler flips, useful from in-process callers (e.g.
an admin HTTP endpoint).

## Tests

```bash
./tests/run_tests.sh /path/to/amc
```

## License

Apache-2.0 — see `LICENSE`. No vendored third-party code.
