# zhtmldb

**The storage format is also the UI.**

zhtmldb is a key-value datastore whose records are XHTML pages. A single-file Java 25 CLI writes them; a browser, an XML parser, `git diff`, `grep` or a coding agent reads them. No server, no libraries, no build: one executable script on PATH.

- **Human-readable.** Every record is a valid HTML5 page with its fields in a `<dl>`. Open the folder in a browser and navigate the data through generated index pages.
- **Git-friendly.** One record, one file. Writes are atomic (temp file plus rename), so history and diffs stay per record and a reader never sees a half-written page.
- **Shell- and agent-native.** Data on stdout, diagnostics on stderr, tab-separated output for `keys`, `get` and `find`, exit code 1 on a miss. No client library: pipe it, or parse the pages as XML.
- **Dedicated CLIs.** A symlink named after a table is a preconfigured command for that table, with its own configuration.

## Layout

Each table is a folder, each record its own page, every level linked by an `index.html`:

```
.
├── index.html            # links all tables
├── talks/
│   ├── index.html        # column order + links to all records
│   └── 2026-08-04-142122.html
└── users/
    ├── index.html
    └── duke.html
```

A record page:

```xml
<main>
  <h1>duke</h1>
  <dl>
    <dt>email</dt>
    <dd>duke@airhacks.com</dd>
    <dt>blog</dt>
    <dd>duke.blog</dd>
  </dl>
  <footer>
    <p>updated <time datetime="2026-08-03T15:31:07Z">2026-08-03T15:31:07Z</time></p>
  </footer>
</main>
```

Fields follow the table's declared column order in the page, in `get` and in `list`; undeclared fields sort after them, and without columns a record stays alphabetical. A table index links each record by its first column's value, falling back to the key. Table, key and field names are filename-safe slugs (letters, digits, `_`, `-`; `index` is reserved), values are arbitrary text.

## Dedicated CLIs

The script derives its identity from the invoked file name (busybox-style). A symlink named after a table prepends that table to every command:

```bash
ln -s /usr/local/bin/zhtmldb /usr/local/bin/talks

talks columns title,description                   # once: column order for positional entry
talks add "Java 25" "What's new in the source launcher"   # → talks/2026-08-04-142122.html
talks list
talks find java | cut -f1
```

`add` stores values positionally under a generated timestamp key; `columns` defines the order once. Each persona reads its own global configuration (`~/.talks/app.properties`), so every dedicated CLI can point at its own database.

## Usage

```bash
zhtmldb users set duke email=duke@airhacks.com blog=duke.blog   # creates users/duke.html
zhtmldb users set duke twitter=@duke               # merges into the record
zhtmldb users set duke bio+="Java Champion"        # += appends, newline separated
echo "Java Champion" | zhtmldb users set duke bio  # value from stdin (bio+ appends)
zhtmldb users columns email,blog,twitter
zhtmldb users set jane "jane@airhacks.com,janes.blog,@jane"   # positional by columns

zhtmldb users get duke                             # all fields, field<TAB>value
zhtmldb users get duke email                       # → duke@airhacks.com
zhtmldb users get duk                              # abbreviated key, must be unique
zhtmldb users keys                                 # sorted keys
zhtmldb users list                                 # aligned table, long values cut
zhtmldb users rm duke twitter                      # remove a field
zhtmldb users rm duke                              # remove the record
zhtmldb tables
```

### Filtering

`find` and `list` take the same terms and differ only in output: `find` prints `key<TAB>label` to pipe, `list` renders a table to read.

```bash
zhtmldb users find airhacks                  # any field value, or the key
zhtmldb users find blog=duke                 # one field only
zhtmldb users find blog=                     # records having the field at all
zhtmldb notes list category=todo priority=high   # all terms have to match
```

Matching is a case-insensitive substring. A filter that matches nothing exits 1; an unfiltered `list` of an empty table does not.

### Keys

`set`, `get` and `rm` resolve their key by exact match first, then as a case-insensitive substring of the stored keys, so `get 142122` reaches a timestamp record without the date. Several hits exit 1 and list them. For `get` and `rm` no hit exits 1; for `set` it is the key of a new record, so a new key that is a substring of an existing one updates that record instead.

Exit code 0 on success, 1 on missing keys or tables, no match, or usage errors.

## Configuration

The database root is the current directory. Override it with `db.dir` ([zcfg](https://github.com/AdamBien/zcfg) precedence, later wins):

1. `~/.zhtmldb/app.properties` (or `~/.<persona>/app.properties`)
2. `./app.properties`
3. `-Ddb.dir=<path>`

```properties
db.dir=/path/to/database
```

## Installation

Requires Java 25 or later.

```bash
curl -O https://raw.githubusercontent.com/AdamBien/zhtmldb/main/zhtmldb
chmod +x zhtmldb
./zhtmldb -help
sudo cp zhtmldb /usr/local/bin/            # or: sudo ln -s "$(pwd)/zhtmldb" /usr/local/bin/zhtmldb
```
