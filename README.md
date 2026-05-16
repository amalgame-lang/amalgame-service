# amalgame-service

Pure-Amalgame service facade for [Amalgame](https://github.com/amalgame-lang/Amalgame).
**SIGTERM/SIGINT** signal handler + `ShouldStop()` polling +
interruptible `Sleep()` + **native Windows Service Control Manager
dispatcher** — the primitives every long-running daemon needs to
run cleanly under systemd, launchd, and the Windows SCM with no
external wrapper.

Originally bundled in amc's `src/stdlib/service.am`; extracted into
this external package as part of the framework split (post-v0.7.5).
v0.2.0 added the native Windows SCM dispatcher — `amc new --template
service` scaffolds no longer ship NSSM as a dependency.

## Install

```bash
amc package add service                  # via the curated index
amc package add github.com/amalgame-lang/amalgame-service@v0.2.0
```

Requires **amc 0.7.7+**.

## Surface

```amalgame
import Amalgame.Service

class Program {
    public static int Main(List<string> args) {
        Service.RunAsService("my-daemon")       // install signal handler + (Windows)
                                                // bootstrap SCM dispatcher on a
                                                // worker thread; same binary works
                                                // in console mode AND as a native
                                                // Windows service

        while (!Service.ShouldStop()) {
            // … do work …
            Service.Sleep(5000)                 // interruptible, returns
                                                // immediately on shutdown
        }
        return 0
    }
}
```

`Service.Install()` (no name) is the v0.1.0 entry point — installs
the OS signal/Ctrl handler only, no Windows SCM integration. Still
supported for the "console-only daemon" case and for embedding the
loop in a non-service process.

`Service.RequestStop()` is the programmatic shutdown trigger — same
flag the signal handler flips, useful from in-process callers (e.g.
an admin HTTP endpoint).

## Windows service registration

With v0.2.0, the binary is a real Windows service — register it
with the built-in `sc.exe`, no NSSM download required:

```powershell
sc.exe create my-daemon binPath= "C:\path\to\my-daemon.exe" start= auto
sc.exe start  my-daemon
sc.exe stop   my-daemon
sc.exe delete my-daemon
```

The same binary launched directly from a console keeps behaving
like a plain foreground process — `StartServiceCtrlDispatcher`
returns immediately with `ERROR_FAILED_SERVICE_CONTROLLER_CONNECT`
in that case, and `Ctrl+C` triggers the same `ShouldStop` flag.

## Tests

```bash
./tests/run_tests.sh /path/to/amc
```

## License

Apache-2.0 — see `LICENSE`. No vendored third-party code.
