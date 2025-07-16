---
URL: https://github.com/projectdiscovery/katana
Description: A next-generation crawling and spidering framework.
---
>[!summary]
>Katana is a fast web crawler made by Project Discovery. The tool is both headless and non-headless with a focus on being used in automation workflows. For example Katana could be used to crawl a target and stored all crawled data, or Katana could be used to crawl a site and store all urls with inputs. The following Katana cheat sheet aims to provide an overview of the tools functionality and provide real world examples from existing workflows.

### Usage

Cheatsheet: https://highon.coffee/blog/katana-cheat-sheet/

```bash
katana -silent -list scope.txt -d 3 -jc -kf all -o katana-output.txt -ncb


katana -f qurl -o qurl-output.txt
```