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
bundle exce rake preview
bundle exce rake generate
bundle exce rake deploy
git push origin source
```