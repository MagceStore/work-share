# Mastering VIM
## Why Vim
    历史悠久，自古U/L自带vi，目前vi有很多版本(elvis, vile, nvi).
## Download Vim
    [Vim Org](http://www.vim.org)
## 常用的 Mode
    * normal mode
    * insert mode (i)
    * command mode (:)
    * visual mode (v)
## open files

    ```
    vim file1 file2 file3
    ```

    normal mode :
                 * :n next file
                 * :N previous file
                 
    ```
    vim file1
    :e file2 file3 
    :files / :ls
    :e## / ctrl + ^
    :b + [buffer number]
    ```

    normal mode : 
                 * :bn next file
                 * :bN previous file
                 * :bl last file

    ```
    vim filename(游标会出现在文档的最后)
    vim n filename(游标会出现在文档的第n行的行首)
    vim /string filename(游标会出现在第一个符合搜索条件的字符串上，按n可以继续查找)
    ```

## 常用命令
    * :w :q :e :q!
    * :wq :x == ZZ :w filename(另存为)
    * h j k l (backspace / space 折行)
    * ctrl + f / b
    * 0 ^ $ G(end) gg(start)
    * w e b(word step)
    * H M L
    * % {} [] () step
    * i a o I A O J(join)
    * x X dd dG dgg D d0 (delete)
    * r R 
    * u ctrl + r
    * >> <<  简单的排版(:set shiftwidth)
    * :ce :ri :le
    * yy 2yy y^ y$ yG y1G p P
    * / ? n N
    * *(搜索游标所在的单词) ## 
    * :[range]s/pattren/string/[c(confirm), e(error), g(整行), i(ignore)]
    * :f 修改file name
    * :r 在游标所在处插入文档内容
    * v V ctrl + v (d y c)
    * :! 外部命令
    * :!! 执行前一次外部命令
    * @: 执行前一次冒号命令
    * :sh 执行shell (:set shell)
    * :r !command 在游标处插入命令的执行结果
    * :n,mw !command n-m行的内容作为 command的input
    * k 游标所在单词的 man page

    
## 分屏
    * vim -on file1 file2 (o小写 n分屏个数) 
    * ctrl + w n
    * ctrl + w s == :sp
    * ctrl + w q 关闭当前分屏
    * ctrl + w j / k  下／上
    * vim -On file1 file2 (O大写 n分屏个数)
    * :vsp 

## 缓存区
    * :reg 所有缓存区的内容
    * "ayy 将本行文字复制到 a缓存区 "ap 将a缓存区的内容贴上
    * 5"ayy  复制5行内容到 a 缓存区 5"Ayy 再复制5行增加到a缓存区内容之后

## 书签Mark
    * :marks 查看所有标记
    * mx x26个英文字母标记 ｀x 回到x . 'x 回到x行首
    * mx x小写 作用于文档内部  X大写 作用于各个文档之间

## 加密
    * vim -x filename 给文件加密 解密 :set key= (:set noswf)
