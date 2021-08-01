## New Page
```bash
bundle exec rrake new_page "title"
# > source/_includes/custom/navigation.html
```

## New Post
```bash
bundle exec rrake new_post "title"
# > source/_posts_/2021-04-01-title.markdown
```

## deploy
```bash
bundle exec rake preview
bundle exec rake generate
bundle exec rake deploy
git push origin source
```