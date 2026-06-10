# ChromaDB 记忆集成

## 概述

梦境系统的核心洞察（知识三元组、核心记忆、进化补丁）存储到 ChromaDB，
支持语义搜索历史梦境，实现"越做梦越懂你"。

## 安装

```bash
pip install chromadb sentence-transformers
```

## 使用

```python
from hermes_memory import get_memory

mem = get_memory()

# 存储梦境洞察
def store_dream_insights(dream_date, triples, core_memories, patches):
    """将梦境洞察存入 ChromaDB"""
    
    # 存储知识三元组
    for triple in triples:
        mem.add(
            content=f"{triple['subject']} {triple['relation']} {triple['object']}",
            metadata={
                "type": "dream_triple",
                "date": dream_date,
                "subject": triple["subject"],
                "relation": triple["relation"],
                "object": triple["object"]
            }
        )
    
    # 存储核心记忆
    for memory in core_memories:
        mem.add(
            content=memory,
            metadata={
                "type": "dream_core_memory",
                "date": dream_date,
                "importance": 0.9
            }
        )
    
    # 存储进化补丁
    for patch in patches:
        mem.add(
            content=patch,
            metadata={
                "type": "dream_evolution_patch",
                "date": dream_date,
                "importance": 0.8
            }
        )

# 语义搜索历史梦境
def search_dreams(query, n_results=5):
    """搜索历史梦境洞察"""
    return mem.search(query, n_results=n_results)
```

## 数据库位置

`~/.hermes/memory_db/`
