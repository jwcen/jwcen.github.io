## RUN
local
```shell
bundle exec jekyll s
```

## Creating Posts
add a file to your `_posts `directory with the following format:

```bash
YEAR-MONTH-DAY-title.MARKUP
# such as
2011-12-31-new-years-eve-is-awesome.md
```

All blog post files must begin with front matter which is typically used to set a layout or other meta data. For a simple example this can just be empty:

```bash
---
layout: post
title:  "Welcome to Jekyll!"
---

# Welcome

**Hello world**, this is my first Jekyll blog post.

I hope you like it!
```
## Categories and Tags 
To use these, first set your categories and tags in front matter:
```md
---
layout: post
title: A Trip
categories: [blog, travel]
tags: [hot, summer]
permalink: /:categories/:year/:month/:day/:title:output_ext
---
```

[docs](https://chirpy.cotes.page/posts/write-a-new-post/)

