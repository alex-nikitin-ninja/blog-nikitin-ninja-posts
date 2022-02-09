# Linux `tar` command tips

## Create archive
```
tar -czvf <dst-name>.tar.gz <folder_or_file>
tar -czvf armbian-configured.img.tar.gz armbian-configured.img
```

-more-

## With custom compression level
```
GZIP=-9 tar cvzf file.tar.gz /path/to/directory
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