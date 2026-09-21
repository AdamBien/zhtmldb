# AGENTS.md

## Installation

`zhtmldb` is a single executable Java 25 source-file script. No build tool, no dependencies, nothing to compile.

```bash
java -version                  # 25 or later required
chmod +x zhtmldb
./zhtmldb -help
```

System-wide, on PATH:

```bash
sudo cp zhtmldb /usr/local/bin/
# or, for development, symlink the checkout
sudo ln -s "$(pwd)/zhtmldb" /usr/local/bin/zhtmldb
```

A symlink named after a table becomes a dedicated CLI for that table:

```bash
sudo ln -s /usr/local/bin/zhtmldb /usr/local/bin/talks
```
