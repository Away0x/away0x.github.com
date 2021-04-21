## New Page
```bash
rake new_page["title"]
# > source/_includes/custom/navigation.html
```

## New Post
```bash
rake new_post["title"]
# > source/_posts_/2021-04-01-title.markdown
```

## deploy
```bash
rake preview
rake generate
rake deploy
git push origin source
```