---
layout: post
date:   2018-05-21 14:48:00
title:  "grep 搜索过滤 "
categories: shell
tags:  linux shell grep
mathjax: true
---

* content
{:toc}
*linux*grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。grep全称是*Global Regular Expression Print*，表示全局正则表达式;grep可用于shell脚本，grep通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。






### 命令格式

``` bash
grep [option] pattern file
```

### 命令参数
Regexp selection and interpretation:<br>
  -E, --extended-regexp     PATTERN is an extended regular expression (ERE)<br>
  -F, --fixed-strings       PATTERN is a set of newline-separated fixed strings<br>
  -G, --basic-regexp        PATTERN is a basic regular expression (BRE)<br>
  -P, --perl-regexp         PATTERN is a Perl regular expression<br>
  -e, --regexp=PATTERN      use PATTERN for matching<br>
  -f, --file=FILE           obtain PATTERN from FILE<br>
  -i, --ignore-case         ignore case distinctions<br>
  -w, --word-regexp         force PATTERN to match only whole words<br>
  -x, --line-regexp         force PATTERN to match only whole lines<br>
  -z, --null-data           a data line ends in 0 byte, not newline<br>

Miscellaneous:<br>
  -s, --no-messages         suppress error messages<br>
  -v, --invert-match        select non-matching lines<br>
  -V, --version             display version information and exit<br>
      --help                display this help text and exit<br>

Output control:<br>
  -m, --max-count=NUM       stop after NUM matches<br>
  -b, --byte-offset         print the byte offset with output lines<br>
  -n, --line-number         print line number with output lines<br>
      --line-buffered       flush output on every line<br>
  -H, --with-filename       print the file name for each match<br>
  -h, --no-filename         suppress the file name prefix on output<br>
      --label=LABEL         use LABEL as the standard input file name prefix<br>
  -o, --only-matching       show only the part of a line matching PATTERN<br>
  -q, --quiet, --silent     suppress all normal output<br>
      --binary-files=TYPE   assume that binary files are TYPE;<br>
                            TYPE is 'binary', 'text', or 'without-match'<br>
  -a, --text                equivalent to --binary-files=text<br>
  -I                        equivalent to --binary-files=without-match<br>
  -d, --directories=ACTION  how to handle directories;<br>
                            ACTION is 'read', 'recurse', or 'skip'<br>
  -D, --devices=ACTION      how to handle devices, FIFOs and sockets;<br>
                            ACTION is 'read' or 'skip'<br>
  -r, --recursive           like --directories=recurse<br>
  -R, --dereference-recursive
                            likewise, but follow all symlinks<br>
      --include=FILE_PATTERN
                            search only files that match FILE_PATTERN<br>
      --exclude=FILE_PATTERN
                            skip files and directories matching FILE_PATTERN<br>
      --exclude-from=FILE   skip files matching any file pattern from FILE<br>
      --exclude-dir=PATTERN directories that match PATTERN will be skipped.<br>
  -L, --files-without-match print only names of FILEs containing no match<br>
  -l, --files-with-matches  print only names of FILEs containing matches<br>
  -c, --count               print only a count of matching lines per FILE<br>
  -T, --initial-tab         make tabs line up (if needed)<br>
  -Z, --null                print 0 byte after FILE name<br>

Context control:<br>
  -B, --before-context=NUM  print NUM lines of leading context<br>
  -A, --after-context=NUM   print NUM lines of trailing context<br>
  -C, --context=NUM         print NUM lines of output context<br>
  -NUM                      same as --context=NUM<br>
      --group-separator=SEP use SEP as a group separator<br>
      --no-group-separator  use empty string as a group separator<br>
      --color[=WHEN],<br>
      --colour[=WHEN]       use markers to highlight the matching strings;<br>
                            WHEN is 'always', 'never', or 'auto'<br>
  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)<br>
  -u, --unix-byte-offsets   report offsets as if CRs were not there<br>
                            (MSDOS/Windows)<br>

### 规则表达式

``` bash
^  #锚定行的开始 如：'^grep'匹配所有以grep开头的行    
$  #锚定行的结束 如：'grep$'匹配所有以grep结尾的行   
.  #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
*  #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
.*   #一起用代表任意字符。   
[]   #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
[^]  #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
\(..\)  #标记匹配字符，如'\(love\)'，love被标记为1。    
\<      #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。    
\>      #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。    
x\{m\}  #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。    
x\{m,\}  #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。    
x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。   
\w    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
\W    #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
\b    #单词锁定符，如: '\bgrep\b'只匹配grep。
```

### POSIX字符
*POSIX(The Portable Operating System Interface)*
``` bash
[:alnum:]    #文字数字字符,[[:alnum:]] 等价于 [A-Za-z0-9]
[:alpha:]    #文字字符   
[:digit:]    #数字字符   
[:graph:]    #非空字符（非空格、控制字符）   
[:lower:]    #小写字符   
[:cntrl:]    #控制字符   
[:print:]    #非空字符（包括空格）   
[:punct:]    #标点符号   
[:space:]    #所有空白字符（新行，空格，制表符）   
[:upper:]    #大写字符   
[:xdigit:]   #十六进制数字（0-9，a-f，A-F）
```

注：以上字符只有放在[]内才能成为正则表达式;example：[[:alnum:]]

### Examples

输出A文件中含有B文件行内容的行
``` bash
cat A |grep -f B
```

从B文件中读出行内容作为关键词在A文件中搜索并标识搜索结果行号
``` bash
cat A | grep -nf B
```

输出IP地址：
``` bash
ifconfig eth0|grep -E "([0-9]{1,3}\.){3}[0-9]"
ifconfig eth0|grep "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}"
```

显示当前目录下面以.txt 结尾的文件中的所有包含每个字符串至少有7个连续小写字符的字符串的行
``` bash
grep '[a-z]\{7\}' *.txt
```

