# saule-log

Logging for [Saule](https://github.com/lauriszz123/saule) programs: named
loggers, levels chosen from the environment or in code, and one readable line
per message, to the terminal, a file, or memory.

```
16:50:02.774 INFO  wm          started  screen=1024x600
16:50:02.774 INFO  wm.session  window opened  id=3 title="About This System"
16:50:02.774 WARN  wm.session  no settings service  reason="already published"
```

## Install

```sh
saule install lauriszz123@saule-log:main
```

or in `saule.config`:

```
dependencies: ["lauriszz123@saule-log:main"]
```

## Use

```saule
from log import Logger

local log = Logger("wm")
log.info("started", {screen: "1024x600"})

local session = log.child("session")   -- "wm.session"
session.debug("focus moved", {to: 3})
session.warn("no settings service", {reason: e})
```

Write a fixed message and put what changes in the fields: lines stay easy to
search (`grep "window opened"`), and the fields are printed sorted, as
`name=value`, quoted when they hold a space.

Levels, least to most: `Trace`, `Debug`, `Info`, `Warn`, `Error`, and `Off`.
A logger writes its level and above; the default is `Info`. Checking a level
is one comparison, so `debug` lines can stay in code that runs every frame;
for something costly to work out, ask first:

```saule
if log.enabled(Level.Debug) then
  log.debug("tree", {dump: tree.describe()})
end
```

## Settings

Nothing has to be set up: the first message reads the environment.

| Variable | Does |
|---|---|
| `SAULE_LOG=debug` | every logger at Debug |
| `SAULE_LOG=warn,wm=debug` | Warn, but `wm` and the loggers under it (`wm.session`) at Debug; the longest matching name wins |
| `SAULE_LOG_FILE=/tmp/app.log` | also append every line to that file |
| `NO_COLOR=1`, `SAULE_LOG_COLOR=0` | no colours on the terminal |

Lines go to standard error, so a program's own output stays clean.

In code:

```saule
Log.setLevel(Level.Debug)               -- every logger
Log.setLevel(Level.Trace, "wm.session") -- one, and those under it
Log.configure("warn,wm=debug")          -- as SAULE_LOG reads

Log.addSink(FileSink("/data/logs/wm.log"))
local recent = MemorySink(200)          -- the last 200, for a log window
Log.addSink(recent)
Log.clearSinks()                        -- the terminal too
```

A sink is anything that `implements LogSink` (`fn write(record: LogRecord)`).
`LogFormat.line(record, colour)` gives the line the built-in sinks write.

## Tests

```sh
saule test
```
