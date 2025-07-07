
# Reading

```dataview
table WITHOUT ID ("[["+file.path+"|"+file.frontmatter.title+"]]") as Title, author as Author, ("![|100](" + cover + ")") as Cover, pages, category
from "Personal/Book list"
where contains(reading-status,"Reading")
sort title
```

# To-Read

```dataview
table WITHOUT ID ("[["+file.path+"|"+file.frontmatter.title+"]]") as Title, author as Author, ("![|100](" + cover + ")") as Cover, pages, category
from "Personal/Book list"
where contains(reading-status,"Waiting")
sort title
```

# Finished

```dataview
table WITHOUT ID ("[["+file.path+"|"+file.frontmatter.title+"]]") as Title, author as Author, ("![|100](" + cover + ")") as Cover, pages, category, rating
from "Personal/Book list"
where contains(reading-status,"Finished")
sort rating DESC, title
```

