---
layout: post
title: minDoc迁移到wikijs
tags: python, wiki
categories: python
---

### minDoc导出项目markdown：
- 项目设置中，开启导出
- 项目文档页面，右上角，下载markdown


### markdown多级目录迁移到根目录下：
```python
# coding=utf8

import os
import shutil

# --- 配置 ---
SOURCE_DIR = './NameAaaa'  # 你的原始多级目录
OUTPUT_DIR = './NameAaaa'  # 移动后的目标一级目录

def flatten_md_files(src, dest):
    # 如果目标目录不存在则创建
    if not os.path.exists(dest):
        os.makedirs(dest)
        print(f"创建目标目录: {dest}")

    file_count = 0
    
    # os.walk 递归遍历所有子目录
    for root, dirs, files in os.walk(src):
        # 排除目标目录自身，防止无限循环
        if os.path.abspath(root) == os.path.abspath(dest):
            continue

        for filename in files:
            # 过滤条件：必须是 .md 结尾，且不能是 README.md
            if filename.endswith('.md') and filename.upper() != 'README.MD':
                file_path = os.path.join(root, filename)
                target_path = os.path.join(dest, filename)

                # 处理重名冲突：如果 target_path 已存在，则加上路径前缀或数字
                if os.path.exists(target_path):
                    name, ext = os.path.splitext(filename)
                    # 简单处理：使用父文件夹名作为前缀避免冲突
                    parent_name = os.path.basename(root)
                    new_filename = f"{parent_name}_{name}{ext}"
                    target_path = os.path.join(dest, new_filename)
                    print(f"检测到冲突，重命名为: {new_filename}")

                # 执行移动（若想保留原文件，请将 shutil.move 改为 shutil.copy）
                shutil.move(file_path, target_path)
                print(f"已移动: {filename}")
                file_count += 1

    print(f"\n处理完成！共移动了 {file_count} 个文件到 {dest}")

if __name__ == "__main__":
    flatten_md_files(SOURCE_DIR, OUTPUT_DIR)
```


### 格式化md文件：
```python
# coding=utf8

import os
import re
import datetime

# --- 配置 ---
TARGET_DIR = './NameAaaa' 
# 生成符合 Wiki.js 要求的 ISO 8601 时间戳
CURRENT_TIME = datetime.datetime.now(datetime.timezone.utc).isoformat(timespec='milliseconds').replace('+00:00', 'Z')
CURRENT_TIME = "2022-12-13T06:23:36.498Z"

def process_md_files(directory):
    # 优化后的正则表达式：
    # 1. (?:####|\*\*) : 匹配 #### 或 ** (非捕获组)
    # 2. \s*接口URL\s*(?:\*\*)? : 匹配“接口URL”，并兼容末尾可能的 **
    # 3. \s+>\s+ : 匹配引用符号
    # 4. (?:.*/)? : 忽略路径前缀（如 drama/user/）
    # 5. ([\w_]+) : 捕获最终的接口名
    pattern = re.compile(r'(?:####|\*\*)\s*接口URL\s*(?:\*\*)?\s+>\s+(?:.*/)?([\w_]+)', re.MULTILINE)

    for root, dirs, files in os.walk(directory):
        for filename in files:
            if filename.endswith('.md'):
                file_path = os.path.join(root, filename)
                
                try:
                    with open(file_path, 'r', encoding='utf-8') as f:
                        content = f.read()

                    # 匹配规则
                    match = pattern.search(content)
                    if match:
                        interface_name = match.group(1)
                        
                        # 检查是否已经存在 Wiki.js 头部，避免重复处理
                        if content.startswith('---'):
                            print(f"跳过 {filename}: 已包含元数据头部")
                            continue

                        # 构造 Wiki.js YAML 头部
                        header = (
                            "---\n"
                            f"title: {interface_name}\n"
                            f"description: {interface_name}\n"
                            "published: 1\n"
                            f"date: {CURRENT_TIME}\n"
                            "tags: \n"
                            "editor: markdown\n"
                            f"dateCreated: {CURRENT_TIME}\n"
                            "---\n\n"
                        )

                        new_content = header + content

                        # 写入文件
                        with open(file_path, 'w', encoding='utf-8') as f:
                            f.write(new_content)

                        # 重命名文件
                        new_filename = f"{interface_name}.md"
                        new_file_path = os.path.join(root, new_filename)
                        
                        if file_path != new_file_path:
                            # 如果目标文件名已存在，先删除旧的（或根据需要重命名）
                            if os.path.exists(new_file_path):
                                os.remove(new_file_path)
                            os.rename(file_path, new_file_path)
                            print(f"成功: {filename} -> {new_filename}")
                    else:
                        print(f"未匹配: {filename} (未找到指定的接口URL格式)")
                        
                except Exception as e:
                    print(f"处理文件 {filename} 时出错: {e}")

if __name__ == "__main__":
    if not os.path.exists(TARGET_DIR):
        print(f"错误: 找不到目录 {TARGET_DIR}")
    else:
        process_md_files(TARGET_DIR)
        print("\n转换任务处理完毕！")
```
