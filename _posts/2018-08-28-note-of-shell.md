---
  layout: post
  title: shell笔记
  categories: 开发
  tags:
---

数组操作

    #声明空数组
    declare -a date_array=()
    #向数组中添加新元素
    date_array=("${date_array[@]}" $date)
    #获取数组长度
    echo ${#date_array[@]}
    #遍历数组
    for date in ${date_array[@]}; do
        echo $date
    done

### while 读取文件


