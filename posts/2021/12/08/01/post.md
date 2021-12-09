# Blog shortcuts

### Preview
Make a preview

```
$ pandoc post.md > preview.html
```

### Publishing
Then make a post
```
$ pandoc post.md > post.html; \
  cat post.md | grep -oP ' #[[:alnum:]]+' | grep -oP '#[[:alnum:]]+' > hashtags.txt
```

### Remove publishing
Then make a post
```
$ rm post.html
```

-more-

### Categories

Categories are - one category per row in the file `categories.txt`

### Hashtags
Hashtags must start from space and follow with `#` sign

> article #thisishashtag1 and #thisishashtag2 text

#### Testing hashtags
```
$ cat post.md | grep -oP ' #[[:alnum:]]+' | grep -oP '#[[:alnum:]]+'
```

#### Removing hashtags
```
$ rm hashtags.txt
```
