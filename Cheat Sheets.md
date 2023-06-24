# Links

- [Chripy Blog](https://chirpy.cotes.page/)

more functions about post on https://jekyllrb.com/docs/posts/

# Jekyll Commands

```shell
# --drafts : see draft posts.
bundle exec jekyll s (serve)
```

# Jekyll-Compose

# Image

```md
![img-description](/path/to/image){: w="700" h="400" .left}
_Image Caption_
```

define `w/h` to fix layout before post render.

set arrangement by `.left`, `.right`, `.normal`

set visible on light/dark mode by `.light`, `dark`

set shadow by `.shadow`

set `lqip` by `lqip="/path/to/lqip-file"`

# Prompts

```md
> Example line for prompt.
> {: .prompt-info }
```

class : `tip`, `info`, `warning`, `danger`

# File path highlights

```md
`/path/to/a/file.extend`{: .filepath}
```

# Code Block

```
you can add options with {: options} at the end of the code block.
```

{: options}

no line number with `.nolineno`

filename with `file="path/to/file"`

if you want to make dynamic contents with `data` / `collections` / `etc...` see `liquid code` section of `chripy theme`.

- by using `liquid code`, you can add post link list dynamically.

# Videos

```md
{% include embed/{Platform}.html id='{ID}' %}
```

`Platform` is like `youtube`, `twitch`

`ID` is on the URL.
