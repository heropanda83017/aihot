# aihot API — Windows Python 调用方式

## 问题

Windows git-bash 中 `curl | python3 -c` 管道存在编码/字符集问题，输出乱码。`terminal()` 工具无法可靠执行包含管道和子命令的复杂 curl 调用链。

## 解决方案：Python urllib.request（在 execute_code 中使用）

```python
import urllib.request, json, datetime
from hermes_tools import execute_code  # 在 execute_code 沙盒中可直接 import

UA = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 aihot-skill/0.2.0"

def fetch_aihot_items(mode="selected", since_hours=24, take=20):
    """拉取 aihot 精选条目，返回 JSON dict"""
    since = (datetime.datetime.utcnow() - datetime.timedelta(hours=since_hours)).strftime('%Y-%m-%dT%H:%M:%SZ')
    url = f"https://aihot.virxact.com/api/public/items?mode={mode}&since={since}&take={take}"
    req = urllib.request.Request(url, headers={"User-Agent": UA})
    with urllib.request.urlopen(req, timeout=10) as resp:
        return json.loads(resp.read().decode('utf-8'))

def fetch_daily(date_str=None):
    """拉取日报，不传 date_str 拉最新"""
    path = f"daily/{date_str}" if date_str else "daily"
    url = f"https://aihot.virxact.com/api/public/{path}"
    req = urllib.request.Request(url, headers={"User-Agent": UA})
    with urllib.request.urlopen(req, timeout=10) as resp:
        return json.loads(resp.read().decode('utf-8'))
```

## 何时用 Python 替代 curl

| 场景 | 推荐方式 |
|------|---------|
| 简单 GET 查询，terminal(bash) 编码正常 | curl（aihot SKILL.md 标准方式） |
| 输出需要结构化处理、时间转换、分类聚合 | Python urllib（execute_code） |
| terminal 出现乱码/管道失败 | Python urllib（execute_code） |
| 需要将结果写入文件（E:盘） | Python open()（因 execute_code write_file 写入沙盒） |

## 注意事项

- `execute_code` 中的 `web_extract`/`web_search` 工具不能替代对 aihot API 的直接调用（前者走搜索引擎/通用爬虫，不走 aihot 精选池）
- Python 方式返回的数据结构与 curl+`| jq` 相同，可直接用 `data['items']` 遍历
- 时间转换：publishedAt 是 ISO 8601 UTC，展示前转北京时间
