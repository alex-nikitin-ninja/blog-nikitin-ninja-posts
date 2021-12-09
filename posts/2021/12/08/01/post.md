# Blog shortcuts

### Preview
Make a preview

```
$ pandoc post.md > preview.html
```

### Structure
```
./blog-nikitin-ninja-posts/posts/YYYY/MM/DD/NN/post.md
./blog-nikitin-ninja-posts/posts/2021/12/04/02/post.md
```
`YYYY` - year  
`MM` - month  
`DD` - day  
`NN` - number of the post for the day  

### Publishing
Then make a post
```
$ pandoc post.md > post.html; \
  cat post.md | \
  grep -oP ' #[[:alnum:]]+' | \
  grep -oP '#[[:alnum:]]+' > hashtags.txt
```

### Remove publishing
Then make a post
```
$ rm post.html
```

-more-

### Authors list

Authors are - one author per row in the file `authors.txt`

### Categories

Categories are - one category per row in the file `categories.txt`

### Hashtags
Hashtags must start from space ` ` and follow with `#` sign: ` #yourhashtag`

> article #thisishashtag1 and #thisishashtag2 text

#### Testing/preview hashtags
```
$ cat post.md | grep -oP ' #[[:alnum:]]+' | grep -oP '#[[:alnum:]]+'
```

#### Removing hashtags from post
```
$ rm hashtags.txt
```
