# คู่มือการใช้งาน RAG Pipeline สำหรับค้นหาข้อมูลหลักสูตร

## บทนำหน้า

เอกสารนี้จัดทำขึ้นเพื่อให้คำแนะนำในการใช้งาน RAG (Retrieval-Augmented Generation) Pipeline ที่พัฒนาขึ้นสำหรับค้นหาข้อมูลหลักสูตรมหาวิทยาลัยย โดยมีวัตถุประสงค์เพื่อให้ผู้ใช้งานสามารถนำไปประยุกต์ใช้งานได้อย่างถูกต้องและมีประสิทธิผล

## ภาพรวมของระบบ

RAG Pipeline นี้ประกอบด้วยส่วนประกอบหลักๆ ดังนี้:

1. **การประมวลผลเอกสาร (Document Processing)**
2. **การสร้างเวกเตอร์ดาตาเบส (Vector Database)**
3. **การค้นหาและดึงข้อมูล (Retrieval)**
4. **การสร้างคำตอบ (Generation)**

## การติดตั้งระบบ

### 1. การติดตั้ง Library ที่จำเป็น

```bash
pip install -U \
  "langchain==0.3.*" \
  "langchain-community==0.3.*" \
  "langchain-openai==0.3.*" \
  "langchain-experimental==0.3.4" \
  "sentence-transformers>=3,<4" \
  "faiss-cpu>=1.8" \
  "pythainlp"
```

### 2. การตั้งค่า API Keys

ต้องตั้งค่า API keys สำหรับการเชื่อมต่อกับบริการต่างๆ:

```python
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
```

## การเตรียมข้อมูล

### โครงสร้างไฟล์ข้อมูล

ข้อมูลหลักสูตรควรจัดเก็บในรูปแบบไฟล์ข้อความ (.txt) โดยมีโครงสร้างดังนี้:

```
data/
├── cs_course.txt          # ข้อมูลหลักสูตรวิทยาการคอมพิวเตอร์
├── cy_course.txt          # ข้อมูลหลักสูตรความมั่นคงปลอดภัยไซเบอร์
├── ai_course.txt          # ข้อมูลหลักสูตรปัญญาประดิษฐ์
├── it_course.txt          # ข้อมูลหลักสูตรเทคโนโลยีสารสนเทศ
├── gis_course.txt         # ข้อมูลหลักสูตรภูมิสารสนเทศศาสตร์
└── el.txt               # ข้อมูลวิชาเลือกทั้งหมด
```

### การจัดรูปแบบข้อมูลหลักสูตร

ข้อมูลหลักสูตรควรมีส่วนประกอบหลักๆ ดังนี้:

1. **รหัสและชื่อหลักสูตร**
2. **ปรัชญาและวัตถุประสงค์**
3. **โครงสร้างหลักสูตร**
4. **ระบบการจัดการศึกษา**
5. **แผนการศึกษา**
6. **คำอธิบายรายวิชา**
7. **เกณฑ์การสำเร็จการศึกษา**
8. **ข้อมูลอาจารย์**

## การใช้งานระบบ

### 1. การสร้าง Configuration สำหรับหลักสูตร

```python
from สำเนาของxxx_สำเนาของ_ele_muti_search_retrival import (
    BaselineCourseConfig, 
    BaselineRAG,
    CourseConfig,
    MultiDocumentRAG
)

# การตั้งค่าสำหรับ Baseline RAG
cs_config = BaselineCourseConfig(
    name="Computer Science",
    file_path="data/cs_course.txt",
    elective_file_path="data/el.txt",
    offset=0  # จำนวนตัวอักษรที่จะข้ามจากต้นไฟล์
)

# การตั้งค่าสำหรับ Multi-document RAG
cs_multi_config = CourseConfig(
    name="Computer Science",
    file_path="data/cs_course.txt",
    offset=0,
    section_titles=[
        '1. รหัสและชื่อหลักสูตร',
        '3. หลักสูตรและอาจารย์ผู้สอน',
        'เกณฑ์การสำเร็จการศึกษาตามหลักสูตร',
        '5. ข้อกำหนดเกี่ยวกับการทำโครงงานหรืองานวิจัย'
    ],
    selected_headers=[
        '1. รหัสและชื่อหลักสูตร',
        '3. หลักสูตรและอาจารย์ผู้สอน',
        'เกณฑ์การสำเร็จการศึกษาตามหลักสูตร'
    ],
    chunking_plan={
        "3. หลักสูตรและอาจารย์ผู้สอน": {
            "method": "subtoc", 
            "metadata": {"program": "CS"}
        },
        "__default__": {"method": "semantic", "metadata": {"fallback": True}}
    }
)
```

### 2. การสร้าง RAG System

