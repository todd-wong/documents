## Vim常用的快捷键

Ctrl + c或Ctrl + [代替Esc键

* ### 字符移动

  * h &lt;--&gt;左
  * j &lt;--&gt;下
  * k &lt;--&gt;上
  * l &lt;--&gt;右
* ### 单词移动

  * b 移动光标到上个单词的词首
  * w 移动到下个单词的词首
  * e 移动到下个单词的结尾
  * ge 移动上个单词的结尾
* ### 行首/行尾

  * 数字0 移动到行首
  * $ 移动到行尾
  * ^ 移动到行首第一个非空白字符
* ### H/M/L

  * H 当前窗口顶部
  * M 当前窗口中间
  * L 当前窗口底部
* ### 查找

  * / 向下查找
  * ？向上查找
* ### 翻页

  * ctr + b 向上翻页
  * ctr + f 向下翻页
* ### 文件中移动

  * gg 移动到文件第一行
  * G 移动文件最后一行
* ### 删除

  * D 删除到行尾
  * X 删除前一个字符
  * x 删除后一个字符
  * d^ 删除至行首
  * d$ 删除至行尾
  * dd 剪切当前行
  * dw 从光标处剪切至一个单子/单词的末尾，包括空格
  * de 从光标处剪切至一个单子/单词的末尾，不包括空格

* ### 复制

  * \#yy 复制从光标所在行往下数\#行文字
  * yw 从光标处复制至一个单子/单词的末尾，包括空格
  * ye 从光标处复制至一个单子/单词的末尾，不包括空格
  * y^或y0 从当前光标位置（不包括光标位置）复制之行首
  * y$ 从当前光标复制到行末
  * p 粘贴
* ### 大小写转换

  * gUU 将当前行字母改为大写
  * guu 将当前行字母改为小写
  * gUw 将当前行单词改为大写
  * guw 将当前行单词改为小写
  
* ### indent折叠
  * zc 折叠
  * zC 对所在范围内所有嵌套的折叠点进行折叠
  * zo 展开折叠
  * zO 对所在范围内所有嵌套的折叠点展开
  



