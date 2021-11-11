---
layout: post
title: Cython 建立 執行檔
date: 2021-11-11
tags: python Cython
---

CentOS 安裝 理論其他 linux 也可
```
pip3 install Cython
```

/usr/bin/py2c
```
#!/bin/bash

if [ -z $1 ];then
	printf "Usage: \n\t$0 [python source file][swtich] 1: To C code 2: to bytecode 3: to C and bytecode\n"
	exit 0
fi

source_file=$1
tag=$2

fn_ext=$(echo ${source_file} | awk -F '.' '{print $2}'| tr -d '\r\n')
if [ -z ${fn_ext} ]; then
        fn=${source_file}
elif [ ${fn_ext} == "py" ]; then
	fn=$(echo ${source_file} | awk -F '.' '{print $1}')
fi

if [ ${tag} -eq 1 ]; then
	printf "Transfer python source to C.\n"
	cython --embed -o ${fn}.c ${source_file}
elif [ ${tag} -eq 2 ]; then
	printf "Transfer python source to bytecode.\n"
	gcc -Os -I /usr/include/python3.6m -o ${fn} ${fn}.c -lpython3.6m -lpthread -lm -lutil -ldl
elif [ ${tag} -eq 3 ]; then
	printf "Transfer python source to C sode and bytecode\n"
	cython --embed -o ${fn}.c ${source_file}
	gcc -Os -I /usr/include/python3.6m -o ${fn} ${fn}.c -lpython3.6m -lpthread -lm -lutil -ldl
fi
printf "Transfer End\n"
```
檔案 test.py 前面要加 language_level
```
#!python
#cython: language_level=3
...
```

```
py2c test.py 3
```

```
./test
```