```python
# การสร้าง Baseline RAG System
baseline_rag = BaselineRAG(enable_metadata=True)
baseline_rag.add_course(cs_config)

# การสร้าง Multi-document RAG System
multi_rag = MultiDocumentRAG()
multi_rag.add_course(cs_multi_config)
```

### 3. การสร้าง Vector Database

```python
# การสร้าง Vector Store สำหรับ Baseline
baseline_rag.build_vectorstores()
baseline_rag.save_vectorstores("./vectorstores/baseline")

# การสร้าง Vector Store สำหรับ Multi-document
multi_rag.build_vectorstores()
multi_rag.save_vectorstores("./vectorstores/multi")
```

### 4. การโหลด Vector Database ที่มีอยู่แล้ว

```python
# การโหลด Baseline Vector Store
baseline_rag = BaselineRAG()
baseline_rag.load_vectorstores("./vectorstores/baseline")

# การโหลด Multi-document Vector Store
multi_rag = MultiDocumentRAG()
multi_rag.load_vectorstores("./vectorstores/multi")
```

## การค้นหาข้อมูล

### 1. การตั้งค่า Retrieval Components

```python
from สำเนาของxxx_สำเนาของ_ele_muti_search_retrival import (
    BaselineQuery,
    QueryRewriter,
    MultiQueryRewriter,
    DistributedSearcher,
    BaselineRetriever,
    CrossEncoderRetriever,
    MetadataRetriever
)

# การสร้าง Query Rewriter
query_rewriter = QueryRewriter(
    model="deepseek/deepseek-chat-v3-0324",
    temperature=0.1,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1"
)

# การสร้าง Distributed Searcher
distributed_searcher = DistributedSearcher(multi_rag, K=20)

# การสร้าง Retriever ต่างๆ
baseline_retriever = BaselineRetriever(
    rag_system=multi_rag,
    query_rewriter=query_rewriter,
    distributed_searcher=distributed_searcher,
    K=12,
    k=4
)
```

### 2. การค้นหาข้อมูล

```python
# การค้นหาข้อมูลด้วยคำถาม
query = "ใน Computer Science ต้องเก็บหน่วยกิตของหมวดวิชาศึกษาทั่วไปกี่หน่วยกิต"

# การใช้งาน Retriever
results = baseline_retriever.invoke(query)

# การแสดงผลลัพธ์
for i, doc in enumerate(results, 1):
    metadata = doc.metadata
    print(f"เอกสารที่ {i}:")
    print(f"  หลักสูตร: {metadata.get('course_name', 'N/A')}")
    print(f"  ส่วน: {metadata.get('toc_title', 'N/A')}")
    print(f"  เนื้อหา: {doc.page_content[:200]}...")
    print()
```

## การสร้างคำตอบ

### 1. การตั้งค่า LLM สำหรับการตอบ

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# การสร้าง LLM
llm = ChatOpenAI(
    model="qwen/qwen3-vl-30b-a3b-instruct",
    temperature=0.1,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1"
)

# การสร้าง Prompt Template
answer_prompt = ChatPromptTemplate.from_messages([
    ("system",
     """คุณเป็นผู้ช่วยตอบคำถามสำหรับนักศึกษาระดับปริญญาตรี โดยใช้ข้อมูลจาก [Context]

กฎการตอบ:
1. ตอบเป็นภาษาไทยเท่านั้น
2. ใช้ข้อมูลจาก Context เท่านั้น อย่าเพิ่มข้อมูลนอก Context
3. ตอบสั้น เข้าใจง่าย และกระชับ
4. เริ่มตอบด้วยประโยค์ที่มาจากคำถาม
5. ใช้คำว่า "ครับ" เมื่อสิ้นประโยค์
"""),
    ("user",
     """คำถาม:
{question}

[Context]
{context}

ตอบตามกฎข้างบน""")
])

# การสร้าง Chain
qa_chain = answer_prompt | llm | StrOutputParser()
```

### 2. การสร้างคำตอบจากผลการค้นหา

```python
# การสร้าง Context จากผลการค้นหา
context_text = "\n\n".join([
    f"หลักสูตร: {doc.metadata.get('course_name', 'N/A')}\n"
    f"ส่วน: {doc.metadata.get('toc_title', 'N/A')}\n"
    f"เนื้อหา: {doc.page_content}"
    for doc in results
])

# การสร้างคำตอบ
answer = qa_chain.invoke({
    "question": query,
    "context": context_text
})

print("คำตอบ:")
print(answer)
```

## ตัวอย่างการใช้งานแบบสมบูรณ์

### ตัวอย่างที่ 1: การค้นหาข้อมูลหน่วยกิต

```python
# การตั้งค่า
query = "ใน Computer Science ต้องเก็บหน่วยกิตรวมทั้งหมดกี่หน่วยกิต"

# การค้นหา
results = baseline_retriever.invoke(query)

