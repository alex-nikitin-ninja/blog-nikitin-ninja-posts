# Linux `grep` command tips

Use `man grep` for full details about grep. In this short post i'm trying to
paste very handy one liners for future reference.

```
$ grep -l "PANEL2" *json | grep -cr 'whole foods'
```
`-l`
 `--files-with-matches`

`-c`
 `--count`

`-r`
 `--recursive`

-more-

```
$ grep --include=\*.json -rnP ./ -e '"track_num": "1."'
```

```
$ grep --include=\*.json -rnP ./ -e "PANEL2"
```




this is pretty much it.

