---
URL: https://github.com/tomnomnom/gron
Description: Make JSON greppable!
---
>[!summary]
>gron transforms JSON into discrete assignments to make it easier to `grep` for what you want and see the absolute 'path' to it. It eases the exploration of APIs that return large blobs of JSON but have terrible documentation.

### Usage

```bash
▶ gron "https://api.github.com/repos/tomnomnom/gron/commits?per_page=1" | fgrep "commit.author"
json[0].commit.author = {};
json[0].commit.author.date = "2016-07-02T10:51:21Z";
json[0].commit.author.email = "mail@tomnomnom.com";
json[0].commit.author.name = "Tom Hudson";

▶ gron "https://api.github.com/repos/tomnomnom/gron/commits?per_page=1" | fgrep "commit.author" | gron --ungron
[
  {
    "commit": {
      "author": {
        "date": "2016-07-02T10:51:21Z",
        "email": "mail@tomnomnom.com",
        "name": "Tom Hudson"
      }
    }
  }
]

▶ gron testdata/two.json | grep twitter
json.contact.twitter = "@TomNomNom";
```