# การสร้างคำตอบ
context_text = "\n\n".join([
    f"หลักสูตร: {doc.metadata.get('course_name')}\n"
    f"เนื้อหา: {doc.page_content}"
    for doc in results
])

answer = qa_chain.invoke({
    "question": query,
    "context": context_text
})

print(f"คำถาม: {query}")
print(f"คำตอบ: {answer}")
```

### ตัวอย่างที่ 2: การค้นหาข้อมูลรายวิชา

```python
# การตั้งค่า
query = "ใน CS วิชา SC 352 002 เรียนเกี่ยวกับอะไร"

# การค้นหา
results = baseline_retriever.invoke(query)

# การสร้างคำตอบ
context_text = "\n\n".join([
    f"หลักสูตร: {doc.metadata.get('course_name')}\n"
    f"รายวิชา: {doc.metadata.get('sub_header', 'N/A')}\n"
    f"เนื้อหา: {doc.page_content}"
    for doc in results
])

answer = qa_chain.invoke({
    "question": query,
    "context": context_text
})

print(f"คำถาม: {query}")
print(f"คำตอบ: {answer}")
```

### ตัวอย่างที่ 3: การค้นหาข้อมูลแผนการศึกษา

```python
# การตั้งค่า
query = "แผนการศึกษาปีที่ 2 ภาคการศึกษาที่ 1 มีวิชาอะไรบ้าง"

# การค้นหา
results = baseline_retriever.invoke(query)

# การสร้างคำตอบ
context_text = "\n\n".join([
    f"หลักสูตร: {doc.metadata.get('course_name')}\n"
    f"แผนการศึกษา: {doc.metadata.get('toc_title', 'N/A')}\n"
    f"เนื้อหา: {doc.page_content}"
    for doc in results
])

answer = qa_chain.invoke({
    "question": query,
    "context": context_text
})

print(f"คำถาม: {query}")
print(f"คำตอบ: {answer}")
```

## การปรับแต่งพารามิเตอร์

### 1. การปรับ Chunk Size

```python
# การปรับขนาด Chunk สำหรับ Baseline RAG
config = BaselineCourseConfig(
    name="Computer Science",
    file_path="data/cs_course.txt",
    elective_file_path="data/el.txt",
    chunk_size=400,  # เพิ่มขนาด Chunk
    chunk_overlap=100  # เพิ่มขนาด Overlap
)
```

### 2. การปรับ Retrieval Parameters

```python
# การปรับจำนวนเอกสารที่ดึง
retriever = BaselineRetriever(
    rag_system=multi_rag,
    query_rewriter=query_rewriter,
    distributed_searcher=distributed_searcher,
    K=20,  # ดึง 20 เอกสารแรก
    k=8    คืนค่า 8 เอกสารสุดท้าย
)
```

### 3. การเปลี่ยน Embedding Model

```python
# การใช้ HuggingFace Embeddings
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-mpnet-base-v2",
    encode_kwargs={"batch_size": 64, "normalize_embeddings": True},
)
```

## การบันทึกและโหลดข้อมูล

### การบันทึก Vector Database

```python
# การบันทึกเป็นไฟล์ ZIP
import shutil

vector_db_dir = "./vectorstores/baseline"
zip_filename = "baseline_vectordb.zip"
shutil.make_archive(zip_filename.replace(".zip", ""), 'zip', vector_db_dir)

print(f"Vector database ถูกบันทึกที่: {zip_filename}")
```

### การโหลด Vector Database

```python
import zipfile

# การคลายไฟล์ ZIP
with zipfile.ZipFile("baseline_vectordb.zip", 'r') as zip_ref:
    zip_ref.extractall("./vectorstores/loaded")

# การโหลดใน RAG System
baseline_rag = BaselineRAG()
baseline_rag.load_vectorstores("./vectorstores/loaded")
```

## ข้อควรพิจารณา

### 1. การเลือก Model ที่เหมาะสม

- **OpenAI Models**: เหมาะสำหรับภาษาอังกฤษ แตะมีค่าใช้จ่ายสูง
- **DeepSeek**: ประสิทธิดีสำหรับภาษาไทย ค่าใช้จ่ายปานกลาง
- **Qwen**: สมดุลสำหรับงานที่ต้องการความแม่นยำ

### 2. การจัดการ Memory

```python
# การจำกัดการใช้งาน Memory
import torch

if torch.cuda.is_available():
    torch.cuda.empty_cache()
    print(f"GPU Memory: {torch.cuda.memory_allocated() / 1024**3:.2f} GB")
