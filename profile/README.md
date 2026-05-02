# LynxDB

Two things have always been true about logs:

1. They are noise. Until they aren't.
2. By the time you know what you're looking for, you've already thrown away the answer.

I started building LynxDB because nothing I tried sat in the right place between two
extremes.

On one side: `grep`, `awk`, `jq`. Beautiful, fast, scriptable. But stateless. When the
incident is over, the answer evaporates with the shell session.

On the other side: Splunk, Elasticsearch, Datadog. Capable, but heavy. Six nodes to
keep the lights on. JVM heap tuning. Index mappings you have to get right *before*
you know what your data looks like. Pricing models that punish curiosity.

I wanted something in the middle. Something that respected the Unix tradition -
install it, pipe data through it, get an answer - but that also *remembered*. A tool
I could reach for at 2 AM during an incident and still be using on Monday morning to
power a dashboard.

## The idea behind it

Most log systems make you decide what your logs *mean* at write time. You define a
schema. You map fields. You commit. A week later something breaks in a way you didn't
anticipate, the field you really need is buried inside an unparsed string, and you're
rewriting your pipeline at midnight.

LynxDB inverts this. Logs come in raw. They are stored raw. **Meaning happens at
read time.**

A log line is just a line. You can interpret it as JSON. You can pull fields out
with a regex. You can group it by a value that didn't exist yesterday. None of those
choices touch the underlying data, and none of them are permanent. You can have ten
different ways of looking at the same logs at the same time, change your mind
tomorrow, and never lose a byte of the original.

The unit of "meaning-making" in LynxDB is the **materialized view**. A view is a
*hypothesis* about your logs - a way of saying "if I read these lines through this
lens, here's what they tell me." You can spin one up in seconds. You can stack them.
You can throw them away. The raw data sits underneath, untouched, waiting for the
next question.

That's the whole point. The storage engine, the query language, the CLI - they all
exist so that people can *think with their logs, instead of preparing their logs to
be thought about.*

## What it feels like

```bash
# Pipe mode. No server. No config. Same as grep.
kubectl logs deploy/api | lynxdb query '| stats p99(duration_ms) by endpoint'

# Server mode. Same binary. Persistent.
lynxdb server

# Same query language in both.
lynxdb query 'level=error | stats count by service'
```

One binary does both. No JVM, no extra services, nothing to install on top of it. Run
it on a $5 VPS or scale it across a hundred nodes - the experience of *using* it
doesn't change.

## What I gave up to get here

A general-purpose database would have been easier to build. So would a thin CLI
wrapper around an existing engine. LynxDB is neither. The storage format, the
inverted index, the query planner, the materialized view runtime - all written from
scratch in Go.

That cost time. But it meant every layer could answer to one question - *does this
make log analysis feel light?* If something compromises that - a dependency, a config
flag, a step between "downloaded" and "useful" - it doesn't ship.

## Where it's going

LynxDB is early. The core works. The ideas above are real, not aspirational. But
there's a long road ahead: distribution, sharding, the web UI, a richer query
language, an MCP server so AI agents can analyze logs the same way I do.

I'm building this in the open because I want feedback from people who feel the same
friction I felt. If any of this resonates - install it, break it, tell me what's
wrong