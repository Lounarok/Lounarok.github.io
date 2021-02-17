---
layout: post
title: Linux cheat sheet for myself.
tag: linux grep scp tar man find keywords copy 
---

# How to compress / uncompress (tar/unar) a folder

zip https://www.cyberciti.biz/faq/how-to-zip-a-folder-in-ubuntu-linux/
tar.gz https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/
https://www.hostdime.com/kb/hd/command-line/how-to-tar-untar-and-zip-files
## Tar + Zip:

`tar cvfz FILENAME.tar.gz DIRECTORY/ `

To remove symbolic link when compressing: 

`tar cvfz FILENAME.tar.gz --dereference DIRECTORY/ `

## Unzip to current dir

`tar xvfz FILE.tar.gz`


# How to install mandocs for kernel api

- [Compile Under kernel source]( https://unix.stackexchange.com/questions/148426/how-to-make-the-kernel-section-9-manpages-which-document-functions-data-structu)

    `make mandocs`

- Install: `install sphinx-build`

    https://www.sphinx-doc.org/en/master/usage/installation.html
    `sudo apt-get install python3-sphinx `
    `make installmandocs`

# How to find keywords in a directory

`grep -rnw '/path/to/somewhere/' -e 'pattern'`

Ref: https://stackoverflow.com/a/16957078/4123703




# Copy files while excluding certain directories

Check resync https://stackoverflow.com/questions/4585929/how-to-use-cp-command-to-exclude-a-specific-directory

rsync -av --progress sourcefolder /destinationfolder --exclude thefoldertoexclud

# Show directory size
du -sh (or sudo du -sh )

#Kernel Module do not allow printk with float point

https://stackoverflow.com/a/13886805/4123703
https://stackoverflow.com/questions/31131012/why-printk-in-linux-kernel-modules-lacks-floating-point-support-unlike-printf