```

### 3. การตรวจสอบผลลัพธ์

```python
# การตรวจสอบความถูกต้องของผลลัพธ์
def validate_results(results, query):
    """ตรวจสอบความเกี่ยวข้องของผลการค้นหา"""
    if not results:
        print("⚠️ ไม่พบผลลัพธ์")
        return False
    
    # การตรวจสอบคำสำคัญ
    keywords = query.split()
    for doc in results[:3]:  # ตรวจสอบ 3 เอกสารแรก
        content_lower = doc.page_content.lower()
        keyword_matches = sum(1 for kw in keywords if kw.lower() in content_lower)
        if keyword_matches == 0:
            print(f"⚠️ เอกสารอาจไม่เกี่ยวข้อง: {doc.page_content[:100]}...")
    
    return True

# การใช้งาน
validate_results(results, query)
```

## การประเมินผลระบบ RAG

### บทนำ

การประเมินผลเป็นส่วนสำคัญในการพัฒนาระบบ RAG (Retrieval-Augmented Generation) เพื่อให้มั่นใจว่าระบบสามารถตอบคำถามได้อย่างถูกต้องและเกี่ยวข้องกับข้อมูลที่ดึงมา ในส่วนนี้จะอธิบายวิธีการประเมินผลระบบ RAG โดยใช้เมตริกต่างๆ ได้แก่ Answer Correctness, Faithfulness, Context Precision และ Context Recall

### ภาพรวมของระบบประเมินผล

ระบบประเมินผลประกอบด้วยเมตริกหลัก 4 ประเภท:

1. **Answer Correctness**: วัดความถูกต้องของคำตอบเมื่อเทียบกับคำตอบอ้างอิง
2. **Faithfulness**: วัดความสอดคล้องของคำตอบกับข้อมูลที่ดึงมา
3. **Context Precision**: วัดความเกี่ยวข้องของข้อมูลที่ดึงมาเมื่อเทียบกับคำถาม
4. **Context Recall**: วัดความครบถ้วนของข้อมูลที่ดึงมาเมื่อเทียบกับคำตอบอ้างอิง

### การติดตั้งและการเตรียมข้อมูล

#### 1. การติดตั้ง Library ที่จำเป็น

```bash
pip install "ragas>=0.1.11" "langchain-openai>=0.2.2" "datasets>=2.20.0" "tqdm" "tenacity"
```

#### 2. การตั้งค่า API Keys

ต้องตั้งค่า API keys สำหรับการเชื่อมต่อกับบริการต่างๆ:

```python
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
```

#### 3. การเตรียมข้อมูลสำหรับการประเมินผล

ข้อมูลที่ต้องเตรียมสำหรับการประเมินผลคือไฟล์ CSV ที่มีคอลัมน์ต่อไปนี้:

- `question`: คำถามที่ผู้ใช้ถาม
- `prediction`: คำตอบที่ระบบสร้างขึ้น
- `reference`: คำตอบอ้างอิง (Ground Truth)
- `retrieved_docs`: เอกสารที่ระบบดึงมาเพื่อตอบคำถาม

```python
import pandas as pd

# โหลดข้อมูลจากไฟล์ CSV
df_chatbot_answer = pd.read_csv("qa_results.csv")
df_chatbot_answer = df_chatbot_answer.drop_duplicates(subset=['question'], keep='first')

# แก้ไขข้อมูลอ้างอิง (ถ้าจำเป็น)
df_chatbot_answer.loc[
    df_chatbot_answer['question'] == 'ใน GIS จำนวนหน่วยกิตตลอดหลักสูตรเป็นเท่าไหร่',
    'reference'
] = 'จำนวนหน่วยกิตตลอดหลักสูตรคือ 127 หน่วยกิตครับ'
```

### การประเมินผลด้วย Answer Correctness

Answer Correctness เป็นเมตริกที่วัดความถูกต้องของคำตอบเมื่อเทียบกับคำตอบอ้างอิง โดยใช้ LLM เป็นผู้ตัดสิน (LLM-as-a-Judge)

#### การตั้งค่า LLM สำหรับการประเมินผล

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
import re
import asyncio
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import pandas as pd
from tqdm.asyncio import tqdm_asyncio

# ควบคุมความขนาน (ขึ้นกับ rate limit ของโมเดล)
MAX_CONCURRENCY = 8        # ปรับได้ 4–16 ตามโควต้า/ข้อจำกัด
REQS_PER_SECOND = 3        # ถ้าถูก 429 ให้ลดค่าลง

llm = ChatOpenAI(
    model="x-ai/grok-4-fast",
    temperature=0.0,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1",
)
```

#### การสร้าง Prompt Template สำหรับการประเมินผล

