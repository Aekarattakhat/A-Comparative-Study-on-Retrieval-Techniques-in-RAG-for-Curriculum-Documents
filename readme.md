# คู่มือการใช้งาน

## 1\. การเตรียมข้อมูล (Data Preparation)

ขั้นตอนการเตรียมข้อมูลเป็นขั้นตอนแรกที่สำคัญที่สุด โดยระบบนี้ถูกออกแบบมาเพื่อประมวลผลเอกสารหลักสูตร (ในรูปแบบ `.txt`) สองประเภทหลัก และแปลงเอกสารเหล่านั้นให้เป็น Vector Store (ฐานข้อมูลเวกเตอร์) สำหรับการสืบค้น

### 1.1 ประเภทของข้อมูลนำเข้า

ระบบต้องการไฟล์ข้อมูล 2 ส่วน:

1.  **เอกสารหลักสูตร (Curriculum Documents):** ไฟล์ `.txt` ที่มีเนื้อหาของหลักสูตร มคอ. (เช่น `cs_course.txt`, `ai_course.txt`)
2.  **เอกสารวิชาเลือก (Electives Document):** ไฟล์ `.txt` (เช่น `el.txt`) ที่รวบรวมรายการวิชาเลือกทั้งหมดของทุกหลักสูตร โดยมีรูปแบบเฉพาะที่คลาส `ElectiveCatalog` สามารถประมวลผลได้

### 1.2 ตัวอย่างโค้ดการเตรียมข้อมูลและสร้าง Vector Store

ตัวอย่างนี้แสดงการกำหนดค่าและสร้าง Vector Store สำหรับไปป์ไลน์ขั้นสูง (`MultiDocumentRAG`) ซึ่งใช้การแบ่งส่วนเอกสารตามสารบัญ (TOC-based chunking) และการผสานข้อมูลวิชาเลือก

```python
import os
from pathlib import Path
from retrival_code import ( # สมมติว่าคลาสต่างๆ ถูกย้ายไปไฟล์ .py
    CourseConfig, 
    ElectiveCatalog, 
    MultiDocumentRAG,
    setup_example_courses, # ฟังก์ชัน helpers จากโน้ตบุ๊ก
    setup_ai_courses
)

# --- 1. โหลดข้อมูลวิชาเลือก ---
try:
    with open('/content/el.txt', 'r', encoding='utf-8') as f:
        electives_text = f.read()
    catalog = ElectiveCatalog.from_text(electives_text)
    print("โหลด ElectiveCatalog สำเร็จ")
except Exception as e:
    print(f"ไม่สามารถโหลดไฟล์วิชาเลือก: {e}")

# --- 2. กำหนดค่า Helper ---
# โหลดการตั้งค่าเริ่มต้น (TOC, chunking plan) จากฟังก์ชันในโน้ตบุ๊ก
defaults = setup_example_courses()
ai_defaults = setup_ai_courses()

# --- 3. กำหนดค่าแต่ละหลักสูตร (CourseConfig) ---
# ตัวอย่างหลักสูตร Computer Science
cs_course = CourseConfig(
    name="Computer Science",
    file_path="/content/cs_course.txt",
    offset=9800, # ค่า offset เพื่อข้ามส่วนหัวของไฟล์
    section_titles=defaults['section_titles'],
    selected_headers=defaults['selected_headers'],
    subheader_map=defaults['subheader_map'],
    chunking_plan=defaults['chunking_plan'],
    program_name="วิทยาการคอมพิวเตอร์", # ชื่อที่ตรงกับใน el.txt
    electives_catalog=catalog,
    electives_label="วิชาเลือกเพิ่มเติมที่สามารถลงได้"
)

# ตัวอย่างหลักสูตร Artificial Intelligence
ai_course = CourseConfig(
    name="Artificial Intelligence",
    file_path="/content/ai_course.txt",
    offset=2000,
    section_titles=ai_defaults['section_titles'],
    selected_headers=ai_defaults['selected_headers'],
    subheader_map=ai_defaults['subheader_map'],
    chunking_plan=ai_defaults['chunking_plan'],
    program_name="ปัญญาประดิษฐ์",
    electives_catalog=catalog,
    electives_label="วิชาเลือกเพิ่มเติมที่สามารถลงได้"
)

# (กำหนดค่าหลักสูตรอื่นๆ เช่น Cyber security, IT, GIS ในลักษณะเดียวกัน)

# --- 4. สร้างระบบ RAG และประมวลผลข้อมูล ---
rag_builder = MultiDocumentRAG()

# เพิ่มหลักสูตรที่กำหนดค่าไว้
rag_builder.add_course(cs_course)
rag_builder.add_course(ai_course)
# rag_builder.add_course(cy_course)
# rag_builder.add_course(it_course)
# rag_builder.add_course(gis_course)

print("กำลังสร้าง Vector Stores...")
# สร้าง Vector Store (Chunking, Embedding, Indexing)
rag_builder.build_vectorstores()

# --- 5. บันทึก Vector Store ---
output_dir = "./rag_vectordb"
rag_builder.save_vectorstores(output_dir)
print(f"บันทึก Vector Stores ไปยัง {output_dir} สำเร็จ")

# (ในโน้ตบุ๊กมีการสร้าง BaselineRAG ด้วยขั้นตอนที่คล้ายกัน
# โดยใช้ BaselineCourseConfig และ BaselineRAG)
```

