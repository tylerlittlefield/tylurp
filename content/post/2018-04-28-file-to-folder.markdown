---
title: File to folder
author: ~
date: '2018-04-28'
slug: file-to-folder
categories: ['programming', 'bash']
tags: []
---

I recently needed to create folders for every file in a directory, name these folders after the file, and move the file to the correct folder. The directory I was working with had over 1,000 files so I wasn't going to do this by hand. Instead I discovered a way to automate this process using a shell script.

For example, navigate to a directory and then run the following:


```bash
for i in *; do 
  extension="${i##*.}"
  filename="${i%.*}"
  mkdir -p "$filename/"
  mv "$i" "$filename/"
done
```

And for anyone using Windows, you can create a batch file (`.bat`) like so:


```bash
for %i in (*) do md "%~ni" && move "%~i" "%~ni"
```