```python
answer_correctness_prompt = ChatPromptTemplate.from_template("""
You are an impartial evaluator (LLM-as-a-Judge) for a university curriculum QA system.
You are given three items:

Question: {question}
Model Answer (Prediction): {prediction}
Reference Answer (Ground Truth): {reference}

Your task is to evaluate how correct and aligned the model answer is compared to the reference.
Consider factual accuracy, completeness, and consistency with the reference answer.

Consider factual accuracy, completeness, and consistency with the reference answer.

If the missing answer part is not important, then ignore it.
If the extra information provided is contextually relevant and does not contradict
the reference, it should not lower the score.

Scoring rubric:
0.0 = Incorrect, irrelevant, or hallucinated
0.25  = Partially correct but missing major information
0.5  = Mostly correct with some inaccuracies or omissions
0.75  = Correct and sufficient but not perfectly detailed
1.0  = Fully correct, comprehensive, and well-aligned

Please respond in **two clear lines only**:
Reason: <short justification in English>
Score: <0.0 - 1.0>
""")
```

#### การสร้างฟังก์ชันสำหรับการประเมินผล

```python
# ตัวช่วยพาร์สอย่างทนทาน
score_re = re.compile(r"Score\s*:\s*([0-9]+(?:\.[0-9]+)?)", re.I)
reason_re = re.compile(r"Reason\s*:\s*(.*)", re.I)

def parse_reason_score(text: str):
    text = (text or "").strip()
    # หา reason
    reason_match = reason_re.search(text)
    reason = reason_match.group(1).strip() if reason_match else ""
    # หา score
    score_match = score_re.search(text.replace(",", "."))
    score_str = score_match.group(1) if score_match else None
    return {"score": score_str, "reason": reason or None, "raw": text}

# ฟังก์ชันเรียกโมเดลแบบ async พร้อม retry/backoff
@retry(
    reraise=True,
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=1, max=20),
    retry=retry_if_exception_type(Exception),
)
async def judge_one(row: pd.Series, sem: asyncio.Semaphore):
    # สร้าง message
    prompt = answer_correctness_prompt.format_messages(
        question=row["question"],
        prediction=row["prediction"],
        reference=row["reference"],
    )
    async with sem:
        # rate limit แบบง่าย: เว้นช่วงตาม REQS_PER_SECOND
        await asyncio.sleep(1.0 / max(1, REQS_PER_SECOND))
        res = await llm.ainvoke(prompt)
    return parse_reason_score(res.content)

# ประเมินแบบขนานด้วย asyncio
async def evaluate_parallel(df: pd.DataFrame) -> pd.DataFrame:
    sem = asyncio.Semaphore(MAX_CONCURRENCY)
    tasks = [judge_one(row, sem) for _, row in df.iterrows()]

    # ใช้ gather ของ tqdm.asyncio → คงลำดับผลลัพธ์ = ลำดับ tasks เดิม
    ordered_results = await tqdm_asyncio.gather(*tasks, total=len(tasks), desc="Judging")

    out = df.copy()
    out["score"] = [r.get("score") for r in ordered_results]
    out["reason"] = [r.get("reason") for r in ordered_results]
    out["llm_judge_raw"] = [r.get("raw") for r in ordered_results]
    return out
```

#### การประเมินผล Answer Correctness

```python
# เลือกข้อมูลที่ต้องการประเมิน
df_ac = df_chatbot_answer[['question','prediction','reference']]

# ประเมินผล
df_ac = await evaluate_parallel(df_ac)

# แสดงผลลัพธ์
df_ac[["question", "score", "reason"]]

# เปลี่ยนชื่อคอลัมน์และบันทึกผลลัพธ์
df_ac_summary = df_ac.copy()
df_ac_summary.rename(columns={'score': 'answer_correctness'}, inplace=True)
df_ac_summary.to_csv('rubric.csv', index=False)
```

### การประเมินผลด้วย Faithfulness

Faithfulness เป็นเมตริกที่วัดความสอดคล้องของคำตอบกับข้อมูลที่ดึงมา (retrieved context)

#### การตั้งค่า LLM และ Embeddings สำหรับ Ragas

```python
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness
from ragas.llms import LangchainLLMWrapper
from ragas.embeddings import LangchainEmbeddingsWrapper
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
import pandas as pd

# ตั้งค่า LLM
llm = ChatOpenAI(
    model="google/gemini-2.5-flash-lite-preview-09-2025",
    temperature=0.0,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1",
)

ragas_llm = LangchainLLMWrapper(llm)

# ตั้งค่า Embeddings
emb_openai = OpenAIEmbeddings(
    model="text-embedding-3-small",
    openai_api_key="your-openai-api-key"
)

# wrap ให้เป็นอินเทอร์เฟซของ ragas
emb = LangchainEmbeddingsWrapper(emb_openai)

# ตรวจสอบขนาดของเวกเตอร์
test_vec = emb_openai.embed_query("hello world")
print("Embedding dim:", len(test_vec))
```

#### การสร้างฟังก์ชันสำหรับการประเมินผล Faithfulness

