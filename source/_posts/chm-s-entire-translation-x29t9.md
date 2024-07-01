---
title: CHM整本翻译
date: '2024-06-27 00:18:27'
updated: '2024-07-01 19:25:45'
permalink: /post/chm-s-entire-translation-x29t9.html
comments: true
toc: true
---

# CHM整本翻译

## 新建项目

打开<kbd>Sisulizer 4</kbd>​软件，<kbd>新建项目</kbd>​  
​![image](http://127.0.0.1:6806/assets/image-20240627002036-jvnbk8q.png)​

​![image](http://127.0.0.1:6806/assets/image-20240621223801-5goiazw.png)​

​![image](http://127.0.0.1:6806/assets/image-20240623013927-fovqm31.png)​

保存项目

### 导出文件（Excel）

> 导出Excel的话对文件有限制，xls文件格式较老，只能导出60000多行，比较局限，该用XLF文件，无上限，见导出文件（XLF）

​![image](http://127.0.0.1:6806/assets/image-20240621224837-6ef2mf9.png)​

​![image](http://127.0.0.1:6806/assets/image-20240621224926-f63ygxx.png)​

打开导出的*.xls文件

​![image](http://127.0.0.1:6806/assets/image-20240621225432-wdzjoqw.png)​

打开趣卡翻译，翻译好B列，将译文复制到C列

在Sisulizer 4中导入刚才的*.xls文件即可

### 导出文件（XLF）

> 目前2023.11.30版本的趣卡翻译多行还未调整，不能保持原格式，多行的原文会翻译成一行的译文。可以先排除掉代码块的翻译。排除掉的则不会导出到XLF中  
> ​![image](http://127.0.0.1:6806/assets/image-20240627002614-050lyye.png)​

​![image](http://127.0.0.1:6806/assets/image-20240623014303-6t5i3pa.png)  
后面都是默认，点击<kbd>完成</kbd>​导出

#### XLF转EXCEL

```Python
import re
import html
import pandas as pd
from collections import OrderedDict

def parse_xlf_to_excel(xlf_path: str, excel_path: str):
    if not xlf_path.endswith('.xlf'):
        raise ValueError(f'所给文件 {xlf_path} 不是 xlf 文件')

    with open(xlf_path, encoding='utf-8') as f:
        content = f.read()

    origen_words = re.findall(r'<source[^>]*>(.*?)</source>', content, re.DOTALL)
  
    # 使用 OrderedDict 保留顺序并去重
    words_dict = OrderedDict((html.unescape(word), None) for word in origen_words if word)
    words = list(words_dict.keys())

    df = pd.DataFrame(words, columns=['Original Text'])
    df['Translated Text'] = ''
    df.to_excel(excel_path, index=False)
    print(f'原文已写入 {excel_path}')

# 示例调用
xlf_path = r'D:\Desktop\trans\test\readarx.xlf'
excel_path = r'D:\Desktop\trans\test\translations.xlsx'
parse_xlf_to_excel(xlf_path, excel_path)
```

上述脚本会提取出所有的`<source></source>`​内的内容到translations.xlsx文件中

将translations.xlsx拖入趣卡翻译中翻译，将译文复制到translations.xlsx的B列

> 趣卡翻译多行还未调整，不能保持原格式，多行的原文会翻译成一行的译文。等新版本更新

```Python
import pandas as pd
import html
import re
from collections import OrderedDict

def write_translations_to_xlf(xlf_path: str, excel_path: str, output_xlf_path: str):
    df = pd.read_excel(excel_path)
    translations = OrderedDict(zip(df['Original Text'], df['Translated Text']))

    with open(xlf_path, encoding='utf-8') as f:
        content = f.read()

    def replace_source_with_target(match):
        source_text = html.unescape(match.group(1))
        target_text = translations.get(source_text, '')
        print(f'Matching source: {source_text}')
        print(f'Translation: {target_text}')
        if target_text:
            return f'<source>{match.group(1)}</source>\n\t\t\t\t\t<target>{html.escape(target_text)}</target>'
        return match.group(0)

    # 匹配 <source> 和 <target> 标签，并在 <target> 标签中插入翻译后的内容
    new_content = re.sub(r'<source[^>]*>(.*?)</source>\s*<target>.*?</target>', replace_source_with_target, content, flags=re.DOTALL)

    with open(output_xlf_path, 'w', encoding='utf-8') as f:
        f.write(new_content)
    print(f'翻译后的内容已写入 {output_xlf_path}')


# 示例调用
xlf_path = r'D:\Desktop\trans\test\readarx.xlf'
excel_path = r'D:\Desktop\trans\test\translations.xlsx'
output_xlf_path = r'D:\Desktop\trans\test\readarx_translated.xlf'
write_translations_to_xlf(xlf_path, excel_path, output_xlf_path)
```

上述代码会从translations.xlsx中匹配译文写入到readarx.xlf下的`<target></target>`​标签内

> 代码不晚上，需替换三个值，1、将`&quot;`​替换为`"`​；2、将`&#x27;`​替换为`'`​；3、将`<target>  `​替换为`<target>`​，主要就是去除译文前的两个空格。4.中文分号`；`​替换为英文分号`;`​，不请出原因，在EXCEL转XLF后，在</target>标签前方会产生中文分号`；`​  
> ​![image](http://127.0.0.1:6806/assets/image-20240627003035-19fbygb.png)​

> 上方排除掉代码块后似乎上方的双引号和单引号则不必替换。

最后在Sisulier 4中导入译文即可

## 编译译文Chm

点击用<kbd>选定的语言构建所选的来源</kbd>​重新编译chm

​![image](http://127.0.0.1:6806/assets/image-20240621231405-rsoccy0.png)​

## quicker实现

1. 选中*.xlf文件点击<kbd>XLF2EXCEL</kbd>​动作  
    ​![image](http://127.0.0.1:6806/assets/image-20240626103332-lv0j4j9.png)​
2. 会在同目录下生成translations.xlsx，在第二列粘贴译文保存即可
3. 选中*.xlf文件点击<kbd>EXCEL2XLF</kbd>​动作  
    ​![image](http://127.0.0.1:6806/assets/image-20240626103608-vrtay6g.png)​

## 替换样式

可以在WinCHM Pro中先转成单个html后，在浏览器中修改样式，然后将CSS代码在CHM Editor中替换  
​![image](http://127.0.0.1:6806/assets/image-20240627003255-64i51vn.png)​

​![image](http://127.0.0.1:6806/assets/image-20240627003351-vzskdru.png)​
