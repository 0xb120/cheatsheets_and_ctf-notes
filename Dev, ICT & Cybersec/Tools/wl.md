---
URL: https://github.com/s0md3v/wl
Description: convert case style of words
---
>[!summary]
>convert case style of words

### Usage

>[!note]
>Note: You don't need to use the words `foo` and `bar`, any word e.g. `prod_api_id` will work. `wl` will detect the casing style of your word and apply it to your wordlist. This can be handy in cases like changing your wordlist according to the input recieved from another tool.

```bash
$ wl -h
Usage of wl:
  -c string
        casing style (required)
  -i string
        input file (default stdin)
  -o string
        output file (default stdout)

$ wl -c fooBar -i input.txt -o output.txt
$ cat input.txt | wl -c fooBar
```

#### Common

`foobar, foo_bar, fooBar, FooBar, FOOBAR, F<OO_BAR`

#### More

`foo-bar, FOO-BAR, foo.bar, FOO.BAR, Foo.Bar, Foo_Bar, Foo-Bar, foo.Bar, foo.Bar, foo_Bar, foo-Bar`