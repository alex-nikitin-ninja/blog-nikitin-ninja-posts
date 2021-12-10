# Blog shortcuts

### Structure
```
./blog-nikitin-ninja-posts/posts/YYYY/MM/DD/NN/post.md
./blog-nikitin-ninja-posts/posts/2021/12/04/02/post.md
```
`YYYY` - year  
`MM` - month  
`DD` - day  
`NN` - number of the post for the day  


### Preview
```
$ pandoc post.md > preview.html
```

### Publishing
```
$ pandoc post.md > post.html
```

-more-

### Remove publishing
```
$ rm post.html
$ mv post.html preview.html
```

### Authors list

`authors.txt` - authors - one author per row in the file 

### Categories

`categories.txt` - categories - one category per row in the file 

### Hashtags
`hashtags.txt` - hashtags must start from space ` ` and follow with `#` sign: ` #yourhashtag`

#### Testing/preview hashtags
```
$ cat post.md | grep -oP ' #[[:alnum:]]+' | grep -oP '#[[:alnum:]]+'
```

#### Publishing hashtags
```
$ cat post.md | grep -oP ' #[[:alnum:]]+' | grep -oP '#[[:alnum:]]+' > hashtags.txt
```

#### Removing hashtags from post
```
$ rm hashtags.txt
```

### Blockquotes

> **Example:** article #thisishashtag1 and #thisishashtag2 text


### Table
|Syntax   |Description|
|-|-|
|Header   |Title|
|Paragraph|Text| 