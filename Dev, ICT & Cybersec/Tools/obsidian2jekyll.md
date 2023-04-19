Usage:
```bash
$ ./obdisian2jekyll.py -n valentine -s SecondBrain/Play\ ground/CTFs/valentine.md -d /home/maoutis/0xbro.github.io/dev/_writeups/Web\ Hacking/ -a /home/maoutis/0xbro.github.io/assets/images/Screenshots/
```

>[!warning]
>Run the script from the `/dev` folder

Code:
```python
#!/usr/bin/python3

import urllib.parse, argparse, os, tempfile, re, shutil, string

callouts = [ "bug","danger","error","warning","attention","note","info","tip","hint","question","important","new","done","success","tldr","abstract","summary"]

parser = argparse.ArgumentParser(description="Obsidian to Jekyll markdown converter")
parser.add_argument("-n","--name", help="name for the new file and attachment's folder")
parser.add_argument("-s","--source", help="source markdown file")
parser.add_argument("-d","--destination", help="destination folder (default /tmp)", default="/tmp")
parser.add_argument("-t","--template", help="template file (default /home/maoutis/0xbro.github.io/_drafts/template-chall.md)", default="/home/maoutis/0xbro.github.io/_drafts/template-chall.md")
parser.add_argument("-a","--attachments", help="attachments folder (default /home/maoutis/0xbro.github.io/assets/images/Screenshots/)", default="/home/maoutis/0xbro.github.io/assets/images/Screenshots/")

args = parser.parse_args()

if not(args.source or args.name):
    print("Usage: obsidian2jekyll.py -n <name> -s <source_file> -d <destination_folder> -t <template_file> -a <attachments_folder>")
    exit(0)

# Open template file
with open(args.template, 'r') as template :
    template_data = template.read()
print("{0} load in memory".format(args.template))

# Open obsidian writeup
with open(args.source, 'r') as chall :
    chall_data = chall.read()
print("{0} load in memory".format(args.source))

images_list = re.findall(r'^[!]\[\][(](.*)[)]', chall_data, re.MULTILINE)

# Replacing placeholders with real data
print("Replacing placeholders with real data...")
template_data = template_data.replace('???-name-???',args.name)
template_data = template_data.replace('???-difficulty-???',re.search(r"^Difficulty:.*$", chall_data, re.MULTILINE).group(0).lower())
template_data = template_data.replace('???-platform-???',re.search(r"^Platform:.*(\n[ ]*-[ ]*[a-zA-Z]*)+", chall_data, re.MULTILINE).group(0).lower())
template_data = template_data.replace('???-tags-???',re.search(r"^Tags:.*(\n[ ].-[ ]*[a-zA-Z0-9.-]*)+", chall_data, re.MULTILINE).group(0).lower())
template_data = template_data.replace('???-challenge-???',chall_data[chall_data.find("#"):-1])
template_data = template_data.replace('../../zzz_res/attachments/',"{{page.screenshots}}")
template_data = template_data.replace('```http',"```")

t = string.Template('{: .$w }')
quotes = re.findall(r"^>\[![a-zA-Z]+\][+\- a-zA-Z]+", template_data, re.MULTILINE)

print("Found the following quotes {0}".format(quotes))

for q in quotes:
    w = re.search("!([a-zA-Z]*)", q, re.MULTILINE).group(1)
    if w in callouts:
        foo = t.substitute(w=w)
        print("Replacing {0} with {1}".format(q, foo))
        template_data = template_data.replace(q, foo)

print("Finished")

# Create temp file
tmp_dir = tempfile.TemporaryDirectory()
with open(os.path.join(tmp_dir.name, args.name+".md"), 'w') as tmp_file:
    tmp_file.write(template_data)

print("Created temporary file '{0}'".format(tmp_file.name))

# Create attachments folder
atch_folder = os.path.join(tmp_dir.name, args.name)
try:
    os.mkdir(atch_folder)
    print("Creating attachments folder...")
except OSError as error:
    print(error)    

print("Attachments folder '{0}' created".format(atch_folder))

# Copy attachments inside the temp folder
for img in images_list:
    img_full_path = os.path.abspath(os.path.join(os.path.dirname(os.path.abspath(args.source)), img))
    img_path, img_name = os.path.split(img_full_path)
    img_new_path = os.path.join(atch_folder, img_name)
    print("Copying {0} in {1}".format(img_full_path, img_new_path))
    shutil.copyfile(img_full_path, img_new_path)

# Moving files to destination if it does not exists
final_file_name = os.path.join(args.destination,args.name+".md")
if not os.path.exists(final_file_name):
    print("Moving {0} to {1}".format(tmp_file.name, final_file_name))
    shutil.move(tmp_file.name, final_file_name)
else:
    print("Error: file '{0}' already exists".format(final_file_name))
    exit(1)

final_attachment_folder = os.path.join(args.attachments, args.name)
if not os.path.exists(final_attachment_folder):
    print("Moving {0} to {1}".format(atch_folder, final_attachment_folder))
    shutil.move(atch_folder, final_attachment_folder)
else:
    print("Error: folder '{0}' already exists".format(final_attachment_folder))
    exit(1)

tmp_dir.cleanup()
exit(0)
```