## 删除文件
---
* **git rm**
  * ***git rm file.txt***
    * 删除暂存区和工作区的指定文件。
  * ***git rm -f file.txt***
    * 如果删除之前修改并已经放到暂存区，必须使用 *-f(--force)*。
  * ***git rm --cached file.txt***
    * 删除暂存区文件，保留工作区文件。