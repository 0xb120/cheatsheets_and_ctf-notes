# Vulnerability research and training

## TODO/WIP

```dataview
Table category, product, last-time, file.tags as tags
From #training or #vulnerability-research 
sort last-time DESC
Where contains(file.tags, "training/todo") or contains(file.tags, "vulnerability-research/todo")  or contains(file.tags, "vulnerability-research/wip")
```



## Finished/completed

```dataview
Table category, product, last-time, file.tags as tags
From #training/finished or #vulnerability-research/finished
sort last-time DESC
```


---

# Projects
## All

```dataview
Table project-type as "project type", category, product, last-time
From "Play ground/Projects"
where contains(project-type, "") 
sort last-time DESC
```


## Mobile

```dataview
Table project-type as "project type", product, last-time
From "Play ground/Projects"
where contains(category, "Mobile") 
sort last-time DESC
```


## Black-box

```dataview
Table last-time
From "Play ground/Projects"
where contains(project-type, "black-box pentest") 
sort last-time DESC
```

---
