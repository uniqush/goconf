# goconf (archived)

**This repository is archived and unmaintained. It is kept for history only.**

A parser for INI-style configuration files. This is a fork of
[ifwe/goconf][ifwe], which descends from Stephen Weinberg's goconf; the last
commit here is from **May 2012**. It existed to be [uniqush-push][uniqush-push]'s
config parser, and uniqush-push was its only importer.

uniqush-push now has this code in-tree at [`conf`][adopted], where the bug below
is fixed.

## Use the copy in uniqush-push

```go
import "github.com/uniqush/uniqush-push/conf"

c, err := conf.ReadConfigFile("uniqush.conf")
```

Parsing behaviour is identical — every section and option of uniqush's own
config file was compared between the two parsers, 27 pairs across 14 sections,
with no differences. What changed is around the edges:

- **A failed read is reported.** See below.
- `HasOption` and `GetOptions` no longer consult the default section. They were
  the only functions that did, so `HasOption` could answer true for an option
  `GetString` then reported as missing. The accessors were left alone: making
  them inherit the default section would change the meaning of every existing
  config file.
- `GetOptions` no longer lists the default section's options twice when asked
  for the default section.
- The config *writing* API is dropped, having had no users.
- The remains of the `%(name)s` substitution feature are gone —  `varRegExp`,
  `DepthValues`, `MaxDepthReached`. The feature was removed in 2012; the
  documentation promising it stayed another fourteen years.
- There are tests beyond the single one here.

## The bug, if you have vendored this or another goconf

`Read` returned its `nil` named return value instead of the read error:

```go
func (c *ConfigFile) Read(reader io.Reader) (err error) {
	// ...
		l, buferr := buf.ReadString('\n')
		l = strings.TrimSpace(l)

		if buferr != nil {
			if buferr != io.EOF {
				return err        // nil, at this point
			}
```

So a read that failed part way through a file was reported as a **successful
parse of a complete file**. The caller got a `ConfigFile` containing only the
options that arrived before the failure, with no indication that the rest was
missing — for a program that configures itself from that file, silently running
on half a config.

The fix is `return buferr`.

Worth checking your own copy: `git blame` dates this line to **April 2010**, in
Stephen Weinberg's original. It was inherited by this fork rather than introduced
by it, so [ifwe/goconf][ifwe] and the other descendants of goconf are likely to
have it too.

## If you want a config parser in Go

For INI specifically, [`gopkg.in/ini.v1`][ini] is the maintained option. For a
new project, the ecosystem has largely moved to TOML ([`BurntSushi/toml`][toml])
or YAML, both of which have a specification to be correct against — which
INI-style formats, this one included, do not.

Licence: BSD 3-clause, per `conf/COPYRIGHT`. Unchanged by the move; the in-tree
copy carries the same notice.

[ifwe]: https://github.com/ifwe/goconf
[uniqush-push]: https://github.com/uniqush/uniqush-push
[adopted]: https://github.com/uniqush/uniqush-push/tree/master/conf
[ini]: https://github.com/go-ini/ini
[toml]: https://github.com/BurntSushi/toml
