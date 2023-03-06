<iframe 
	border=0
	frameborder=0 
	height=800 
	width=1200
	src="https://www.runoob.com/linux/linux-comm-read.html">
</iframe>
**读取文件**
```bash
#!/bin/bash
  
count=1    # 赋值语句，不加空格
cat test.txt | while read line      # cat 命令的输出作为read命令的输入,read读到>的值放在line中
do
   echo "Line $count:$line"
   count=$[ $count + 1 ]          # 注意中括号中的空格。
done
echo "finish"
exit 0
```

