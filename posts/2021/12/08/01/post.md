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