```python
def evaluate_faithfulness_df(df_in: pd.DataFrame, llm, emb, batch_size: int = 20) -> pd.DataFrame:
    """
    Run faithfulness eval for all records in df_in (batch-friendly).

    Args:
        df_in: DataFrame with ['question','prediction','retrieved_docs']
        llm: ragas_llm (LangchainLLMWrapper)
        emb: ragas embeddings (LangchainEmbeddingsWrapper)
        batch_size: number of rows per batch

    Returns:
        DataFrame with 'faithfulness' score aligned to df_in.index
    """
    n = len(df_in)
    out_chunks = []

    for bi in range(0, n, batch_size):
        chunk = df_in.iloc[bi:bi+batch_size]

        def safe_parse_context(x):
            if isinstance(x, list):
                return x
            if isinstance(x, str):
                x = x.strip()
                # case: looks like JSON list string
                if x.startswith("[") and x.endswith("]"):
                    try:
                        return json.loads(x)
                    except json.JSONDecodeError:
                        pass
                # fallback: wrap entire text in a single-element list
                return [x]
            return [str(x)]

        contexts = chunk["retrieved_docs"].apply(safe_parse_context).tolist()

        ds = Dataset.from_dict({
            "question": chunk["question"].tolist(),
            "answer": chunk["prediction"].tolist(),
            "contexts": contexts,
        })

        sc = evaluate(
            ds,
            metrics=[faithfulness],
            llm=llm,
            embeddings=emb,
        )

        pdf = sc.to_pandas()
        pdf.index = chunk.index
        out_chunks.append(pdf)

    return pd.concat(out_chunks).sort_index()
```

#### การประเมินผล Faithfulness

```python
# เลือกข้อมูลที่ต้องการประเมิน
df_ff = df_chatbot_answer[['question','prediction','retrieved_docs']]

# ประเมินผล
df_scores_faith = evaluate_faithfulness_df(df_ff, ragas_llm, emb)

# แสดงผลลัพธ์
df_scores_faith.sort_values(by='faithfulness', ascending=False)

# จัดการค่าที่เป็น NaN
df_scores_faith.fillna(0, inplace=True)

# เปลี่ยนชื่อคอลัมน์และบันทึกผลลัพธ์
df_ff_summary = df_scores_faith.copy()
df_ff_summary.rename(columns={'user_input':'question'}, inplace=True)
df_ff_summary.to_csv('faithfulness.csv', index=False)
```

### การประเมินผลด้วย Context Precision

Context Precision เป็นเมตริกที่วัดความเกี่ยวข้องของข้อมูลที่ดึงมาเมื่อเทียบกับคำถาม

#### การตั้งค่า LLM สำหรับการประเมินผล

```python
import re
import asyncio
from tqdm import tqdm
from ragas import SingleTurnSample
from ragas.metrics import LLMContextPrecisionWithReference
from langchain_openai import ChatOpenAI

# ตั้งค่าโมเดล
llm_ctx_pre = ChatOpenAI(
    model="google/gemini-2.5-flash-lite-preview-09-2025",
    temperature=0.0,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1",
)

# ตั้งค่า Metric
context_precision_metric = LLMContextPrecisionWithReference(llm=llm_ctx_pre)
```

#### การสร้างฟังก์ชันสำหรับการประเมินผล Context Precision

```python
async def compute_llm_context_precision_parallel(df_ctx_pre, evaluator_llm, max_concurrency: int = 3, show_progress=True):
    metric = LLMContextPrecisionWithReference(llm=evaluator_llm)
    regex_pattern = r"__.*?__\n\s*(.*?)(?=\n\n---Context---|$)"

    samples = []
    for _, row in df_ctx_pre.iterrows():
        text = row["retrieved_docs"][0]

        # ตัด context ให้เป็น list ก่อน
        if isinstance(text, str):
            matches = re.findall(regex_pattern, text, flags=re.DOTALL)
            retrieved_contexts_clean = [m.strip() for m in matches]
        elif isinstance(text, list):
            retrieved_contexts_clean = text
        else:
            retrieved_contexts_clean = []

        samples.append(
            SingleTurnSample(
                user_input=row["question"],
                reference=row["reference"],
                retrieved_contexts=retrieved_contexts_clean,
            )
        )

    sem = asyncio.Semaphore(max_concurrency)
    results = [None] * len(samples)

    async def evaluate_sample(i, sample):
        async with sem:
            try:
                score = await metric.single_turn_ascore(sample)
                return (i, score)
            except Exception as e:
                print(f"⚠️ Error in row {i}: {e}")
                return (i, None)

    tasks = [evaluate_sample(i, s) for i, s in enumerate(samples)]

    if show_progress:
        for fut in tqdm(asyncio.as_completed(tasks), total=len(tasks)):
            i, score = await fut
            results[i] = score
    else:
        results = await asyncio.gather(*tasks)

    df_ctx_pre["ctx_pre_score"] = results
    return df_ctx_pre
```

#### การประเมินผล Context Precision

