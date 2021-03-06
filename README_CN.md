# 小说整理工具

## 为什么要做这个工具

首先声明，本人支持正版，在可能的情况下，请上官网订阅作者。我的这套工具基本都用于全本订阅的小说。

但是，起点等网站会因各种原因下架小说的部分/全部章节，严重影响阅读体验。在这种情况下，不得不在网络上下载txt版本。txt本身没有格式，即使部分阅读软件有格式匹配能够自动生成目录，但部分问题仍然无法解决：

- 无法支持自定义格式的卷名/章节名。

某些小说可能不会采用正统的章节名。例如：「少女之战第X幕」。

- 由于作者疏忽，章节名会有重复/遗漏。

例如出现两个100章，或者100章之后直接变成102章。

- 章节名字的格式不规则。

比如中文数字与阿拉伯数字混用，或者「章」后面是否有空格。

对于强迫症患者来说，这是无法接受的。虽然每个小说的问题各不相同，不太可能可以用一套通用的脚本来处理所有的小说，但是脚本至少可以对章节进行简单的自动分割，再进行手动调整。

## 额外库

你需要`natsort`来对数字进行自然排序（1, 2, ..., 10而非1, 10, 11, ..., 2）。

```shell
pip3 install natsort
```

## 使用方法

- 所有的脚本都有`argparse`，可以通过`-h`参数来查看使用指南。

小说下载下来之后，先删去前后的网站声明、书名、简介等无关内容，只留下卷名和章节名。同时确保文件为**utf8**编码，而非gb2312。

-----

然后使用`split.py`对txt文件进行分卷、分章节。如果小说没有分卷，那么默认会加入“正文”分卷。同时会检测重复或遗漏的分卷/章节名，并在控制台上输出。你可以根据输出来手动对分卷名或章节名进行调整。

使用方式如下：

```shell
python3 split.py -f FILENAME [-o OUT_DIR] [-d]  [-h]
```

其中`-f`是小说文件名，为必选参数，其余为可选参数。`-o`指定输出路径，默认为小说所在的路径；`-d`可以控制重复/遗漏名称检测模块中，在分卷间是否抛弃章节的id。有些小说的分卷会连续编号，而有些小说每一卷都会从第一章开始重新编号。

`split.py`通过不同的`Matcher`来识别卷名和章节名。目前一共有三个不同的`Matcher`：

- `NumberedMatcher`：匹配正则表达式，识别章节名中的数字，并将中文数字转换成阿拉伯数字（以便于排序）。

- `SpecialMatcher`：匹配给定的特殊名称数组，例如”引子“、”终章“。

- `VolumeMatcher`：匹配不规则的分卷名。你需要在小说目录生成一个`volumes.json`来声明卷名。json的格式如下：
  
  ```json
  [
      {
          "name": "生成分卷的文件夹名（需要是合法的文件夹名）",
          "volume": "小说中分卷的标题"
      },
  ]
  ```

如果你的分卷名比较规则，但需要手动排序，那么你可以在生成所有分卷文件夹后，可以使用`generate_order.py`来自动生成这个json文件。该脚本接受一个`-d`参数，指名小说分卷文件夹的根目录。如果分卷的名字不规则，不符合自然排序规则，那么需要手动生成`volumes.json`并对分卷进行排序。

-----

你可以参考`matchers.py`来添加新的`Matcher`。所有的`Matcher`接收一个`args`参数，并通过`match()`方法来判断小说中的每一行是否是卷名/章节名。

脚本默认会从`default_matchers.json`中获取`Matcher`及其参数。如果你需要更复杂的配置，可以将该文件复制到小说目录下，更名为`matchers.json`后进行修改，脚本会自动识别该文件并创建相应的`Matcher`。

-----

如果小说的分卷有简介，那么将其存为`_intro.txt`并放入各自的分卷目录中。因为章节会自然排序，前面的下划线可以保证简介文件在合并时被第一个读取。对于小说的简介，将`_intro.txt`放入主目录下。在合并时脚本会自动检测该文件并将其放入合并过后的文档中。

-----

然后，你就可以使用`concatenate.py`来合并所有的分卷/章节，保存到一个统一的Markdown文件中。

使用方式如下：

```shell
python3 concatenate.py -i IN_DIR [-o OUT_DIR] [-t TITLE_HEADING] [-v VOLUME_HEADING] [-c CHAPTER_HEADING] [-a]
```

其中`-i`是小说目录，为必选参数，其余的参数均可选。`-o`指定输出路径（默认为小说路径）；`-t`，`-v`，`-c`分别指小说、分卷和章节的标题格式（h1-h6）；`-a`可以让你控制是否需要将分卷名添加到合并后的文件中。如果小说没有分卷，保存在**正文**文件夹下，那么你可以使用此选项。**注意**：在使用`-a`后，章节标题会默认提升一级。

-----

现在，你就可以将整理完成的文件导入到Calibre等电子书管理软件中，统一进行管理。你可以手动操作，亦可以使用`add_to_calibre.py`和`calibre_convert.py`。这两个脚本均接受一个`-d`参数，指明小说所在目录。对于这两个脚本，你需要下载一张封面图，保存为`cover.jpg`，同时需要手动生成一个`metadata.json`文件以保存元信息，模板如下：

```json
{
    "title": "My Favorite Book",
    "author": "Best author in the world",
    "id": "isbn:1234567890",
    "tags": ["Fiction"],
    "publisher": "Hello World",
    "languages": ["English"]
}
```
