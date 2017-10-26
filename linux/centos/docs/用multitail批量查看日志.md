# 用MultiTail查看多个日志
参考源码 <https://github.com/flok99/multitail>

* 安装
> 环境：CentOS7
> 
> 下载rpm包 （有rpm包请跳过）   
> `wget http://pkgs.repoforge.org/multitail/multitail-5.2.9-1.el6.rf.x86_64.rpm`  
> rpm方式安装  
> `rpm -ivh multitail-5.2.9-1.el6.rf.x86_64.rpm`  
> 注意：如果wget下载不了,请找vpn[下载](http://pkgs.repoforge.org/multitail/multitail-5.2.9-1.el6.rf.x86_64.rpm)并ftp到服务器

*  常见使用案例  
摘取自 <https://www.vanheusden.com/multitail/examples.php>

#Examples
No examples of coloring are given as that is configurable via the configurationfile. Also for most
commonly used files color schemes have already been designed.  

*  Automatically add new files to a window:  
`multitail -Q /var/log/myprogram/log-2014*.txt` *# 自动将新文件添加到窗口中*

* Merge 2 logfiles in one window:  
`multitail /var/log/apache/access.log -I /var/log/apache/error.log`  *# 在一个窗口中合并2个日志文件*

* Show 3 logfiles in 2 columns:  
`multitail -s 2 /var/log/apache/access.log /var/log/messages /var/log/mail.log` *#在2列中显示3个日志文件*
* Show 5 logfiles while merging 2 and put them in 2 columns with only one in the left column:  
`1.multitail -s 2 -sn 1,3  /var/log/apache/access.log -I /var/log/apache/error.log /var/log/messages  /var/log/mail.log /var/log/syslog` *# 显示5个日志文件，将前两个合并为一个， 分两列，每列分别有1，3个窗口 （-s 列数  -sn n,m... 每列窗口个数）*  
`2.multitail -s 3 -sn 1,1,2  /var/log/apache/access.log -I /var/log/apache/error.log /var/log/messages  /var/log/mail.log /var/log/syslog`  *#显示五个文件，将前两个合并为一个，分三列，每列分别有1，1，2个窗口*
* Merge the output of 2 ping commands while removing "64 bytes received from" from only 1 of them:
`multitail -l "ping 192.168.0.1" -ke "64 bytes from" -L "ping 192.168.0.2"` *# 合并2个ping命令的输出，并忽略"ping 192.168.0.1" 执行结果中的"64 bytes from" 字符串（-l:执行指令 -L：执行指令并将结果追加到末尾 -ke 'xx' 忽略匹配到的字符串）*
* Show the output of a ping-command and if it displays a timeout, send a message to all users currently logged in:  
`multitail -ex timeout "echo timeout | wall" -l "ping 192.168.0.1"`  *# 显示ping命令的输出，如果显示超时，则向当前登录的 所有用户发送消息*
* In one window show all new TCP connections and their state changes using netstat while in the other window displaying the merged access and error logfiles of apache  
`multitail -R 2 -l "netstat -t" /var/log/apache/access.log -I /var/log/apache/error.log`  *# 在一个窗口中，显示所有新的TCP连接及其状态更改，使用netstat，而在其他窗口中显示apache的合并访问和错误日​​志文件*
* As the previous example but also copy the output to the file netstat.log  
`multitail -a netstat.log -R 2 -l "netstat -t tcp" /var/log/apache/access.log -I /var/log/apache/emultitail -l "ping 192.168.0.1" -ke "64 bytes from" -L "ping 192.168.0.2"`  *#同上例，但将输出复制到文件netstat.log*
* rror.log
* Show 2 logfiles merged in one window but give each logfile a different color so that you can easily see what lines are for what logfile:  
`multitail -ci green /var/log/apache/access.log -ci red -I /var/log/apache/error.log`  *#两个文件合并显示，但为其分别设置颜色，方便查看日志具体属于哪个文件*
* Show 3 rssfeeds merged in one window using rsstail 
`multitail -cS rsstail -l "rsstail -n 1 -z -l -d -u http://setiathome.berkeley.edu/rss_main.php" \
        -cS rsstail -L "rsstail -n 1 -z -l -d -u http://www.biglumber.com/index.rss" -cS rsstail \
        -L "rsstail -n 1 -z -l -u http://kernel.org/kdist/rss.xml"`   *#未尝试  注：\ 为连字符，类似于c++中宏定义中的连字符，再输入指令时需要去掉*
* Show a Squid (proxy server) logfile while converting timestamps to something readable  
`multitail -cv squid /var/log/squid/access.log` *#未尝试*
* Display Q-Mail logging while converting the timestamp into human readable format  
`multitail -cv qmailtimestr /var/log/qmail/qmail.smtpd.log` *#未尝试*
* Merge ALL apache logfiles (*access_log/*error_log) into one window:  
`multitail -cS apache --mergeall /var/log/apache/*access_log --no-mergeall -cS apache_error \ --mergeall /var/log/apache/*error_log --no-mergeall`   *#未尝试*
* Monitor the logfile of an other system:
For this you need to setup a couple of things. MultiTail runs on system A, the logfile on system B. In this example we're going to monitor the apache logfile. Add the following to <b>/etc/services</b>:  
apachelog       20000/tcp

Add this to <b>/etc/inetd.conf</b>:  
apachelog stream tcp nowait root /usr/local/sbin/tail_apache_log /usr/local/sbin/tail_apache_log
and create the file <b>/usr/local/sbin/tail_apache_log</b> with the following content:  
`#!/bin/sh`  
`/usr/bin/tail -f /var/log/apache2/access.log`

make sure that you don't forget to make that script executable (chmod +x filename).
Then on host A start MultiTail like this:  
`multitail -cS apache -l "telnet B 20000"`

Please note that logfiles go in plaintext across the network. You may also need to adjust
the files <b>/etc/hosts.[allow|deny]</b> on host B to only allow host A to connect

* Monitoring Tomcat  
`multitail -cS apache -cS log4j "${TOMCAT_HOME}/logs/catalina.out"`


-------------------------------------------------------------------------------
## 附帮助文档（man multitail指令的输出结果）


MULTITAIL(1)                                                                                                            General Commands Manual                                                                                                            MULTITAIL(1)

NAME
       MultiTail - browse through several files at once

SYNOPSIS
       MultiTail [options]

       options: [-cs|-Cs|-c-] [-s] [-i] inputfile [-i anotherinputfile] [...]

DESCRIPTION
       The  program  MultiTail  lets  you view one or multiple files like the original tail program. The difference is that it creates multiple windows on your console (with ncurses). It can also monitor wildcards: if another file matching the wildcard has a more
       recent modification date, it will automatically switch to that file. That way you can, for example, monitor a complete directory of files. Merging of 2 or even more logfiles is possible. It can also use colors while displaying the logfiles (through regular
       expressions),  for  faster  recognition  of what is important and what not. It can also filter lines (again with regular expressions). It has interactive menus for editing given regular expressions and deleting and adding windows. One can also have windows
       with the output of shell scripts and other software. When viewing the output of external software, MultiTail can mimic the functionality of tools like 'watch' and such. When new mail arrives for the current user, the statuslines will become green. To reset
       this "mail has arrived"-state, press ' ' (a space). For help at any time, press F1.

OPTIONS
       -i file
              Select a file to monitor. You can have multiple -i file parameters.  You only need to add -i file in front of a filename if the filename starts with a dash ('-').

       -I file
              Same as -i file but add the output to the previous window (so the output is merged).

       -iw file interval
              -Iw  file  interval  Like  '-i'/'-I'  but expects the parameter to be a wildcard and the second(!) an interval.  Initially MultiTail will start monitoring the first file with the most recent modification time. Every interval it will check if any new
              files were created (or modified) and start tailing that one. *Don't forget* to put quotation marks around the filename as otherwhise the shell will try to substite them!

       -l command
              Command to execute in a window. Parameter is the command. Do not forget to use "'s if the external command needs parameter! (e.g. -l "ping host").

       -L command
              Same as -l but add the output to the previous window (so the output is merged).

       -j     Read from stdin (can be used only once as there is only 1 stdin).

       -J     Same as -j but add the output to the previous window (so the output is merged).

       --mergeall
              Merge all of the following files into the same window (see '--no-mergeall').

       --no-mergeall
              Stop merging all files into one window (see '--mergeall');

       --no-repeat
              When the same line is repeated, it will be suppressed while printing a "Last message repeated x times" message.

       --mark-interval x
              Print every 'x' seconds a mark-line when nothing else was printed.

       -q i path
              Check path for new files with interval 'i', all in new windows. One can enter paths here understood by the shell. E.g. "/tmp/*". Note: do not forget to add quotes around the pathname to prevent the shell from parsing it!

       -Q i path
              Like -q: but merge them all in one window.

       --new-only
              For -q/-Q: only create windows for files created after MultiTail was started.

       --closeidle x
              Close windows when more then 'x' seconds no new data was processed.

       -a x   Write the output also to file 'x' (like 'tee') AFTER it was filtered by MultiTail.

       -A x   Write the output also to file 'x' (like 'tee') BEFORE it was filtered by MultiTail.

       -g x   Send the output also to command 'x' AFTER it was filtered by MultiTail.

       -G x   Send the output also to command 'x' BEFORE it was filtered by MultiTail.

       -S     Prepend merged output with subwindow-number.

       -t title
              With this switch, "title" is displayed in the statusline instead of the filename or commandline.

       -n number_of_lines
              Number of lines to tail initially. The default depends on the size of the terminal-window.

       -r interval
              Restart the command (started with -l/-L) after it has exited. With interval you can set how long to sleep before restarting.

       -R interval
              Restarts a command like -r only this one shows the difference in output compared to the previous run.

       -rc / -Rc interval
              Like -r / -R but clears the window before each iteration.

       -h     The help.

       -f     Follow the following filename, not the descriptor.

       --follow-all
              For all files after this switch: follow the following filename, not the descriptor.

       -fr filter
              Use the predefined filter(s) from the configfile.

       -e     Use the next regular expression on the following file.

       -ex    Use regular expression on the following file and execute the command when it matches. The command gets as commandline parameter the whole matching line.

       -eX    Like '-ex' but only give the matching substring as parameter. This requires a regular expression with '(' and ')'.

       -ec    Use regular expression on the following file and display the matches.

       -eC    Use regular expression on the following file but display everything and display the matches inverted.

       -E     Use the next regular expression on the following files.

       -v     Negate the next regular expression.

       -s x   Splits the screen vertically in 'x' columns.

       -sw x  At what position to split the screen. e.g. '-sw 20,40,,10' (=4 columns)

       -sn x  How many windows per column for vertical split (use with -s or -sw). e.g. '-sn 3,,2'.

       -wh x  Sets the height of a window (advisory: if it won't fit, the height is adjusted).

       -cS scheme
              Show the next given file using the colorscheme selected with 'scheme' (as defined in multitail.conf).

       -CS scheme
              Show all following files using the colorscheme selected with 'scheme' (as defined in multitail.conf).

       -csn   Extra switch for the following switches; do not use reverse (inverted) colors.

       -cs    Show the next given file in colors (syslog).

       -c     Show the next given file in colors.

       -Cs    Show all following files in color (through syslog-scheme).

       -C     Show all following files in color.

       -Cf field_index delimiter
              Show all following files in color depending on field selected with field_index. Fields are delimited by the defined delimiter.

       -cf field_index delimiter
              Show the next file in color depending on field selected with field_index. Fields are delimited by the defined delimiter.

       -ci color
              Use a specific color. Usefull when merging multiple outputs.

       -cT terminalmode
              Interpret terminal codes. Only ANSI supported at this time.

       -c-    Do NOT colorize the following file.

       -C-    Do NOT colorize the following files.

       -ts    Add a timestamp to each line (format is configurable in multitail.conf).

       -Z color
              Specify the color-attributes for the markerline.

       -T     A timestamp will be placed in the markerline.

       -d     Do NOT update statusline.

       -D     Do not display a statusline at all.

       -du    Put the statusline above the data window.

       -z     Do not display "window closed" windows.

       -u     Set screen updateinterval (for slow links).

       -m nlines
              Set buffersize Set nlines to 0 (zero) if you want no limits on the buffering.

       -mb x  Set scrollback buffer size (in bytes, use xKB/MB/GB).

       -M nlines
              Set the buffersize on ALL following files.

       -p x [y]
              Set linewrap: a = print everything including linewrap. l = just show everything starting at the left until the rightside of the window is reached. r = show everything starting from the right of the line. s = show everything starting  with  the  pro-
              cessname. S = show everything starting after the processname. o = show everything starting at offset 'y'.

       -P x [y]
              Like -p but for all following windows.

       -ke x  Strip parts of the input using regular expression 'x'.

       -kr x y
              Strip parts of the input starting at offset x and ending (not including!) offset y.

       -kc x y
              Strip parts of the input: strip column 'y' with delimiter 'x'.

       -ks x  Use editscheme 'x' from configfile.

       -w     Do not use colors.

       -b n   Sets the TAB-width.

       --config filename
              Load the configuration from given filename.

       -x     Set  xterm-title: %f will be replaced with the last changed file, %h with the hostname, %l with the load of the system, %m with "New mail!" when the current user has new mail, %u with the current effective user, %t timestamp of last changed file, %%
              with a %

       -o configfile-item
              Proces a configurationfile item via the commandline in case you cannot edit the default configfile.

       --cont Reconnect lines with a '? at the end.

       --mark-interval interval
              When nothing comes in, print a '---mark---' line every 'interval' seconds.

       --mark-change
              When multiple files are merged an multitail switches between two windows, print a markerline with the filename.

       --no-mark-change
              Do NOT print the markerline when the file changes (overrides the configfile).

       --label text
              Put "text" in front of each line. Usefull when merging multiple files and/or commands.

       --retry
              Keep trying to open the following file if it is inaccessible.

       --retry-all
              Like --retry but for all following files.

       -cv x  Use conversion scheme 'x' (see multitail.conf).

       --basename
              Only display the filename (and not the path) in the statusline.

       -F file
              Use 'file' as configfile (instead of default configfile).

       --no-load-global-config
              Do NOT load the global configfile.

       --beep-interval x
              Let the terminal beep for every x-th line processed. Press 'i' in the main menu to see how many times it beeped.

       --bi x Like '--beep-interval' but only for current (sub-)window. Statistics on the number of beeps can be found in the statistics for this (sub-)window. Press 't' in the main menu.

       -H     Show heartbeat (to keep your sessions alive).

       -V     Show the version and exit.


KEYS
       You can press a couple of keys while the program runs.  To see a list of them, press F1 (or ^h). You can press F1 (or ^h) at any time: it gives you context related information.  Press 'q' to exit the program.


EXAMPLES
       See http://www.vanheusden.com/multitail/examples.html for more and other examples.

       multitail /var/log/apache/access_log logfile -i -filestartingwithdatsh
              This creates three windows. One with the contents of /var/log/apache/access_log, one with the contents of logfile and so on.

       multitail -R 2 -l "netstat -t"
              This runs netstat every 2 seconds and then shows what has changed since the previous run. That way one can see new connections being made and closed connections fading away.

       multitail logfile -l "ping 192.168.1.3"
              This creates two windows. One with the contents of logfile, one with with the output of 'ping 192.168.1.3'.

       multitail /var/log/apache/access_log -I /var/log/apache/error_log
              This creates one window with the contents of /var/log/apache/access_log merged with the contents of /var/log/apache/error_log.

       multitail -M 0 /var/log/apache/access_log -I /var/log/apache/error_log
              Same as previous example. This example will store all logged entries in a buffer so that you can later on browse through them (by pressing ' b
               ').

BUGS
       As this program grew larger and larger over the time with new functionality sometimes added ad-hoc, some bugs may have been introduced. Please notify folkert@vanheusden.com if you find any.

       Well, except for the resizing of your terminal window. The program might crash when doing such things. Upgrading the ncurses library to at least version 5.3 might help in that case.

SEE ALSO
       http://www.vanheusden.com/multitail/

NOTES
       This page describes MultiTail as found in the multitail-4.3.1 package; other versions may differ slightly.  Mail corrections and additions to folkert@vanheusden.com.  Report bugs in the program to folkert@vanheusden.com.

MultiTail                                                                                                                       2007-02                                                                                                                    MULTITAIL(1)
## THE END
