# Linux `grep` command tips

Use `man grep` for full details about grep. In this short post i'm trying to
paste very handy one liners for future reference.

```
$ grep --include=\*.json -rnP ./ -e "PANEL2"
```

`-r` `--recursive` - recursively search through folders

`-n` `--line-number` - show lines numbers

`-P` `--perl-regexp` - perl-compatible  regular  expressions

`./` - current directory for search

`-e` `--regexp` - search expression

`--include` - only filter the files according to template

-more-

```
$ grep --include=\*.* --exclude=\*.json -rnP ./ -e "PANEL2"
```

`--exclude` - remove files indicated here from search (syntax similar to above)

### grep vs. egrep

There's another command called `egrep`, which has similar syntax, but uses
extended regular expressions. `egrep` is an equivalent of `grep -E` , sometimes
just use `egrep` as a shortcut.

The difference is extended vs "basic" regexp is interpreting characters
`+`, `?`, `|`, `(`, `)` as meta characters in `egrep` and as pattern in `grep`

### Common syntax and handy flags

```
$ grep [OPTION...] PATTERNS [FILE...]
```
some other helpful flags to use:

`-i` `--ignore-case` - case insensitive search

`-o` `--only-matching` - print only what matches search criteria

`-l` `--files-with-matches` - show files names for later usage

`-L` `--files-without-match` - helpful to show files which don't met search
     expression criteria


 #linux #grep #helper

### Conclusion

This is pretty much it.


