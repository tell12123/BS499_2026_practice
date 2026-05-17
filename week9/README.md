# BS499_2026_practice

Practice codes for BS499.

This repository contains the codes used in class so that students can review and practice them more easily.

---

# Week7

## Login to server

```bash
ssh workshop@ (the server #, need to ask TA)
```

---

## Conda environment

Conda is a tool that manages programs and packages in separate environments.

Activate BS483 environment:

```bash
conda activate BS483
```

---

## Important rules

1. Do not run any heavy job or program directly on `junglab.kaist.ac.kr`
   except for ssh login.

2. Storage space is limited.
   Please compress large files when possible (example: `.gz` format).

3. Use `qsub/qstat` for heavy jobs on `alphacom.kaist.ac.kr`.

4. If possible, use other clusters you have access to.

5. Use Conda/Anaconda for installing programs.

---

# Useful Linux commands

## File and directory commands

```bash
pwd
```

Show current directory.

```bash
ls
```

List files and directories.

```bash
mkdir myfolder
```

Make new directory.

```bash
cd myfolder
```

Move directory.

```bash
cp file1.txt backup.txt
```

Copy file.

```bash
mv old.txt new.txt
```

Move or rename file.

```bash
rm file.txt
```

Remove file.

```bash
rm -r myfolder
```

Remove directory.

Special directory symbols:

```bash
~
```

Home directory.

```bash
.
```

Current directory.

```bash
..
```

Previous directory.

---

# Text editor (vim)

Open file:

```bash
vim filename.txt
```

Basic usage:

- `i` → insert/edit mode
- `ESC` → leave insert mode
- `:wq` → save and quit
- `:q` → quit
- `:q!` → quit without saving

Useful vim commands:

- `gg` → first line
- `Shift+g` → last line
- `/word` → search word
- `p` → paste copied text

---

# Standard input/output

```bash
cat file.txt
```

Print file content.

```bash
zcat file.gz
```

Print compressed file content.

```bash
grep word file.txt
```

Search word in file.

```bash
less file.txt
```

Open large text file.

```bash
echo "hello"
```

Print text.

---

# Download and transfer data

```bash
wget URL
```

Download file from internet.

```bash
scp file user@server:path
```

Transfer file between servers.

---

# Other useful commands

```bash
which fastqc
```

Show program location.

```bash
find . -name "*.txt"
```

Find files.

```bash
gzip file.txt
```

Compress file.

```bash
gunzip file.txt.gz
```

Uncompress file.

---

# PATH setting

Open bash profile:

```bash
vim ~/.bash_profile
```

Add path:

```bash
PATH=$PATH:$HOME:/home/users/workshop/miniconda3/bin/
```

Apply changes:

```bash
source ~/.bash_profile
```

Check program:

```bash
which fastqc
which fasterq-dump
```

---

# PBS (Portable Batch System)

PBS is used to submit jobs to compute nodes.

---

## Example PBS script

```bash
########################################################
### Example PBS script for qsub
### Lines starting with "#PBS" are settings for PBS
###
### To submit:
### qsub PBS_template_BS499.pbs
###
### To pass a variable into the PBS script:
### qsub -v VariableName=i PBS_template_BS499.pbs
########################################################

#!/bin/bash

#PBS -N myjobname
#PBS -q workq
#PBS -l nodes=1:ppn=1
#PBS -j oe

## Move to the folder where qsub was submitted
## $PBS_O_WORKDIR = the pwd where qsub was run

cd $PBS_O_WORKDIR

sleep 5

echo "I am on this computer:"
hostname

## Put your code below
sleep 30

## End the script
exit 0
```

---

## PBS option explanation

```bash
#PBS -N myjobname
```

Set job name.

```bash
#PBS -q workq
```

Set queue name.

```bash
#PBS -l nodes=1:ppn=1
```

- `nodes=1` → use one available node
- `ppn=1` → use one CPU core

```bash
#PBS -j oe
```

Save output and error messages together.

```bash
cd $PBS_O_WORKDIR
```

Move to the directory where `qsub` was run.

```bash
hostname
```

Show which node is running the job.

```bash
sleep 30
```

Wait 30 seconds (example test job).

---

## Submit PBS job

```bash
qsub PBS_template_BS499.pbs
```

---

## Check running jobs

```bash
qstat
```

---

## PBS output file

After the job finishes:

```bash
myjob.o123456
```

will be created.

This file contains normal output and error messages.
