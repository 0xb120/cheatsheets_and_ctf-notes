---
raindrop_id: 1557583442
raindrop_highlights:
  6973368fb879df73c14b6c20: 55afc086b5b4462e23ab43917934a4be
  697336ffc106a2635b0dd288: 1ec52a541567c1dcfcf56e36336eca43
  69733717f019b5f3ae85a978: c7376ae53cb4edb1ca5db467a0f31355
  6973371dc106a2635b0ddac0: 803c0dfcd2e568c39d94f8d723252d56
  697337322e688b1ea4428f9b: d160df294a3cf9d0ef40786f38b35dfe
  6973374743eb74a962209cb9: c15da2068e6a8934da20ebb38c3955f5
  69733786f904453c0cc6b23a: 99234bb9e561e2bfde1c4f8ff6e8a6ba
  6973378b52a3c536e4b1dff7: 23f1c6e06317e081d8250b6382da1e40
  697337df96812e6d95c13952: cb16b2328fce2e42f82e75816183d01e
  697337f1ec221223669fc427: f83b24f04150a77c7cb63047e3399b58
  69733821e4bf191f03ae4b70: 2a46f381486624fd2bf0d2bdfcb1c86b
  69733827bd08d3c14c80b867: c88eceaf30eeac816f481797a10d8b4d
  6973385be4bf191f03ae5cfe: 5d8019d7c0d08b87f8e3e6011320cc23
  697338673afa22f1b182e0fb: 84a49234aabba2ff3c1c56965aaa8689
  69733876e4bf191f03ae6444: 13c44b86c785c5ae1251bf07430b2d2d
  69733885bef92d1f009f7a1f: 4eb902bfdbb1104d91c7709680a6e5ce
title: "Yet another ZIP trick"

source: https://hackarcana.com/article/yet-another-zip-trick

created: 2026-01-23
sync-date: 1769635439374
tags:
  - "_index"

 
  - "tech-blog"

---
# Yet another ZIP trick

![](https://hackarcana.com/assets/img/favicon.png)





Yet another ZIP trick
Multiple Personality ZIPs

A schizophrenic file is a file that is interpreted in at least two different ways by two different parsers.
personally I prefer to refer to it as a multiple personality files or multiple personality disorder file, but I think the original name stuck.
See also: Ange Albertini & Gynvael Coldwind: Schizophrenic Files – A file that thinks it's many (39m42s, Area41 2014)
So, a schizophrenic ZIP is an archive file that can be seen—when using two different programs—as having two different sets of files within. The actual behaviour can range from having totally different sets of files, to having extra or missing some files. One way or another, the content would be different.
These discrepancies in ways a file can be interpreted have three main causes:

Redundant information inside a file, where parser developers basically have a choice which set of information to use.
Ambiguity in the specification.
Ignoring part of the specification, be it by mistake, accident, or deliberate omission.
there are multiple ways to craft schizophrenic ZIPs. I've catalogued a lot of these ways and presented them a few years back during my "Ten thousand security pitfalls: The ZIP file format" talk at the Technische Hochschule Ingolstadt, in Germany.

But it seems I've missed at least one.
EoCDR: Size vs Offset
Even though "End of Central Directory Record" has "end" in its name, and it's located at the end of the file, this is actually the first structure one has to parse to interpret a ZIP file.
EoCDR contains very basic information about the content of the file, including the offset of the start of the Central Directory—the main index of files within the archive.
I pointed to this offset field
and said that the way to fix the ZIP file to show all files is to correct the offset to point to the actual first "hidden" Central Directory entry, as currently it's skipped.
As pointed out earlier, redundant information is a primary reason for schizophrenic file existence. And this specific case is a great example of that.

As such, I proceeded to create, on purpose this time, a schizophrenic file with 2 files inside:

offset_of_start.txt – pointed to by the offset field
size_of_central_directory.txt – relatively pointed to by the size of CD field
At the same time "total number of entries" fields claim that there is only a single file within the archive. As such parsers using the offset, will see only the offset_or_size.txt file, and parsers using the size will see only the size_of_central_directory.txt file.
At this point you're probably wondering how would the software you are or your company is using interpret such a file. Will different systems see a ZIP file differently?
If you like dabbling in hexeditors, you can create a file like that yourself. Here's an CTF-like exercise for additional motivation: Yet Another ZIP File (75 pts)

If you just want to test it, you can download the test file here: offset_or_size.zip (651 bytes)