-----

## 2\. การใช้งานไปป์ไลน์ (Pipeline Usage)

หลังจาก Vector Store ถูกสร้างและบันทึกไว้ ผู้ใช้สามารถโหลดมาใช้งานเพื่อสืบค้นและสร้างคำตอบได้

### 2.1 การตั้งค่า Components

ก่อนการสืบค้น จำเป็นต้องโหลด Vector Store ที่บันทึกไว้ และตั้งค่า Components ที่จำเป็นสำหรับไปป์ไลน์ (เช่น Query Rewriter, Reranker)

```python
from langchain_openai import ChatOpenAI
from retrival_code import (
    MultiDocumentRAG,
    BaselineRAG,
    DistributedSearcher,
    QueryRewriter,
    MultiQueryRewriter,
    MetadataScore,
    CrossEncoderScore,
    ScoreFusion,
    MetadataCrossEncoderRetriever, # ไปป์ไลน์ขั้นสูง (Hypothesis)
    MultiMetadataCrossEncoderRetriever, # ไปป์ไลน์ขั้นสูงแบบ Multi-Query
    BaselineRetriever, # ไปป์ไลน์พื้นฐาน
    # ... และ retrievers อื่นๆ
)

# --- 1. กำหนดค่า API Keys (ต้องตั้งค่า) ---
OPEN_ROUTER_KEY = "YOUR_OPEN_ROUTER_KEY"
OPEN_API_KEY = "YOUR_OPENAI_API_KEY"

# --- 2. โหลด Vector Stores ---
# โหลด RAG ขั้นสูง (TOC-based)
rag_advanced = MultiDocumentRAG()
rag_advanced.load_vectorstores("./rag_vectordb")
print("โหลด Advanced RAG Vector Stores สำเร็จ")

# โหลด RAG พื้นฐาน (Baseline)
rag_baseline = BaselineRAG()
rag_baseline.load_vectorstores("./baseline_vectordb")
print("โหลด Baseline RAG Vector Stores สำเร็จ")

# --- 3. ตั้งค่า Components ที่ใช้ร่วมกัน ---

# LLM สำหรับ Query Rewriting (ใช้ OpenRouter)
llm_rewriter = ChatOpenAI(
    model="qwen/qwen3-next-80b-a3b-instruct",
    temperature=0.0,
    openai_api_key=OPEN_ROUTER_KEY,
    base_url="https://openrouter.ai/api/v1"
)

# 3.1) Query Rewriters
# แบบ Single-Query
query_rewriter = QueryRewriter(llm=llm_rewriter)
# แบบ Multi-Query
multi_query_rewriter = MultiQueryRewriter(llm=llm_rewriter, max_atomic_queries=3)

# 3.2) Searcher (ตัวค้นหาข้าม Vector Stores)
searcher_advanced = DistributedSearcher(rag_advanced, K=20)
searcher_baseline = DistributedSearcher(rag_baseline, K=12)

# 3.3) Reranking Components
metadata_score = MetadataScore(
    first_line_boost=0.2,
    keywords_boost=0.4,
    metadata_total_cap=0.70
)

cross_encoder_score = CrossEncoderScore(
    model_name="Pongsasit/mod-th-cross-encoder-minilm",
    max_doc_chars=2800
)

score_fusion = ScoreFusion(
    ce_weight=0.50,
    meta_weight=0.50,
    vec_weight=0.4,
    dist_weight=0.05
)

# 3.4) LLM สำหรับสร้างคำตอบ
llm_qa = ChatOpenAI(
    model="qwen/qwen3-vl-30b-a3b-instruct", # หรือโมเดลอื่นตามที่ตั้งค่า
    temperature=0.1,
    openai_api_key=OPEN_ROUTER_KEY,
    base_url="https://openrouter.ai/api/v1",
)

# (สร้าง answer_prompt และ qa_chain ตามโค้ดในโน้ตบุ๊ก)
# qa_chain = answer_prompt | llm_qa | StrOutputParser()
```

