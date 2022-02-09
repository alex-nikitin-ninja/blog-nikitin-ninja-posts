# Linux `tar` command tips

## Create archive
```
tar -czvf <dst-name>.tar.gz <src-folder-or-file>
```

Example:
```
tar -czvf armbian-configured.img.tar.gz armbian-configured.img
```

-more-

## With custom compression level
```
GZIP=-9 tar cvzf file.tar.gz <dst-name>.tar.gz <src-folder-or-file>
```

## Extract archive

### In the same folder
```
tar -xzvf armbian-configured.img.tar.gz
```

### Different folder
```
tar -xzvf <name>.tar.gz -C <destination path>
```

## Parameters description

`-c` - create archive

`-x` - extract archive

`-z` - compress with gzip

`-v` - verbose or show progress

`-f` - spcify destination archive file name

