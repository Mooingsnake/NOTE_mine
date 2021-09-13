### 字符集
https://www.liaoxuefeng.com/wiki/1016959663602400/1017075323632896

unicode：万国码，把所有国家语言的字符都编入其中，因为太多了所以要分区，称为Plane(平面),下面图片里面Scope代表分区的范围，而且是32bit的unicode

日常生活中的所有字符，包括中文，都在plane 0，可见用标准unicode是十分浪费的。

![v2-81d041143488f901271210e456b50945_r](https://user-images.githubusercontent.com/47411365/133041865-c8ded012-b11a-4c5b-a610-6778865f63a8.jpg)

字体：https://zhuanlan.zhihu.com/p/53036815  ，虽然没啥用，反正字体是另外一张表格，和unicode关系不大

UTF-8并不是存储8bit的字符，utf-8是可变长的，在表示a的时候是8bit，在表示汉字“中”的时候是24bit，utf-8还支持ascii，二者可以无缝对接（也就是说前 127个编码和ascii一毛一样）