### 2.2 การสืบค้นและสร้างคำตอบ (End-to-End Example)

ในงานวิจัยนี้มีการสร้าง Retrievers หลายรูปแบบเพื่อการทดสอบ นี่คือตัวอย่างการใช้งานไปป์ไลน์ 2 รูปแบบ: (1) Baseline และ (2) ไปป์ไลน์ขั้นสูง (Hypothesis)

```python
# --- 1. เลือก Retriever ที่ต้องการทดสอบ ---

# ตัวอย่าง 1: Baseline (Chunking พื้นฐาน, Query พื้นฐาน, Rerank ไม่มี)
retriever_baseline = BaselineRetriever(
    rag_system=rag_baseline,
    query_rewriter=BaselineQuery(), # ไม่ใช้ LLM rewrite
    distributed_searcher=searcher_baseline,
    k=4,
    use_query_rewrite=True
)

# ตัวอย่าง 2: ไปป์ไลน์ขั้นสูง (TOC Chunking, Multi-Query, Metadata + Cross-Encoder Rerank)
# (นี่คือไปป์ไลน์ 'hy_toc_multi_metadatacrossen' ในโน้ตบุ๊ก)

# 2.1 สร้าง Retriever สำหรับ Single-Query ก่อน
single_query_advanced_retriever = MetadataCrossEncoderRetriever(
    rag_system=rag_advanced,
    distributed_searcher=searcher_advanced,
    metadata_score=metadata_score,
    cross_encoder_score=cross_encoder_score,
    score_fusion=score_fusion,
    query_rewriter=query_rewriter, # ใช้ rewriter แบบ single-query
    K=12,
    k=4,
    use_query_rewrite=True
)

# 2.2 หุ้มด้วย Multi-Query
retriever_advanced_multi_query = MultiMetadataCrossEncoderRetriever(
    retriever=single_query_advanced_retriever,
    rag_system=rag_advanced,
    query_rewriter=multi_query_rewriter, # ใช้ rewriter แบบ multi-query
    max_queries=3,
    K=12, # K สำหรับแต่ละ atomic query
    k=2,  # k สำหรับแต่ละ atomic query
    use_query_rewrite=True
)

# --- 2. ตั้งค่าคำถามและเลือกระบบที่จะใช้ ---
query = "ในหลักสูตร AI มีวิชาเลือกอะไรบ้างที่น่าสนใจ"
# เลือก retriever ที่จะใช้จริง
selected_retriever = retriever_advanced_multi_query 

# --- 3. ดึงเอกสาร (Retrieval) ---
print(f"กำลังสืบค้นด้วย: {type(selected_retriever).__name__}")
retrieved_docs = selected_retriever.invoke(query)

# --- 4. สร้าง Context สำหรับ LLM ---
context_text = "\n\n".join([
    f"---Context---\n"
    f"หลักสูตร: {d.metadata.get('course_name', 'N/A')}\n"
    f"หัวข้อ: {d.metadata.get('toc_title', 'N/A')}\n"
    f"เนื้อหา: {d.page_content}"
    for d in retrieved_docs
])

print(f"--- Context ที่ได้ ---\n{context_text}\n--------------------")

# --- 5. สร้างคำตอบ (Generation) ---
# (ต้องมี qa_chain ที่กำหนดไว้จากข้อ 2.1)
# final_answer = qa_chain.invoke({
#     "question": query,
#     "context": context_text
# })

# print(f"--- คำตอบ --- \n{final_answer}")
```

