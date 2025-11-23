# คู่มือการสร้าง API Key และ OAuth 2.0 Client IDs

คู่มือนี้จะแนะนำขั้นตอนการสร้าง API Key และ OAuth 2.0 Client IDs สำหรับการใช้งานกับระบบ n8n และ AI Agent

---

## 📑 สารบัญ

1. [Google Services](#1-google-services)
   - [การสร้าง API Key ของ Gemini](#11-การสร้าง-api-key-ของ-gemini)
   - [การสร้าง OAuth 2.0 Client IDs](#12-การสร้าง-oauth-20-client-ids)
2. [OpenAI](#2-openai)
3. [OpenRouter](#3-openrouter)

---

## 1. Google Services

### 1.1 การสร้าง API Key ของ Gemini

Gemini เป็น AI model จาก Google ที่สามารถใช้งานผ่าน API ได้

#### ขั้นตอนการสร้าง API Key

1. **เข้าสู่ Google AI Studio**
   - ไปที่ [https://makersuite.google.com/app/apikey](https://makersuite.google.com/app/apikey)
   - หรือ [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)

2. **เข้าสู่ระบบด้วย Google Account**
   - ใช้บัญชี Google ของคุณในการเข้าสู่ระบบ

3. **สร้าง API Key**
   - คลิกปุ่ม **"Create API Key"** หรือ **"Get API Key"**
   - เลือก Google Cloud Project ที่ต้องการใช้งาน หรือสร้าง Project ใหม่
   - ระบบจะสร้าง API Key ให้อัตโนมัติ

4. **คัดลอก API Key**
   - คลิก **"Copy"** เพื่อคัดลอก API Key
   - **⚠️ สำคัญ:** เก็บ API Key ไว้ในที่ปลอดภัย อย่าแชร์กับผู้อื่น

5. **ทดสอบ API Key (ไม่บังคับ)**
   - สามารถทดสอบการใช้งานได้ที่ [Google AI Studio](https://aistudio.google.com/)

#### ข้อมูลเพิ่มเติม
- **เอกสารอ้างอิง:** [Gemini API Documentation](https://ai.google.dev/docs)
- **ราคา:** มี Free tier ให้ใช้งานฟรีในระดับหนึ่ง
- **Rate Limits:** ตรวจสอบ rate limits ได้ที่ [Pricing Page](https://ai.google.dev/pricing)

---

### 1.2 การสร้าง OAuth 2.0 Client IDs

OAuth 2.0 Client ID ใช้สำหรับการเชื่อมต่อกับบริการของ Google เช่น Google Sheets, Google Drive, Gmail

#### ขั้นตอนการสร้าง OAuth 2.0 Client IDs

1. **เข้าสู่ Google Cloud Console**
   - ไปที่ [https://console.cloud.google.com/](https://console.cloud.google.com/)
   - เข้าสู่ระบบด้วย Google Account

2. **สร้างหรือเลือก Project**
   - คลิกที่ dropdown ด้านบนซ้าย (ชื่อ Project)
   - คลิก **"New Project"** เพื่อสร้าง Project ใหม่ หรือเลือก Project ที่มีอยู่
   - ตั้งชื่อ Project เช่น "n8n-integration"
   - คลิก **"Create"**

3. **เปิดใช้งาน APIs**
   - ไปที่ **"APIs & Services"** > **"Library"**
   - ค้นหาและเปิดใช้งาน APIs ต่อไปนี้:
     - ✅ **Google Sheets API**
     - ✅ **Google Drive API**
     - ✅ **Gmail API**
   - คลิก **"Enable"** สำหรับแต่ละ API

4. **ตั้งค่า OAuth Consent Screen**
   - ไปที่ **"APIs & Services"** > **"OAuth consent screen"**
   - เลือก **"External"** (สำหรับการใช้งานทั่วไป)
   - คลิก **"Create"**

   **กรอกข้อมูล:**
   - **App name:** ระบุชื่อแอป เช่น "n8n Workflow"
   - **User support email:** เลือก email ของคุณ
   - **Developer contact information:** กรอก email ของคุณ
   - คลิก **"Save and Continue"**

   **Scopes:**
   - คลิก **"Add or Remove Scopes"**
   - เลือก Scopes ที่ต้องการ เช่น:
     - `.../auth/spreadsheets` (Google Sheets)
     - `.../auth/drive` (Google Drive)
     - `.../auth/gmail.send` (Gmail)
   - คลิก **"Update"** และ **"Save and Continue"**

   **Test Users (สำหรับ Development):**
   - คลิก **"Add Users"**
   - เพิ่ม email ที่จะใช้ทดสอบ
   - คลิก **"Save and Continue"**

5. **สร้าง OAuth 2.0 Credentials**
   - ไปที่ **"APIs & Services"** > **"Credentials"**
   - คลิก **"Create Credentials"** > **"OAuth client ID"**

   **เลือก Application type:**
   - **Application type:** เลือก **"Web application"**

   **ตั้งค่า:**
   - **Name:** ตั้งชื่อ เช่น "n8n OAuth Client"
   - **Authorized JavaScript origins:**
     - เพิ่ม URL ของ n8n instance เช่น `http://localhost:5678`
   - **Authorized redirect URIs:**
     - เพิ่ม Redirect URI ของ n8n เช่น:
       - `http://localhost:5678/rest/oauth2-credential/callback`
     - **⚠️ หมายเหตุ:** URL จะแตกต่างกันขึ้นอยู่กับ n8n instance ของคุณ

   - คลิก **"Create"**

6. **คัดลอก Credentials**
   - ระบบจะแสดง **Client ID** และ **Client Secret**
   - คลิก **"Download JSON"** เพื่อดาวน์โหลด credentials (แนะนำ)
   - หรือคัดลอก Client ID และ Client Secret เก็บไว้ในที่ปลอดภัย
   - **⚠️ สำคัญ:** อย่าแชร์ Client Secret กับผู้อื่น

#### การใช้งาน OAuth 2.0 ใน n8n

1. ใน n8n ไปที่ **Credentials** > **Create New**
2. เลือกประเภท credential ที่ต้องการ เช่น **"Google Sheets OAuth2 API"**
3. กรอก **Client ID** และ **Client Secret**
4. คลิก **"Connect my account"**
5. ระบบจะเปิดหน้าต่างให้ Authorize
6. เลือกบัญชี Google และอนุญาตการเข้าถึง

#### ข้อมูลเพิ่มเติม
- **เอกสารอ้างอิง:** [Google OAuth 2.0 Documentation](https://developers.google.com/identity/protocols/oauth2)
- **n8n Documentation:** [Google OAuth2 Setup](https://docs.n8n.io/integrations/builtin/credentials/google/)

---

## 2. OpenAI

OpenAI เป็นผู้ให้บริการ AI models เช่น GPT-4, GPT-3.5 ที่นิยมใช้งานกันอย่างแพร่หลาย

### 2.1 การสร้าง API Key ของ OpenAI

#### ขั้นตอนการสร้าง API Key

1. **สร้างบัญชี OpenAI**
   - ไปที่ [https://platform.openai.com/signup](https://platform.openai.com/signup)
   - สมัครสมาชิกด้วย email หรือ Google/Microsoft account

2. **ยืนยัน Email และเบอร์โทรศัพท์**
   - ยืนยัน email ที่ได้รับจาก OpenAI
   - ยืนยัน เบอร์โทรศัพท์มือถือ (จำเป็นสำหรับความปลอดภัย)

3. **เข้าสู่ API Keys Page**
   - ไปที่ [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - หรือคลิก **"API keys"** จากเมนูด้านซ้าย

4. **สร้าง API Key ใหม่**
   - คลิกปุ่ม **"Create new secret key"**
   - ตั้งชื่อ API Key (ไม่บังคับ) เช่น "n8n-integration"
   - คลิก **"Create secret key"**

5. **คัดลอก API Key**
   - ระบบจะแสดง API Key ให้เพียงครั้งเดียว
   - คลิก **"Copy"** เพื่อคัดลอก API Key
   - **⚠️ สำคัญ:** เก็บ API Key ไว้ในที่ปลอดภัย เพราะจะไม่สามารถดูได้อีก

6. **ตั้งค่า Billing (จำเป็น)**
   - ไปที่ **"Billing"** > **"Payment methods"**
   - เพิ่ม Credit card หรือ Debit card
   - ตั้งค่า Usage limits เพื่อควบคุมค่าใช้จ่าย

#### การใช้งานใน n8n

1. ใน n8n ไปที่ **Credentials** > **Create New**
2. เลือก **"OpenAI API"**
3. กรอก **API Key** ที่คัดลอกมา
4. คลิก **"Save"**

#### ข้อมูลเพิ่มเติม
- **เอกสารอ้างอิง:** [OpenAI API Documentation](https://platform.openai.com/docs)
- **ราคา:** [OpenAI Pricing](https://openai.com/pricing)
- **Usage Dashboard:** [https://platform.openai.com/usage](https://platform.openai.com/usage)
- **Rate Limits:** ขึ้นอยู่กับประเภทบัญชีและ tier ของคุณ

---

## 3. OpenRouter

OpenRouter เป็น API Gateway ที่ให้เข้าถึง AI models จากหลายผู้ให้บริการในที่เดียว (Claude, GPT-4, Gemini, ฯลฯ)

### 3.1 การสร้าง API Key ของ OpenRouter

#### ขั้นตอนการสร้าง API Key

1. **สร้างบัญชี OpenRouter**
   - ไปที่ [https://openrouter.ai/](https://openrouter.ai/)
   - คลิก **"Sign In"** หรือ **"Get Started"**
   - เข้าสู่ระบบด้วย Google account หรือ email

2. **เข้าสู่ API Keys Page**
   - คลิกที่ชื่อผู้ใช้มุมขวาบน
   - เลือก **"Keys"** หรือไปที่ [https://openrouter.ai/keys](https://openrouter.ai/keys)

3. **สร้าง API Key ใหม่**
   - คลิกปุ่ม **"Create Key"**
   - ตั้งชื่อ API Key (ไม่บังคับ) เช่น "n8n-workflow"
   - เลือก Permissions ที่ต้องการ (ปกติใช้ default ได้)
   - คลิก **"Create"**

4. **คัดลอก API Key**
   - ระบบจะแสดง API Key
   - คลิก **"Copy"** เพื่อคัดลอก API Key
   - **⚠️ สำคัญ:** เก็บ API Key ไว้ในที่ปลอดภัย

5. **เติมเงินเข้าบัญชี (ถ้าต้องการใช้งาน)**
   - ไปที่ **"Credits"** หรือ [https://openrouter.ai/credits](https://openrouter.ai/credits)
   - คลิก **"Buy Credits"**
   - เลือกจำนวนเงินที่ต้องการเติม
   - ชำระเงินผ่าน Credit card

#### การใช้งานใน n8n

1. ใน n8n ไปที่ **Credentials** > **Create New**
2. เลือก **"OpenRouter API"** (หรือ HTTP Request พร้อมกำหนด Header)
3. กรอก **API Key** ในรูปแบบ:
   - **Header:** `Authorization`
   - **Value:** `Bearer YOUR_API_KEY`

#### ข้อมูลเพิ่มเติม
- **เอกสารอ้างอิง:** [OpenRouter Documentation](https://openrouter.ai/docs)
- **ราคา:** [OpenRouter Pricing](https://openrouter.ai/docs#models) (ราคาแตกต่างกันตาม model ที่เลือกใช้)
- **Models ที่รองรับ:** [https://openrouter.ai/models](https://openrouter.ai/models)
- **Usage Dashboard:** ดูได้ใน Dashboard ของ OpenRouter

---

## 🔒 ความปลอดภัย

### ข้อควรระวังในการใช้งาน API Keys

1. **อย่าแชร์ API Key กับผู้อื่น**
   - API Key เป็นข้อมูลลับที่ไม่ควรแชร์

2. **อย่า Commit API Key ลง Git**
   - ใช้ Environment Variables แทน
   - เพิ่ม `.env` เข้า `.gitignore`

3. **ตั้งค่า Rate Limits และ Usage Limits**
   - จำกัดการใช้งานเพื่อป้องกันค่าใช้จ่ายที่ไม่คาดคิด

4. **ตรวจสอบ Usage เป็นประจำ**
   - เช็คการใช้งานและค่าใช้จ่ายเป็นประจำ

5. **Rotate API Keys เป็นระยะ**
   - เปลี่ยน API Key เป็นระยะเพื่อความปลอดภัย

6. **ใช้ Service-specific Keys**
   - สร้าง API Key แยกสำหรับแต่ละ service หรือ environment

---

## 📚 ทรัพยากรเพิ่มเติม

### เอกสารอ้างอิง
- [n8n Documentation](https://docs.n8n.io/)
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [OpenAI Platform Documentation](https://platform.openai.com/docs)
- [OpenRouter Documentation](https://openrouter.ai/docs)

### วิดีโอสอน (ตัวอย่าง)
- [n8n Official YouTube Channel](https://www.youtube.com/@n8n-io)
- [Google Cloud YouTube Channel](https://www.youtube.com/@GoogleCloudTech)

---

## ❓ FAQ

### Q: API Key หายไป ทำอย่างไร?
**A:** สร้าง API Key ใหม่และลบ API Key เก่าเพื่อความปลอดภัย

### Q: API Key ถูกจำกัดการใช้งาน?
**A:** ตรวจสอบ rate limits, usage limits และยอดเงินคงเหลือในบัญชี

### Q: ใช้ API Key เดียวกันหลาย Project ได้ไหม?
**A:** ได้ แต่แนะนำให้สร้างแยกเพื่อความปลอดภัยและง่ายต่อการจัดการ

### Q: OAuth 2.0 และ API Key ต่างกันอย่างไร?
**A:**
- **API Key:** ใช้สำหรับการเข้าถึง API โดยตรง เหมาะสำหรับ server-to-server
- **OAuth 2.0:** ใช้สำหรับการเข้าถึงข้อมูลผู้ใช้ โดยผู้ใช้ต้อง authorize ก่อน

---

**อัปเดตล่าสุด:** พฤศจิกายน 2568
