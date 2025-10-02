---
date: 2025-10-01
title: LangChain 核心概念入门：文档加载、分割与向量检索
slug: langchain-basics
image: OIP.jpg
categories:
    - langchain
---

## 前言
本文内容基于 LangChain 官方文档进行简化和提炼，主要为个人学习总结。如需更详尽的信息，建议直接查阅[官方文档](https://python.langchain.com/docs)。

## 什么是 LangChain
LangChain 是一个基于大语言模型的应用程序开发框架，旨在简化 LLM 应用生命周期的各个阶段：开发、生产与部署。本文将重点介绍文档与文档加载器、文本分割、嵌入模型以及向量存储与检索的基本使用方法。

## Documents and Document Loaders

### 1. Document

`Document` 对象包含三个主要属性：
- `page_content`: 存储文本内容的字符串
- `metadata`: 包含任意元数据的字典
- `id` (可选): 文档的字符串标识符

```python
from langchain_core.documents import Document

documents = [
    Document(
        page_content="Dogs are great companions, known for their loyalty and friendliness.",
        metadata={"source": "mammal-pets-doc"},
    ),
    Document(
        page_content="Cats are independent pets that often enjoy their own space.",
        metadata={"source": "mammal-pets-doc"},
    ),
]
```

### 2. Document Loader

这里以 `PyPDFLoader` 为例，它适用于处理结构化的 PDF 内容。若需处理非结构化内容（如扫描版 PDF），可考虑使用 `UnstructuredPDFLoader`。

`PyPDFLoader` 将 PDF 的每一页存储为一个独立的 `Document` 对象，便于通过 `page_content` 和 `metadata` 访问内容。

```python
from langchain_community.document_loaders import PyPDFLoader

file_path = "../example_data/nke-10k-2023.pdf"
loader = PyPDFLoader(file_path)
docs = loader.load()

print(len(docs))
print(f"{docs[0].page_content[:200]}\n")
print(docs[0].metadata)
```

输出示例：
```python
107

Table of Contents
UNITED STATES
SECURITIES AND EXCHANGE COMMISSION
Washington, D.C. 20549
FORM 10-K
(Mark One)
☑ ANNUAL REPORT PURSUANT TO SECTION 13 OR 15(D) OF THE SECURITIES EXCHANGE ACT OF 1934
FO

{'source': '../example_data/nke-10k-2023.pdf', 'page': 0}
```

## 文本分割 (Splitting)

直接将整个文档提供给 LLM 不仅效率低下，还会消耗大量 token。因此，我们需要将文档分割成小块，通过检索筛选出与问题最相关的内容片段，再提供给 LLM，从而获得更精准的回答。

这里介绍一种常用的分割方法：`RecursiveCharacterTextSplitter`。它非常适合 RAG 应用场景，能够最大程度保持文本的语义完整性。该方法按照预设的分隔符列表递归地分割文本，直到每个块都符合设定的 `chunk_size`。与 `CharacterTextSplitter` 相比，虽然速度稍慢，但分割效果更智能。

主要参数说明：
- `chunk_size`: 期望的文本块大小
- `chunk_overlap`: 文本块之间的重叠部分大小
- `add_start_index`: 是否为每个块添加在原文本中的起始位置索引

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200, add_start_index=True
)
all_splits = text_splitter.split_documents(docs)

len(all_splits)
```

```python
514
```

## 嵌入 (Embedding)

嵌入的核心作用是将文本转换为高维向量表示，便于后续通过向量相似度计算来检索相关信息（特别是在 Q&A 场景中）。LangChain 支持多种嵌入模型，可以根据具体需求选择。

以下示例使用 HuggingFace 的免费嵌入模型：

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings_model = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")
texts = [doc.page_content for doc in split_docs]
doc_embeddings = embeddings_model.embed_documents(texts)
```

## 向量存储与检索

生成文本嵌入后，需要将其存储起来以便后续查询。这里使用 Chroma 作为向量数据库：

```python
from langchain_chroma import Chroma

vector_store = Chroma(
    collection_name="example_collection",
    embedding_function=embeddings_model,
    persist_directory="./chroma_langchain_db",  # 本地存储路径，如不需要可移除
)
```

存储完成后，即可通过 `similarity_search` 方法查找与查询问题最相关的文档片段。该方法通常基于余弦相似度、欧氏距离或点积等算法计算向量相似度，其中余弦相似度是最常用的方法。

```python
results = vector_store.similarity_search(
    "How many distribution centers does Nike have in the US?"
)

print(results[0])
```

```python
page_content='direct to consumer operations sell products through the following number of retail stores in the United States:
U.S. RETAIL STORES NUMBER
NIKE Brand factory stores 213 
NIKE Brand in-line stores (including employee-only stores) 74 
Converse stores (including factory stores) 82 
TOTAL 369 
In the United States, NIKE has eight significant distribution centers. Refer to Item 2. Properties for further information.
2023 FORM 10-K 2' metadata={'page': 4, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 3125}
```