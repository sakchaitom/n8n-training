# ตัวอย่าง Script และ Prompt สำหรับ n8n Workshop

เอกสารนี้รวบรวมตัวอย่าง code และ prompt ที่ใช้ในการอบรม n8n และ AI Agent

---

## 📑 สารบัญ

1. [Function Node Scripts](#1-function-node-scripts)
   - [Parse JSON Response จาก AI](#11-parse-json-response-จาก-ai)
   - [เปรียบเทียบหมวดหมู่](#12-เปรียบเทียบหมวดหมู่)
2. [AI Prompts](#2-ai-prompts)
   - [Prompt การจัดหมวดหมู่งานวิจัย](#21-prompt-การจัดหมวดหมู่งานวิจัย)
   - [Prompt การวิเคราะห์และสกัดข้อมูลบทความวิจัย](#22-prompt-การวิเคราะห์และสกัดข้อมูลบทความวิจัย)

---

## 1. Function Node Scripts

### 1.1 Parse JSON Response จาก AI

Function Node นี้ใช้สำหรับประมวลผลข้อมูลที่ได้รับจาก AI Agent โดยจะ:
- ดึงข้อความจาก AI response
- ลบ markdown code block (```json...```) ออก
- Parse JSON และจัดรูปแบบข้อมูล
- จัดการ error กรณี parse ไม่สำเร็จ

#### Code

```javascript

  // ดึงข้อความที่เป็นผลลัพธ์จาก content[0].text
  const text = $input.first().json.output;

  // ลบครอบ ```json ... ``` ออก
  const cleaned = text
    .replace(/```json/i, '')  // ลบ ```json
    .replace(/```/g, '')      // ลบ ``` ที่เหลือ
    .trim();

  let parsed = {};
  try {
    parsed = JSON.parse(cleaned);
  } catch (error) {
    // ถ้า parse ไม่ได้ แปะ raw ไว้ให้ debug
    parsed = {
      category: null,
      confidence: null,
      error: 'JSON parse failed',
      rawText: text
    };
  }

  // แปลง confidence เป็นตัวเลข (ถ้าต้องการ)
  const confidenceNumber = parsed.confidence != null
    ? Number(parsed.confidence)
    : null;

  return {
    json: {
      category: parsed.category || null,
      confidence: confidenceNumber,
      // เก็บของเดิมไว้ด้วย เผื่อใช้ต่อ
      original: text
    }
  };

```

#### คำอธิบาย

1. **ดึงข้อมูลจาก AI Response**
   ```javascript
   const text = ai.content?.[0]?.text || '';
   ```
   - ใช้ optional chaining (`?.`) เพื่อป้องกัน error กรณีไม่มีข้อมูล

2. **ทำความสะอาดข้อความ**
   ```javascript
   const cleaned = text
     .replace(/```json/i, '')  // ลบ ```json (case-insensitive)
     .replace(/```/g, '')      // ลบ ``` ทั้งหมด
     .trim();                  // ลบช่องว่างหัว-ท้าย
   ```

3. **Parse JSON พร้อม Error Handling**
   ```javascript
   try {
     parsed = JSON.parse(cleaned);
   } catch (error) {
     // กรณี parse ไม่สำเร็จ
     parsed = {
       category: null,
       confidence: null,
       error: 'JSON parse failed',
       rawText: text
     };
   }
   ```

4. **แปลงข้อมูลและ Return**
   - แปลง confidence เป็นตัวเลข
   - เก็บข้อมูลต้นฉบับไว้สำหรับ debug

#### ตัวอย่าง Input/Output

**Input (AI Response):**
```json
{
  "output": [{
    "content": [{
      "text": "```json\n{\n  \"category\": \"วิทยาศาสตร์ธรรมชาติ\",\n  \"confidence\": \"95\"\n}\n```"
    }]
  }]
}
```

**Output:**
```json
{
  "category": "วิทยาศาสตร์ธรรมชาติ",
  "confidence": 95,
  "original": { /* ข้อมูล AI เดิม */ }
}
```

---

### 1.2 เปรียบเทียบหมวดหมู่

Function Node นี้ใช้สำหรับเปรียบเทียบผลลัพธ์จาก AI 2 ตัว เพื่อตรวจสอบความสอดคล้องของการจัดหมวดหมู่

#### Code

```javascript
const item1 = $input.all()[0].json.category;
const item2 = $input.all()[1].json.category;

const isSame = item1 === item2;

return [{
  json: {
    category1: item1,
    category2: item2,
    isSame: isSame,
    message: isSame
      ? "✅ หมวดหมู่ตรงกัน"
      : "❌ หมวดหมู่ไม่ตรงกัน"
  }
}];
```

#### คำอธิบาย

1. **รับข้อมูลจาก Input 2 รายการ**
   ```javascript
   const item1 = $input.all()[0].json.category;
   const item2 = $input.all()[1].json.category;
   ```
   - `$input.all()` ดึงข้อมูลจากทุก input ที่เชื่อมต่อเข้ามา
   - `[0]` และ `[1]` คือ input แรกและที่สอง

2. **เปรียบเทียบค่า**
   ```javascript
   const isSame = item1 === item2;
   ```
   - ใช้ `===` เพื่อเปรียบเทียบค่าแบบเข้มงวด (strict equality)

3. **สร้าง Output พร้อม Message**
   - แสดงผลทั้งสองหมวดหมู่
   - แสดงสถานะ `isSame`
   - แสดงข้อความที่เป็นมิตรกับผู้ใช้

#### ตัวอย่าง Input/Output

**Input:**
- Input 1: `{ "category": "วิทยาศาสตร์ธรรมชาติ" }`
- Input 2: `{ "category": "วิทยาศาสตร์ธรรมชาติ" }`

**Output (กรณีตรงกัน):**
```json
{
  "category1": "วิทยาศาสตร์ธรรมชาติ",
  "category2": "วิทยาศาสตร์ธรรมชาติ",
  "isSame": true,
  "message": "✅ หมวดหมู่ตรงกัน"
}
```

**Output (กรณีไม่ตรงกัน):**
```json
{
  "category1": "วิทยาศาสตร์ธรรมชาติ",
  "category2": "วิศวกรรมและเทคโนโลยี",
  "isSame": false,
  "message": "❌ หมวดหมู่ไม่ตรงกัน"
}
```

#### Use Case

ใช้สำหรับ:
- ตรวจสอบความสอดคล้องของ AI หลายตัว
- Validation ผลลัพธ์การจัดหมวดหมู่
- A/B Testing ระหว่าง AI models ต่างๆ

---

## 2. AI Prompts

### 2.1 Prompt การจัดหมวดหมู่งานวิจัย

Prompt นี้ใช้สำหรับการจัดหมวดหมู่งานวิจัยตามมาตรฐาน **OECD Fields of Science and Technology**

#### Prompt Template

```
You are a research classification expert based on OECD Fields of Science.

Classify the following text into ONE of these categories:

1. วิทยาศาสตร์ธรรมชาติ
2. วิศวกรรมและเทคโนโลยี
3. วิทยาศาสตร์การแพทย์และสุขภาพ
4. วิทยาศาสตร์การเกษตรและสัตวแพทย์
5. สังคมศาสตร์
6. มนุษยศาสตร์และศิลปะ

ตอบเป็นรูปแบบ JSON เท่านั้น:

{
  "category": "<ชื่อสาขา>",
  "confidence": "<0-100>"
}

Text:
{{ $json.chatInput }}
```

#### คำอธิบาย

1. **System Role**
   ```
   You are a research classification expert based on OECD Fields of Science.
   ```
   - กำหนดบทบาทของ AI ให้เป็นผู้เชี่ยวชาญด้านการจัดหมวดหมู่งานวิจัย
   - อ้างอิงมาตรฐาน OECD เพื่อความน่าเชื่อถือ

2. **หมวดหมู่ (OECD Fields)**
   - 6 หมวดหมู่หลักตามมาตรฐานสากล
   - ครอบคลุมทุกสาขาวิชา

3. **รูปแบบ Output**
   ```json
   {
     "category": "<ชื่อสาขา>",
     "confidence": "<0-100>"
   }
   ```
   - กำหนดให้ตอบเป็น JSON เพื่อง่ายต่อการ parse
   - รวมค่า confidence เพื่อประเมินความมั่นใจของ AI

4. **Input Variable**
   ```
   {{ $json.chatInput }}
   ```
   - ใช้ n8n expression เพื่อดึงข้อมูลจาก workflow

#### ตัวอย่างการใช้งาน

**Input Text:**
```
การพัฒนาระบบปัญญาประดิษฐ์สำหรับการวิเคราะห์ข้อมูลทางการแพทย์
```

**Expected Output:**
```json
{
  "category": "วิทยาศาสตร์การแพทย์และสุขภาพ",
  "confidence": "85"
}
```

---

**Input Text:**
```
การศึกษาพฤติกรรมผู้บริโภคในยุคดิจิทัล
```

**Expected Output:**
```json
{
  "category": "สังคมศาสตร์",
  "confidence": "90"
}
```

#### การปรับแต่ง Prompt

**เพิ่มรายละเอียดหมวดหมู่:**
```
1. วิทยาศาสตร์ธรรมชาติ (Natural Sciences)
   - คณิตศาสตร์, ฟิสิกส์, เคมี, ชีววิทยา, วิทยาศาสตร์โลก

2. วิศวกรรมและเทคโนโลยี (Engineering and Technology)
   - วิศวกรรมเครื่องกล, คอมพิวเตอร์, ไฟฟ้า, โยธา, เทคโนโลยีสารสนเทศ
...
```

**เพิ่ม Examples (Few-shot Learning):**
```
Examples:
- "การพัฒนาแอพพลิเคชั่นมือถือ" → วิศวกรรมและเทคโนโลยี
- "การศึกษาพฤติกรรมสัตว์" → วิทยาศาสตร์ธรรมชาติ
- "การวิเคราะห์นโยบายสาธารณะ" → สังคมศาสตร์

Now classify this text:
{{ $json.chatInput }}
```

**เพิ่มภาษาอังกฤษ (สำหรับ AI ที่รองรับดีกว่า):**
```
Classify into ONE category:

1. Natural Sciences (วิทยาศาสตร์ธรรมชาติ)
2. Engineering and Technology (วิศวกรรมและเทคโนโลยี)
3. Medical and Health Sciences (วิทยาศาสตร์การแพทย์และสุขภาพ)
...
```

---

### 2.2 Prompt การวิเคราะห์และสกัดข้อมูลบทความวิจัย

Prompt นี้ใช้สำหรับการวิเคราะห์บทความวิจัยและสกัดข้อมูลสำคัญทางบรรณานุกรมและวิธีการวิจัยออกมาเป็น JSON

#### Prompt Template

```
You are an expert in research article analysis and academic information extraction.

Your task:
Read the following research article and extract key bibliographic and research methodology information.

Output the result in JSON format with the following fields:

{
  "title": "",
  "authors": [],
  "publication_year": "",
  "journal_or_source": "",
  "abstract": "",
  "objective": "",
  "research_questions": "",
  "methodology": "",
  "sample_population": "",
  "data_collection": "",
  "data_analysis": "",
  "results": "",
  "conclusion": "",
  "keywords": [],
  "doi": "",
  "other_important_info": ""
}

Rules:
- If any field cannot be found, write "N/A"
- Keep JSON clean and valid
- Summarize using concise academic language
- Do NOT add information that is not in the article
- Do NOT guess
- Preserve original meaning
- Authors must be in list format
- publication_year must be 4 digits
- keywords must be extracted from the article

Now analyze the following article text:

---
{{ $json.text }}
---
```

#### คำอธิบาย

1. **System Role**
   ```
   You are an expert in research article analysis and academic information extraction.
   ```
   - กำหนดให้ AI เป็นผู้เชี่ยวชาญด้านการวิเคราะห์บทความวิจัยและการสกัดข้อมูลทางวิชาการ

2. **โครงสร้าง JSON Output**

   **ข้อมูลบรรณานุกรม (Bibliographic Information):**
   - `title`: ชื่อบทความ
   - `authors`: รายชื่อผู้เขียน (Array)
   - `publication_year`: ปีที่ตีพิมพ์ (4 หลัก)
   - `journal_or_source`: ชื่อวารสารหรือแหล่งที่มา
   - `doi`: Digital Object Identifier
   - `keywords`: คำสำคัญจากบทความ (Array)

   **ข้อมูลเนื้อหาและวิธีการวิจัย (Content & Methodology):**
   - `abstract`: บทคัดย่อ
   - `objective`: วัตถุประสงค์การวิจัย
   - `research_questions`: คำถามวิจัย
   - `methodology`: วิธีการวิจัย
   - `sample_population`: กลุ่มตัวอย่าง/ประชากร
   - `data_collection`: วิธีการเก็บรวบรวมข้อมูล
   - `data_analysis`: วิธีการวิเคราะห์ข้อมูล
   - `results`: ผลการวิจัย
   - `conclusion`: สรุปและข้อเสนอแนะ
   - `other_important_info`: ข้อมูลสำคัญอื่นๆ

3. **กฎการทำงาน (Rules)**

   **หลักความแม่นยำ:**
   - ไม่พบข้อมูล → ใส่ "N/A"
   - ห้ามคาดเดาหรือสมมติข้อมูล
   - ห้ามเพิ่มข้อมูลที่ไม่มีในบทความ
   - รักษาความหมายเดิมของบทความ

   **รูปแบบข้อมูล:**
   - JSON ต้อง valid และ clean
   - ใช้ภาษาวิชาการที่กระชับ
   - Authors และ Keywords เป็น Array
   - publication_year เป็นตัวเลข 4 หลัก

4. **Input Variable**
   ```
   {{ $json.text }}
   ```
   - รับข้อมูลเต็มของบทความวิจัย

#### ตัวอย่างการใช้งาน

**Input (ตัวอย่างบทความย่อ):**
```
การพัฒนาระบบปัญญาประดิษฐ์สำหรับการวิเคราะห์ความเสี่ยงทางการเงิน

โดย สมชาย ใจดี, สมหญิง รักงาน

บทคัดย่อ: งานวิจัยนี้นำเสนอระบบปัญญาประดิษฐ์ที่ใช้ในการวิเคราะห์และประเมินความเสี่ยง
ทางการเงินของสถาบันการเงิน โดยใช้เทคนิค Machine Learning...

วัตถุประสงค์: เพื่อพัฒนาระบบ AI ที่สามารถประเมินความเสี่ยงทางการเงินได้อย่างแม่นยำ

วิธีการ: ใช้ข้อมูลธุรกรรมทางการเงิน 100,000 รายการ วิเคราะห์ด้วย Random Forest

ผลการวิจัย: ระบบมีความแม่นยำ 94.5%

คำสำคัญ: ปัญญาประดิษฐ์, ความเสี่ยงทางการเงิน, Machine Learning
```

**Expected Output:**
```json
{
  "title": "การพัฒนาระบบปัญญาประดิษฐ์สำหรับการวิเคราะห์ความเสี่ยงทางการเงิน",
  "authors": ["สมชาย ใจดี", "สมหญิง รักงาน"],
  "publication_year": "N/A",
  "journal_or_source": "N/A",
  "abstract": "งานวิจัยนี้นำเสนอระบบปัญญาประดิษฐ์ที่ใช้ในการวิเคราะห์และประเมินความเสี่ยงทางการเงินของสถาบันการเงิน โดยใช้เทคนิค Machine Learning",
  "objective": "เพื่อพัฒนาระบบ AI ที่สามารถประเมินความเสี่ยงทางการเงินได้อย่างแม่นยำ",
  "research_questions": "N/A",
  "methodology": "ใช้เทคนิค Machine Learning และ Random Forest",
  "sample_population": "ข้อมูลธุรกรรมทางการเงิน 100,000 รายการ",
  "data_collection": "N/A",
  "data_analysis": "Random Forest",
  "results": "ระบบมีความแม่นยำ 94.5%",
  "conclusion": "N/A",
  "keywords": ["ปัญญาประดิษฐ์", "ความเสี่ยงทางการเงิน", "Machine Learning"],
  "doi": "N/A",
  "other_important_info": "N/A"
}
```

#### การปรับแต่ง Prompt

**เพิ่มฟิลด์เฉพาะทาง:**
```json
{
  // เพิ่มฟิลด์สำหรับงานวิจัยเฉพาะด้าน
  "funding_source": "",
  "ethics_approval": "",
  "limitations": "",
  "future_research": ""
}
```

**กำหนดรูปแบบการสรุป:**
```
Summarize rules:
- Abstract: maximum 200 words
- Methodology: maximum 150 words
- Results: focus on key findings only
```

**รองรับหลายภาษา:**
```
Rules:
- If the article is in Thai, output in Thai
- If the article is in English, output in English
- Preserve original language of keywords and title
```

**เพิ่ม Validation:**
```
Before returning the JSON, verify:
- All required fields are present
- Arrays are properly formatted
- Year is 4 digits or "N/A"
- JSON is valid (no trailing commas, proper quotes)
```

#### Use Cases

1. **สร้างฐานข้อมูลวรรณกรรม (Literature Database)**
   - สกัดข้อมูลจากบทความจำนวนมาก
   - สร้าง Metadata สำหรับ Repository

2. **การทำ Systematic Review**
   - วิเคราะห์และจัดหมวดหมู่บทความวิจัย
   - สกัด Methodology และ Results เพื่อเปรียบเทียบ

3. **การจัดทำรายงานสรุปงานวิจัย**
   - สกัดข้อมูลสำคัญอัตโนมัติ
   - สร้างสรุปข้อมูลในรูปแบบมาตรฐาน

4. **Quality Check**
   - ตรวจสอบความครบถ้วนของข้อมูลในบทความ
   - หา Missing Information

#### การประมวลผล Output

**Function Node สำหรับจัดการผลลัพธ์:**

```javascript
// Parse และ validate output
return $input.all().map(item => {
  const ai = item.json.output[0];
  const text = ai.content?.[0]?.text || '';

  // Clean และ parse JSON
  const cleaned = text
    .replace(/```json/gi, '')
    .replace(/```/g, '')
    .trim();

  let data = {};
  try {
    data = JSON.parse(cleaned);

    // Validate required fields
    const requiredFields = ['title', 'authors', 'abstract'];
    const missingFields = requiredFields.filter(field =>
      !data[field] || data[field] === 'N/A'
    );

    return {
      json: {
        ...data,
        _validation: {
          isValid: missingFields.length === 0,
          missingFields: missingFields,
          completeness: calculateCompleteness(data)
        }
      }
    };
  } catch (error) {
    return {
      json: {
        error: 'Parse failed',
        rawText: text
      }
    };
  }
});

// คำนวณความครบถ้วนของข้อมูล
function calculateCompleteness(data) {
  const allFields = Object.keys(data);
  const filledFields = allFields.filter(key =>
    data[key] && data[key] !== 'N/A' &&
    (Array.isArray(data[key]) ? data[key].length > 0 : true)
  );
  return Math.round((filledFields.length / allFields.length) * 100);
}
```

---

## 📚 เอกสารอ้างอิง

### OECD Fields of Science and Technology
- **เอกสารอย่างเป็นทางการ:** [OECD Frascati Manual](https://www.oecd.org/innovation/frascati-manual-2015-9789264239012-en.htm)
- **Wikipedia:** [OECD Fields of Science](https://en.wikipedia.org/wiki/OECD_classification_of_research_and_development)

### n8n Documentation
- **Function Node:** [https://docs.n8n.io/code/builtin/function/](https://docs.n8n.io/code/builtin/function/)
- **Expressions:** [https://docs.n8n.io/code/expressions/](https://docs.n8n.io/code/expressions/)
- **AI Node:** [https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.ai/](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.ai/)

---

## 💡 Tips & Best Practices

### Function Node

1. **ใช้ Optional Chaining (`?.`)**
   ```javascript
   const text = ai.content?.[0]?.text || '';
   ```
   - ป้องกัน error กรณีข้อมูลไม่ครบ

2. **Error Handling ที่ดี**
   ```javascript
   try {
     // ทำงานหลัก
   } catch (error) {
     // จัดการ error และ return ข้อมูลที่เป็นประโยชน์
     return { error: error.message, rawData: data };
   }
   ```

3. **เก็บข้อมูลต้นฉบับไว้**
   ```javascript
   return {
     json: {
       processed: processedData,
       original: originalData  // เก็บไว้ debug
     }
   };
   ```

### AI Prompts

1. **ระบุบทบาท (Role) ของ AI ให้ชัดเจน**
   - "You are an expert..."
   - ช่วยให้ AI เข้าใจบริบทและตอบได้ตรงจุด

2. **กำหนดรูปแบบ Output ที่ชัดเจน**
   - ระบุว่าต้องการ JSON, CSV, หรือรูปแบบอื่น
   - ให้ตัวอย่างโครงสร้างข้อมูล

3. **ใช้ Few-shot Learning**
   - ให้ตัวอย่างการจัดหมวดหมู่ 2-3 ตัวอย่าง
   - ช่วยเพิ่มความแม่นยำ

4. **ทดสอบหลาย Test Cases**
   - ทดสอบกับข้อมูลหลากหลายประเภท
   - ตรวจสอบ edge cases

---

## 🔧 Troubleshooting

### ปัญหาที่พบบ่อย

#### 1. JSON Parse Error

**อาการ:** `JSON parse failed`

**สาเหตุ:**
- AI ตอบมาพร้อม markdown (```json...```)
- Format ไม่ถูกต้อง
- มีข้อความอื่นปนอยู่

**วิธีแก้:**
```javascript
// เพิ่มการทำความสะอาดเพิ่มเติม
const cleaned = text
  .replace(/```json/gi, '')
  .replace(/```/g, '')
  .replace(/^[^{]*/, '')  // ลบข้อความก่อน {
  .replace(/[^}]*$/, '')  // ลบข้อความหลัง }
  .trim();
```

#### 2. Confidence เป็น String

**อาการ:** `confidence: "95"` แทนที่จะเป็น `95`

**วิธีแก้:**
```javascript
const confidenceNumber = parsed.confidence != null
  ? Number(parsed.confidence)
  : null;
```

#### 3. AI ไม่ตอบตาม Format

**วิธีแก้:**
- เน้นย้ำใน Prompt: "ตอบเป็น JSON เท่านั้น ห้ามแปะข้อความอื่น"
- ลองเปลี่ยน AI model
- ใช้ temperature ต่ำกว่า (0.1-0.3) เพื่อให้ AI มีความ deterministic มากขึ้น

---

**อัปเดตล่าสุด:** พฤศจิกายน 2568