-----

## 3\. การประเมินผล (Evaluation)

ไฟล์ `eva_retrival.ipynb` มีกระบวนการสำหรับการประเมินผลไปป์ไลน์อย่างเป็นระบบโดยใช้ชุดข้อมูลทดสอบ (Test Set) และการวัดผลหลายมิติ

### 3.1 ข้อมูลที่ต้องเตรียม

1.  **ไฟล์ข้อมูลอ้างอิง (Reference Data):**

      * ไฟล์ `reference_answer.csv`
      * ต้องมีคอลัมน์: `question` (คำถาม) และ `reference` (คำตอบที่ถูกต้อง หรือ Ground Truth)

2.  **ไฟล์ผลลัพธ์จากไปป์ไลน์ (Pipeline Output):**

      * ไฟล์ CSV ที่ได้จากการรันไปป์ไลน์ (เช่น `qa_results_s2nd_toc_rewriter_baseline.csv`)
      * ต้องมีคอลัมน์: `question`, `prediction` (คำตอบจากโมเดล), และ `retrieved_docs` (Context ที่ใช้ตอบ)

### 3.2 การรันประเมินผล

โน้ตบุ๊ก `eva_retrival.ipynb` จะทำการโหลดไฟล์ผลลัพธ์จากไปป์ไลน์ และประมวลผลด้วยตัวชี้วัด 4 ด้านหลัก:

1.  **Answer Correctness (LLM-as-a-Judge):**

      * **เป้าหมาย:** ประเมินความถูกต้องของ `prediction` (คำตอบ) เทียบกับ `reference` (คำตอบอ้างอิง)
      * **วิธีการ:** ใช้ LLM (เช่น Grok-4) เป็นผู้ตัดสิน (Judge) ให้คะแนนตามเกณฑ์ (Rubric) 0.0 ถึง 1.0
      * **ผลลัพธ์:** บันทึกใน `rubric.csv`

2.  **Faithfulness (Ragas):**

      * **เป้าหมาย:** ประเมินว่า `prediction` (คำตอบ) สอดคล้องและอ้างอิงจาก `retrieved_docs` (Context) ที่ดึงมาได้หรือไม่ (ลดการ Hallucination)
      * **วิธีการ:** ใช้ `ragas.metrics.faithfulness`
      * **ผลลัพธ์:** บันทึกใน `faithfulness.csv`

3.  **Context Precision (Ragas - LLM-based):**

      * **เป้าหมาย:** ประเมินว่า `retrieved_docs` (Context) ที่ดึงมา มีความเกี่ยวข้องกับ `question` และ `reference` มากน้อยเพียงใด (มีเอกสารที่ไม่จำเป็นปนมาหรือไม่)
      * **วิธีการ:** ใช้ `ragas.metrics.LLMContextPrecisionWithReference`
      * **ผลลัพธ์:** บันทึกใน `ctx_precision.csv`

4.  **Context Recall (Ragas - LLM-based):**

      * **เป้าหมาย:** ประเมินว่า `retrieved_docs` (Context) ที่ดึงมา มีข้อมูลที่จำเป็นสำหรับการตอบ `reference` ครบถ้วนหรือไม่
      * **วิธีการ:** ใช้ `ragas.metrics.LLMContextRecall`
      * **ผลลัพธ์:** บันทึกใน `ctx_recall.csv`

### 3.3 การรวบรวมผลลัพธ์

สุดท้าย สคริปต์ในโน้ตบุ๊กจะรวบรวมไฟล์ CSV ทั้งหมด (รวมถึง `reference_context.csv`) โดยใช้ `question` เป็น Key เพื่อรวมผลการประเมินทั้งหมดไว้ในไฟล์เดียว (`all_process.csv`) สำหรับการวิเคราะห์ในลำดับต่อไป
