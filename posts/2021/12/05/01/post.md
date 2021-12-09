# Linux `grep` command tips

[micropost]

Use `man grep` for full details about grep. In this short post i'm trying to
paste very handy one liners for future reference.

```
$ grep -l "PANEL2" ./*.json | grep -cr 'whole foods'
```
`-l` `--files-with-matches` - repost files names

`-c` `--count` - show appearance count

`-r` `--recursive` - recursively search through folders

-more-

```
$ grep --include=\*.json -rnP ./ -e '"track_num": "1."'
```

```
$ grep --include=\*.json -rnP ./ -e "PANEL2"
```

 #linux #grep #helper

This is pretty much it.


