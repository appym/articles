I have been meaning to note down my *nix checklist of commands (For MacOS) which are very handy for basic operations on data. I will modify this post as and when I remember or come across something that fits here. These *nix commands are specifically tested for Mac OS.
**Uniques**  
uniq - This is the unix unique function which can be primarily used to remove duplicates from a file amongst other things. The file has to be pre sorted for uniq to work
Consider file test which contains the following
```bash
$ cat test
aa
bb
bb
cc
cc
cc
```
**Remove duplicates**   
```shell
$uniq test
aa
bb
cc
```
Count occurences of each item
```bash
$ uniq -c test
1 aa
2 bb
3 cc
```
Print only duplicate items in file
```bash
$ uniq -d test
bb
cc
```
Print only unique lines
```bash
$ uniq -u test
aa
```
Consider test now contains
```bash
$cat test
aa
bb
cc
AA
cC
```
Remove duplicate case insensitive. 
This file is not sorted though. So it has to be sorted first before uniq. -i flag is for case in sensitive
```bash
$ sort test | uniq -i
AA
bb
cC
```
**Case conversion**  
Convert all upper case in fileA to lower case and output as fileB
```bash
$ tr '[:upper:]' '[:lower:]' < fileA.txt > fileB.txt
```
**File comparision**  
Compare two files and keep strings present in fileA but not in fileB
```bash
$ comm -23 fileA fileB
```
Compare two files and keep strings present in fileB but not in fileA
```bash
$ comm -13 fileA fileB
```
Compare two files and keep only strings which are present in both files
```bash
$ comm -3 fileA fileB
```

