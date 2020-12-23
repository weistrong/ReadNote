## .gitignore 格式规范
> GitHub有针对特定语言的 [.gitignore文件列表](https://github.com/github/gitignore)

* 所有空行或以 *#* 开头的行都会被Git忽略。
* 可以使用标准的 *glob* 模式匹配。
* 匹配模式可以以 */* 开头防止递归。
* 匹配模式可以以 */* 结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加 *！* 取反。
