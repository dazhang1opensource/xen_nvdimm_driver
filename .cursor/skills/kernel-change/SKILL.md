---
name: kernel-change
description: "修改 Linux 内核树里的 C、头文件、Kconfig 或 Makefile，以及准备上游补丁时使用。用户提到内核补丁、驱动改动、Signed-off-by、checkpatch、get_maintainer 或用户态 ABI 时也要使用。"
---

# 改内核代码

先确认这棵树的版本，再按这棵树自己的文档改。不要套用更新内核才有的接口或 100 列风格。

- `linux` 这棵树是 5.0-rc1。风格在 `Documentation/process/coding-style.rst` 和 `Documentation/process/submitting-patches.rst`。
- `xen_nvdimm_driver` 是 4.8-rc7。先找 `Documentation/CodingStyle` 和 `Documentation/SubmittingPatches`，没有再找 `Documentation/process/`。
- 这两棵树的 checkpatch 默认行宽都是 80 列。缩进用 tab，tab 宽 8。

## 动手前

1. 用 `git log -20 -- <path>` 看这块代码最近为什么长这样，新代码跟同文件的锁、错误码、分配器和日志宏对齐。
2. 接口以这棵树的头文件为准。找不到的函数、宏、结构体就说明这里没有，不要按更新的内核补一个同名包装。
3. 改 Kconfig 时同时改对应 Makefile，写清依赖。默认值跟同子系统已有选项一致，没有理由就保持 `n`。
4. 导出给用户态的结构放在 `include/uapi/`。给已有 ioctl 或结构体加字段时只追加，并保持旧程序的尺寸和对齐还能用。

## 检查

对本次改动跑，不要为了告警去重排整文件：

```
git diff | scripts/checkpatch.pl --strict
```

## 提交说明

标题一行：`子系统: 一句话说明做了什么`。正文写原因和影响。需要时按同目录最近补丁的顺序带这些标签：

- `Fixes: <12位 sha> ("原标题")`
- `Cc: stable@vger.kernel.org` 只在缺陷适合回移时加
- `Signed-off-by:` 用仓库里已经在用的开发者身份，不要编造名字

收件人用 `scripts/get_maintainer.pl --nogit-fallback` 加上改动文件。发给上游的补丁里不要包含 `.cursor/`。