```python
# เลือกข้อมูลที่ต้องการประเมิน
df_ctx_pre = df_chatbot_answer[['question','reference','retrieved_docs']]
df_ctx_pre["retrieved_docs"] = df_ctx_pre["retrieved_docs"].apply(lambda x: [x])

# ประเมินผล
df_ctx_pre_result = await compute_llm_context_precision_parallel(df_ctx_pre, llm_ctx_pre)

# แสดงผลลัพธ์
df_ctx_pre_result.head()

# บันทึกผลลัพธ์
df_ctx_pre_result.to_csv('ctx_precision.csv', index=False)
```

### การประเมินผลด้วย Context Recall

Context Recall เป็นเมตริกที่วัดความครบถ้วนของข้อมูลที่ดึงมาเมื่อเทียบกับคำตอบอ้างอิง

#### การตั้งค่า LLM สำหรับการประเมินผล

```python
import re
import asyncio
from tqdm import tqdm
from ragas import SingleTurnSample
from ragas.metrics import LLMContextRecall
from langchain_openai import ChatOpenAI

# ตั้งค่าโมเดล
llm_ctx_recall = ChatOpenAI(
    model="x-ai/grok-4-fast",
    temperature=0.0,
    openai_api_key="your-api-key",
    base_url="https://openrouter.ai/api/v1",
)

# ตั้งค่า Metric
context_recall_metric = LLMContextRecall(llm=llm_ctx_recall)
```

#### การสร้างฟังก์ชันสำหรับการประเมินผล Context Recall

```python
async def compute_llm_context_recall_parallel(df_ctx_recall, evaluator_llm, max_concurrency: int = 8, show_progress=True):
    metric = LLMContextRecall(llm=evaluator_llm)
    regex_pattern = r"__.*?__\n\s*(.*?)(?=\n\n---Context---|$)"

    samples = []
    for _, row in df_ctx_recall.iterrows():
        text = row["retrieved_docs"][0]

        # ตัด context ให้เป็น list ก่อน
        if isinstance(text, str):
            matches = re.findall(regex_pattern, text, flags=re.DOTALL)
            retrieved_contexts_clean = [m.strip() for m in matches]
        elif isinstance(text, list):
            retrieved_contexts_clean = text
        else:
            retrieved_contexts_clean = []

        samples.append(
            SingleTurnSample(
                user_input=row["question"],
                response=row['prediction'],
                reference=row["reference"],
                retrieved_contexts=retrieved_contexts_clean,
            )
        )

    sem = asyncio.Semaphore(max_concurrency)
    results = [None] * len(samples)

    async def evaluate_sample(i, sample):
        async with sem:
            try:
                score = await metric.single_turn_ascore(sample)
                return (i, score)
            except Exception as e:
                print(f"⚠️ Error in row {i}: {e}")
                return (i, None)

    tasks = [evaluate_sample(i, s) for i, s in enumerate(samples)]

    if show_progress:
        for fut in tqdm(asyncio.as_completed(tasks), total=len(tasks)):
            i, score = await fut
            results[i] = score
    else:
        results = await asyncio.gather(*tasks)

    df_ctx_recall["ctx_recall_score"] = results
    return df_ctx_recall
```

#### การประเมินผล Context Recall

```python
# เลือกข้อมูลที่ต้องการประเมิน
df_ctx_recall = df_chatbot_answer[['question','prediction','reference','retrieved_docs']]
df_ctx_recall["retrieved_docs"] = df_ctx_recall["retrieved_docs"].apply(lambda x: [x])

# ประเมินผล
df_ctx_recall = await compute_llm_context_recall_parallel(df_ctx_recall, llm_ctx_recall)

# แสดงผลลัพธ์
df_ctx_recall

# บันทึกผลลัพธ์
df_ctx_recall.to_csv('ctx_recall.csv', index=False)
```

### การรวมผลการประเมินและการสรุปข้อมูล

หลังจากประเมินผลด้วยเมตริกทั้งหมดแล้ว สามารถรวมผลการประเมินเพื่อวิเคราะห์โดยรวมได้

#### การรวมผลการประเมินทั้งหมด

```python
import pandas as pd

# โหลดผลการประเมินทั้งหมด
rubric_score = pd.read_csv('rubric.csv')
faithfulness_score = pd.read_csv('faithfulness.csv')
ctx_precision_score = pd.read_csv('ctx_precision.csv')
ctx_recall_score = pd.read_csv('ctx_recall.csv')
ref_ctx = pd.read_csv('reference_ctx.csv')  # ถ้ามีข้อมูล reference context

# รวมข้อมูลทั้งหมดจาก column "question"
all_process_df = (
    rubric_score
    .merge(
        faithfulness_score[['question', 'faithfulness']],
        on='question',
        how='left'
    )
    .merge(
        ctx_precision_score[['question', 'retrieved_docs', 'ctx_pre_score']],
        on='question',
        how='left'
    )
    .merge(
        ctx_recall_score[['question', 'ctx_recall_score']],
        on='question',
        how='left'
    )
    .merge(
        ref_ctx[['question', 'reference_context']],
        on='question',
        how='left'
    )
)

# เลือกเฉพาะคอลัมน์ที่ต้องการแสดง
all_process_df = all_process_df[
    [
        'question',
        'prediction',
        'reference',
        'answer_correctness',
        'reason',
        'faithfulness',
        'retrieved_docs',
        'ctx_pre_score',
        'ctx_recall_score',
        'reference_context'
    ]
]

# แสดงผลลัพธ์
all_process_df.to_csv('all_process.csv', index=False)
all_process_df
```