**Sed**   
Primary purpose of sed is string replacement or pattern replacement.
Consider the following file as input
```bash
$ cat file.txt
unix is great os. unix is opensource. unix is free os.
learn operating system.
unixlinux which one you choose.
```
1. Replacing or substituting string
```bash
$ sed 's/unix/linux/' file.txt
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```
By default, the sed command replaces the first occurrence of the pattern in each line and it won't replace the second, third...occurrence in the line.
Here the "s" specifies the substitution operation. The "/" are delimiters. The "unix" is the search pattern and the "linux" is the replacement string.
If you miss a delimiter then the expression errors out as below
```bash
$ sed 's/unix/linux' file.txt 
sed: 1: "s/unix/linux": unterminated substitute in regular expression
``` 
2 Replacing the nth occurrence of a pattern in a line.
Use the /1, /2 etc flags to replace the first, second occurrence of a pattern in a line. The below command replaces the second occurrence of the word "unix" with "linux" in a line.
```bash
$ sed 's/unix/linux/2' file.txt
unix is great os. linux is opensource. unix is free os.
learn operating system.
unixlinux which one you choose.
```
Here is the first occurence which is the default option
```bash
$ sed 's/unix/linux/1' file.txt
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```
And the third occurence
```bash
$ sed 's/unix/linux/3' file.txt
unix is great os. unix is opensource. linux is free os.
learn operating system.
unixlinux which one you choose.
```
To replace all the occurence use 'g' (global replacement)
```bash
$ sed 's/unix/linux/g' file.txt
linux is great os. linux is opensource. linux is free os.
learn operating system.
linuxlinux which one you choose.
```
To make the search case insensitive sed on mac does not have a flag but you can use plain regex to achieve it.
For example modify the file.txt to below
```bash
$ vi file.txt
unix is great os. Unix is opensource. unix is free os.
learn operating system.
Unixlinux which one you choose.
sed 's/[Uu]nix/linux/g' file.txt
linux is great os. linux is opensource. linux is free os.
learn operating system.
linuxlinux which one you choose.
```
How to find a string in all the files contained in a directory. You could use grep or find.
```shell
grep -lr searchStr mydir
grep --recursive --ignore-case --files-with-matches “searchStr" mydir
find mydir -type f | xargs grep -l searchStr
```
To find/replace multiple strings use the -e flag. 
```bash
sed -e 's/unix/linux/g' -e 's/Unix/Linux/g' file.txt
linux is great os. Linux is opensource. linux is free os.
learn operating system.
Linuxlinux which one you choose.
```
To replace a string that begins with a pattern use the regex for it alongwith sed
```bash
sed 's/^learn/learn to use/g' file.txt
unix is great os. Unix is opensource. unix is free os.
learn to use operating system.
Unixlinux which one you choose
```
To remove whitespace characters at end of the line
```bash
sed 's/[<spc><tab>]*|/|/g' file.txt
```
Unix command to know if your file has whitespace or tab characters
```bash
vi file.txt
:set list
```
Unix command to remove BOM (Byte Order Mark) characters from your file
Open the file in binary mode using -b flag to verify if you have BOM. And then remove them
```bash
vi -b file.txt 
:set nobomb
:wq
```
Use the -i flag to overwrite the existing file and create a backup of the original file.
For example to remove all white spaces in a file.
```bash
sed 's/ //g' file.txt
cat file.txt
unixisgreatos.Unixisopensource.unixisfreeos.
learnoperatingsystem.
Unixlinuxwhichoneyouchoose
```
This will create a backup file called file.txt.bak with the original file contents and overwrite file.txt with no spaces
To remove only the trailing spaces in a line use *$. The * character means "any number of the previous character" and $ refers to end of line.
```bash
sed -i .bak 's/ *$//g' file.txt
```
Verify the trailing whitespaces are removed by :set list
```bash
vi file.txt
:set list
unix is great os. Unix is opensource. unix is free os.$
learn operating system.$
Unixlinux which one you choose.$
```
To replace a blank line with something else. You can match a blank line by specifying an end-of-line immediately after a beginning-of-line, i.e. with ^$
```bash
vi file.txt
unix is great os. Unix is opensource. unix is free os.
learn operating system.
Unixlinux which one you choose.
sed 's/^$/this used to be a blank line/' file.txt
unix is great os. Unix is opensource. unix is free os.
this used to be a blank line
learn operating system.
Unixlinux which one you choose.
```
To remove tabs at the end of a line. 
Ex: Add a tab to the end of first line, so :set list will show ^I
```bash
vi file.txt 
unix is great os. Unix is opensource. unix is free os.^I$
learn operating system.$
Unixlinux which one you choose.$
```
To create a tab in your sed command. use ctrl + v and then ctrl + i
```bash
sed -i.bak 's/ *$//' file.txt
vi file.txt
:set list
unix is great os. Unix is opensource. unix is free os.$
learn operating system.$
Unixlinux which one you choose.$
```
Consider file test which contains the following
```bash
$ cat test
(firstname).aa
(firstname).bb
(firstname).bb
(firstname).cc
(firstname).CC
(lastname).hh
(lastname).jj
(lastname).ll
```
To extract the content after firstname
```bash
sed -En 's/.*firstname\)\.([A-Za-z]+).*/\1/p' test
aa
bb
bb
cc
CC
```
**Search Strings**   
Total occurences of searchStr in current directory
```bash
grep -ro searchStr . | wc -l | xargs echo "Total matches :"
```
Total number of files where searchStr occurs in current directory
```bash
grep -lor searchStr . | wc -l | xargs echo "Total matches :"
```
To get an exact word match use the -w flag. 
```bash
grep -lwr searchStr mydir
```
Recursively replace string original with replacement in all files under OSx directory mydir recursively(Excludes hidden files and folders)
```bash
find mydir \( ! -regex '.*/\..*' \) -type f -exec sed -i '' 's/original/replacement/g' {} \;
```
OR
```bash
find mydir \( ! -regex '.*/\..*' \) -type f -exec sed -i '' 's/original/replacement/g' {} +
```
The regex excludes all hidden files and folders which is particularly important if you want to avoid messing up your .DS_Store or .git files unknowningly.
if you use zsh then the following would also work 
```bash
sed -i -- 's/original/replacement/g' **/*(D*)
```
This isnt exlcuding hidden files though. The **/*(D*) is basically zsh way of saying recursively go through all sub directories and all files.
