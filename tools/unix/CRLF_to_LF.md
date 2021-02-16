# ��� �������� ��������� ����� � ������� � ������ ^M . ��� �������� ��������� �����

����� � ASCII-��������� ��� ����������� ������ ��������, ��� ����������� ����� ������ ���������� �������:
<ul>
 	<li>LF (�� ����. Line feed (������� ������), 0x0A)</li>
 	<li>CR (�� ����. Carriage Return, 0x0D)</li>
 	<li>CRLF (�.�. ��� �������).</li>
</ul>
� windows-�������� ������ ������������ CRLF, � unix - LF . � web-���������� ��������������� ������������ unix-�������, � ��� ���������, �������, ����� � ������ ��������� ����� ������������ � ������� LF.

��� �� �����, ������ ���������� ���� ��� �������������� � ������ ���:
<pre>FOREACH row = services;^M
    SET service_row = {^M
        attr =&gt; {},^M
        list =&gt; [],^M
    };^M
^M
</pre>
��� ��������, ��� ���-�� �������������� ���� � ��������� ��������� ��� Windows. �������� ������ ����� ����� �������� � ���������. ����� ����, ����� ����� ����� �������� �������������, �������� � unix-�����. ��������� �������� ^M �������� � ��������� ������� ����� � ������������� ��� ����������� ������.


## ������� dos2unix � unix2dos

��� ����������� �������������� ������� ����� ������ ������������ ������� dos2unix . ��� ������������� ����� ��������� � �������� ��������������, � ������� unix2dos .

���������:
<pre>$ sudo apt-get install dos2unix
$ sudo apt-get install unix2dos
</pre>
�� ���������, ������� dos2unix ����������� ��������� ����. �� ����� ������������ ���� -n � ����� ��������������� ����� ����� �������� � ������ �����, �������� �������� ����� ������� � �������� ������� ���������.
<pre>$ dos2unix filename.txt
dos2unix: converting file filename.txt to Unix format ...
</pre>
��� ��������� �������������� ������ ����� ������������ �����:
<pre>$ dos2unix -k *.inc
</pre>
���� -k �������� �������� ��� ��������� ����� �������� �����.

����� ������ ����������� ��������� ����� ������� ����� ����, ������� ����� ��������� ������ � ������ �������:
<pre>$ dos2unix -n in.txt out.txt
</pre>
������������� unix2dos ���������� ������� dos2unix:
<pre>$ unix2dos filename.txt
</pre>

## ������� todos � fromdos

������ �������, ��� �� ��� ������� dos2unix � unix2dos, ��������� ����������� ������ �� unix-������� � windows-������, � �������.

<i>��� ��� ������ ������ ������������ dos2unix, ������� fromdos � �� �������������, �� ������� ����� ����� ����� �����������.</i>

## ������� tr

<pre>$ cat service.inc | tr -d '\r' &gt; memo.txt</pre>
�.�. ������� tr ������ ������� "\r" �� �����.

## �������������� ����� ����� � ������ � ������� perl

����� �������� windows-������ ��������� �����  (CRLF) �� unix-������(LF), ����� ������������ perl:
<pre># perl -pi -e 's/\r\n/\n/;' filename.txt</pre>
�������� �������������� �� unix-������� (LF) � windows-������ (CRLF):
<pre># perl -pi -e 's/\n/\r\n/;' filename.txt</pre>
��� �������������� ����� ���������� ������, ����� ������������ �����, ������ ������� �������� ����� �����:
<pre># perl -pi -e 's/\n/\r\n/;' b*</pre>
��� �������� �����, ������� �� ������������, ������� �� ���������� �������� � ������
<b>"No such file or directory"</b>:
<pre># perl -pi -e 's/\n/\r\n/;' "b*"
Can't open b*: No such file or directory.
</pre>
���� �� ���������� ��������� �������������� ����� ����� ��� windows, �� �������� ������
<b>"Can't do inplace edit without backup"</b>:
<pre>&gt; perl -pi -e 's/\r\n/\n/;' filename.txt
Can't do inplace edit without backup.
</pre>
��� ������� � ������������� windows. ����� ������������ ��� ����� �������:
<pre>perl -i.bak -p -e 's/\r\n/\n/;' filename.txt &amp;&amp; del filename.txt.bak</pre>


## ��������� ��������� ����� � ������� ������� iconv

����� �������������� ���� in.txt �� WINDOWS-1251 � UTF-8 out.txt, ����� ������������ ������� iconv ( �� ������� Linux Ubunta ):
<pre># iconv -f WINDOWS-1251 -t UTF-8 in.txt &amp;&gt; out.txt</pre>
����� ���������� ������ ������ ���������, � �������� ������� ����� ��������, ����� ������� iconv � ���������� --list :
<pre># iconv --list
The following list contains all the coded character sets known.  This does
not necessarily mean that all combinations of these names can be used for
the FROM and TO command line parameters.  One coded character set can be
listed with several different names (aliases).

  437, 500, 500V1, 850, 851, 852, 855, 856, 857, 860, 861, 862, 863, 864, 865,
  866, 866NAV, 869, 874, 904, 1026, 1046, 1047, 8859_1, 8859_2, 8859_3, 8859_4,
  8859_5, 8859_6, 8859_7, 8859_8, 8859_9, 10646-1:1993, 10646-1:1993/UCS4,
  ANSI_X3.4-1968, ANSI_X3.4-1986, ANSI_X3.4, ANSI_X3.110-1983, ANSI_X3.110,
...
</pre>

