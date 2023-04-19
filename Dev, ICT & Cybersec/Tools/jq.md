---
Description: jq is a lightweight and flexible command-line JSON processor.
URL: https://github.com/stedolan/jq
---

# Documentation

>[!note]
>Documentation: https://stedolan.github.io/jq/

# Basic usage

```bash
# takes the input and produces it unchanged as output
$ cat 20220716095603_users.json | jq '.'

# takes specific part the input and outputs specific blocks
$ cat 20220716095603_users.json | jq '.["meta"]'
$ cat 20220716095603_users.json | jq '.[0]'

# Extract all the keys
$ cat 20220716095603_users.json | jq '. | keys'

# extract only specific field 
$ cat 20220716095603_users.json | jq '.data[]'
$ cat 20220716095603_users.json | jq '.data[].Properties'
$ cat 20220716095603_users.json | jq '.data[].Properties | .name'
$ cat 20220716095603_users.json | jq '.data[].Properties | {name: .name, domain: .domain}'

# Filter specific data from the input
$ cat 20220716095603_users.json | jq '.data[].Properties | select(.enabled==true) | .name'

# Convert integers to string
$ cat 20220716095603_users.json | jq '.data[].Properties | .name + " " + (.lastlogontimestamp|tostring) + " " + (.pwdlastest|tostring)'

# Find account which password has been reset but which have not yet logged in
$ cat 20220716095603_users.json | jq '.data[].Properties | select(.enabled==true) | select(.pwdlastset > .lastlogontimestamp) | .name + " " + (.lastlogontimestamp|tostring) + " " + (.pwdlastest|tostring)'
```