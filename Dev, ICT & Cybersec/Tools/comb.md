---
URL: https://github.com/tomnomnom/comb
Description: Combine the lines from two files in every combination
---
>[!summary]
>Combine the lines from two files in every combination

### Usage

```bash
▶ cat prefixes
1
2

▶ cat suffixes
A
B
C

▶ comb prefixes suffixes
1A
1B
1C
2A
2B
2C

▶ comb --flip prefixes suffixes
1A
2A
1B
2B
1C
2C

▶ comb --separator="-" prefixes suffixes
1-A
1-B
1-C
2-A
2-B
2-C
```