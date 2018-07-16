Linux - DebugFS
==
<br/>

Catch file deleted LSOF

<br/>

```bash
	root@:/tmp # debugfs /dev/mapper/system-root
	debugfs 1.41.12 (17-May-2010)
	debugfs:  dump <565> /tmp/toto
	debugfs:  quit
	
	root@:/tmp # l /tmp/toto
	-rw-r--r-- 1 root root 13K 25 janv. 17:10 /tmp/toto

```
