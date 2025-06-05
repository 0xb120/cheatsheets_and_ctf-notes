## Tomnomnom

OSINT crawler
`assetfinder > domains.txt`

search http web servers from domain a list. Maybe can do the same with whatweb or httpx
`cat domains.txt | httprobe | tee hosts.txt`

request a specific path on multiple hosts, and store http and https results for every host in separate directories
`meg -d 1000 -v /`
`comb + fff`

Take a list of files in input and extract html attrbutes
`find . -type f | html-tool` 

Create a list of unique path from a list of urls 
`unfurl -u path`
Create a list of unique parameters from a list of urls 
`unfurl -u keys`
Maybe also `htmlq`

Make record unique
`cmd | anew`
`cmd | sort | uniq -c | sort -nr`

Run commands for every line in input (n = number of line; P = number of commands in parallel; {} placeholder to decide where to place arguments)
`cat domains.txt | xargs -n1 -P5 host`
`find . -type f -name '*.txt' | xargs -n1 -l{} cp {} ../backup/`


`gron`

## Piper for burp
Use tools that read from stdin and print to stdout, directly in burpsuite