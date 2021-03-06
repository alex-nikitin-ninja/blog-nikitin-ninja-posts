# Linux `tar` command  quick tips

Command `tar` stands for tape archive, is used to create archive and extract the archive files.

## Create archive

Common syntax:
```
tar -czvf <dst-name>.tar.gz <src-folder-or-file>
```

Example:
```
tar -czvf armbian-configured.img.tar.gz armbian-configured.img
```

-more-

## With custom `gzip` compression level
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
tar -xzvf <name>.tar.gz -C <destination-path>
```

## Parameters description

`-c` - create archive

`-x` - extract archive

`-z` - compress with gzip (make sure to add `.gz` extension)

`-v` - verbose or show progress

`-f` - spcify destination archive file name


 #linux #tar #helper 
