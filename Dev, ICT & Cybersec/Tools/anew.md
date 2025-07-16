---
URL: https://github.com/tomnomnom/anew
Description: A tool for adding new lines to files, skipping duplicates
---
>[!summary]
>Append lines from stdin to a file, but only if they don't already appear in the file. Outputs new lines to `stdout` too, making it a bit like a `tee -a` that removes duplicates.

### Usage

```bash
▶ cat things.txt
Zero
One
Two

▶ cat newthings.txt
One
Two
Three
Four

▶ cat newthings.txt | anew things.txt
Three
Four

▶ cat things.txt
Zero
One
Two
Three
Four
```