---
author: "Kalyan"
title: "Adding images in Hugo"
date: "2021-03-05"
description: "Adding images properly in Hugo"
tags: ["webpage", "static"]
ShowToc: true
---

## Adding images in Hugo

I had a lot of trouble adding images in Hugo and searched a lot on the internet and finally solved the problem.
The thanks go to the [comment](https://github.com/gohugoio/hugo/issues/1240#issuecomment-171722560) under an issue.

The secret is to store the images in the `static` folder and also follow the correct directory structure.
If we take this [page](https://parzival2.github.io/manabu.github.io/posts/qtcreator-ros/) as an example, the images are added into `static\posts\qtcreator-ros\` folder to mimic the search path.

So the path should be `static\posts\<Your Post name>\image.png` and you can link it in your page as follows 

```
![Title of image](image.png)
```
Notice there is no path before the image name.

The folders on my computer looks as follows for the above mentioned example

#### Static

```bash
└── posts
    └── qtcreator-ros   <- Notice the name of the post
        └── qtcreator-buildfolder.png
```

#### Content

```bash
├── archives.md
├── posts
│   ├── add-images-hugo.md
│   ├── mathtypesetting.md
│   ├── qtcreator-ros.md    <- The image will be linked in this post.
│   └── rich-content.md
└── search.md
```

