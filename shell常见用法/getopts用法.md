getpots是Shell命令行参数解析工具，旨在从Shell Script的命令行当中解析参数。getopts被Shell程序用来分析位置参数，option包含需要被识别的选项字符，**如果这里的字符后面跟着一个冒号，表明该字符选项需要一个参数，其参数需要以空格分隔**。冒号和问号不能被用作选项字符。

示例：

```
while getopts 'k:v:n:s:m:' OPT; do
    case $OPT in
        k)
            nodelabelkey="$OPTARG";;
        v)
            nodelabelvalue="$OPTARG";;
        n)
            kubereserved="$OPTARG";;
        s)
            systemreservedratio="$OPTARG";;
        m)
            magnification="$OPTARG";;
        ?)
            echo "Usage: `dirname $0`/entrypoint.sh -k xxx -v xxx -n xxx -s xxx -m xxx"
    esac
done
```