#### การวิเคราะห์ผลการประเมิน

```python
# คำนวณค่าเฉลี่ยของแต่ละเมตริก
mean_answer_correctness = pd.to_numeric(all_process_df["answer_correctness"], errors="coerce").mean()
mean_faithfulness = pd.to_numeric(all_process_df["faithfulness"], errors="coerce").mean()
mean_ctx_precision = pd.to_numeric(all_process_df["ctx_pre_score"], errors="coerce").mean()
mean_ctx_recall = pd.to_numeric(all_process_df["ctx_recall_score"], errors="coerce").mean()

print(f"📊 Mean Answer Correctness: {mean_answer_correctness:.3f}")
print(f"📊 Mean Faithfulness: {mean_faithfulness:.3f}")
print(f"📊 Mean Context Precision: {mean_ctx_precision:.3f}")
print(f"📊 Mean Context Recall: {mean_ctx_recall:.3f}")

# แสดงกราฟเปรียบเทียบผลการประเมิน
import matplotlib.pyplot as plt

metrics = ['Answer Correctness', 'Faithfulness', 'Context Precision', 'Context Recall']
values = [mean_answer_correctness, mean_faithfulness, mean_ctx_precision, mean_ctx_recall]

plt.figure(figsize=(10, 6))
plt.bar(metrics, values, color=['blue', 'green', 'orange', 'red'])
plt.title('Average Evaluation Metrics')
plt.ylabel('Score')
plt.ylim(0, 1)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### การบันทึกและการแชร์ผลการประเมิน

#### การบันทึกผลการประเมินเป็นไฟล์ ZIP

```python
import shutil
import os

# สร้างโฟลเดอร์สำหรับเก็บผลการประเมิน
result_path = "./evaluation_results"
os.makedirs(result_path, exist_ok=True)

# คัดลอกไฟล์ผลการประเมินทั้งหมดไปยังโฟลเดอร์
shutil.copy('rubric.csv', result_path)
shutil.copy('faithfulness.csv', result_path)
shutil.copy('ctx_precision.csv', result_path)
shutil.copy('ctx_recall.csv', result_path)
shutil.copy('all_process.csv', result_path)

# บีบอัดโฟลเดอร์เป็นไฟล์ ZIP
zip_filename = "evaluation_results.zip"
shutil.make_archive(zip_filename.replace(".zip", ""), 'zip', result_path)

print(f"Evaluation results successfully zipped to: {zip_filename}")
```

### ข้อควรพิจารณาในการประเมินผล

1. **การเลือกโมเดลสำหรับการประเมิน**: ควรเลือกโมเดลที่มีความสามารถในการเข้าใจภาษาไทยเป็นอย่างดี
2. **การจัดการ Rate Limit**: ควรปรับค่า MAX_CONCURRENCY และ REQS_PER_SECOND ให้เหมาะสมกับข้อจำกัดของ API
3. **การจัดการข้อมูลที่เป็น NaN**: ควรตรวจสอบและจัดการค่าที่เป็น NaN ในผลการประเมิน
4. **การตีความผลการประเมิน**: ควรพิจารณาผลการประเมินร่วมกับข้อมูลเชิงคุณภาพเพื่อการปรับปรุงระบบ

## สรุปเทศ

RAG Pipeline นี้มีความสามารถในการค้นหาข้อมูลหลักสูตรด้วยกลไกลดังนี้:

1. **รองรับหลายสูตร**: รองรับข้อมูลได้ทั้งในรูปแบบ Baseline และ Multi-document
2. **การประมวลผลอัตโนมัติ**: รองรับการตัดคำและการแบ่งย่อยหมวดหัวข้อ
3. **การค้นหาขั้นสูง**: รองรับ Query Rewriting และ Multi-query Retrieval
4. **การจัดอันดับบิ**: รองรับ Cross-encoder และ Metadata-based Scoring
5. **ความยืดหยุ่น**: รองรับการทำงานแบบ Batch และ Concurrent Processing

ผู้ใช้งานสามารถปรับแต่งพารามิเตอร์ต่างๆ ให้เหมาะสมกับความต้องการและลักษณะข้อมูลที่มีอยู่