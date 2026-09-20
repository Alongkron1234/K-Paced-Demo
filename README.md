# K-PACED

**AI Financial Guidance & Adaptive Scam Protection for First Jobbers on K PLUS**

> AI guides the pace. You stay in control.

Prototype สำหรับ KBTG Kampus Hackathon 2026 — Track 2: Data Science & Intelligence

---

## ปัญหา

First Jobbers (21–25 ปี) เจอปัญหาการเงินสองเรื่องพร้อมกันในช่วงเดียวกันของชีวิต

1. **บริหารเงินเดือนก้อนแรกไม่เป็น** — ต้นเดือนใช้เยอะ กลางเดือนเริ่มร่อยหรอ สิ้นเดือนไม่เหลือ
2. **ตกเป็นเป้าของ APP Scam** — Authorized Push Payment คือกลโกงที่เหยื่อกดยืนยันโอนเองด้วยความเต็มใจ ทำให้ระบบ anti-fraud เดิมที่ออกแบบมาจับมัลแวร์ตามไม่ทัน

ทั้งสองเรื่องเกิดกับคนกลุ่มเดียวกัน และปะทุในจังหวะเดียวกัน คือตอนเงินกำลังจะหมดหรือตอนอยากหาเงินเพิ่ม

## แนวคิดหลัก

K-PACED ไม่ได้เอาระบบออมกับระบบกันโกงมาวางคู่กัน แต่ใช้ **"จังหวะการเงินที่ผิดปกติของผู้ใช้"** เป็นสะพานเชื่อมสองระบบ โดยทั้งคู่ใช้ข้อมูลพฤติกรรมชุดเดียวกัน

```
Transaction & Behavioral Data
            ↓
     Shared Feature Store
        ↙         ↘
   Model 1        Model 2
Spending Guidance  Scam Detection
   (Batch)        (Real-time)
```

ทำงานบนบัญชี K-eSavings เดิม ผู้ใช้ไม่ต้องเปิดบัญชีใหม่

---

## หน้าใน prototype

| ไฟล์ | เนื้อหา |
|---|---|
| `index.html` | **App Demo** — แอปที่กดเล่นได้เหมือนของจริง ภาษาคนทั่วไป ไม่มีศัพท์เทคนิค |
| `shortfall.html` | **โมเดลบริหารเงิน** — Monte Carlo 1,500 เส้นทาง คำนวณความน่าจะเป็นที่จะเงินขาดมือ |
| `architecture.html` | **โครงสร้างระบบ** — แยกเงินที่ K-Paced ดูแลกับยอดอิสระ พร้อม decision log ของสองโมเดล |

---

## สิ่งที่ทำให้ต่างจากแอปจัดการเงินทั่วไป

**ตอบเป็นความน่าจะเป็น ไม่ใช่ตัวเลขงบ** — Model 1 ไม่ได้บอกว่า "สัปดาห์นี้ใช้ได้เท่าไหร่" แต่บอกว่า "โอกาสที่คุณจะไปไม่ถึงวันเงินเดือนออกเท่าไหร่" ทำให้ทุกคำแนะนำวัดผลด้วยตัวเลขเดียวกันได้

**แยกรายได้ที่วางแผนได้ออกจากรายได้ที่เผื่อไม่ได้** — เงินเดือน OT ฟรีแลนซ์ ไม่ใช่เงินประเภทเดียวกัน ระบบให้ Income Reliability Score รายสาย แล้วนับเฉพาะส่วนที่เชื่อถือได้เข้าแผนประจำ

**หักกลบเงินที่เพื่อนจะโอนคืน** — จับคู่เงินออกก้อนใหญ่กับเงินเข้าจากบุคคลหลายรายภายในไม่กี่วัน ทำให้งบไม่เพี้ยนเวลาจ่ายแทนเพื่อน

**Commitment Cliff** — พยากรณ์จุดที่ภาระผ่อนจบหรือเริ่ม แล้วดักเงินที่ว่างขึ้นเข้าออมก่อนถูกกลืนด้วยค่าใช้จ่ายใหม่

**ถังเงินคือเจตนาที่ผู้ใช้ประกาศไว้ล่วงหน้า** — ทำให้ Model 2 มีฟีเจอร์ที่ระบบ anti-fraud ทั่วไปไม่มี คือ intent–action mismatch

**ถามเหตุผลระหว่างหน่วงรายการ** — ช่วงหน่วงไม่ใช่บทลงโทษ แต่เป็นหน้าต่างเก็บข้อมูลชิ้นที่มีค่าที่สุด และระบบไม่ได้เชื่อคำตอบเฉย ๆ แต่เอาไปเทียบกับข้อมูลที่ธนาคารมีอยู่แล้ว

---

## สถาปัตยกรรมโมเดล

### Model 1 — Dynamic Spending Predictor (batch)

- Temporal Fusion Transformer / LightGBM Quantile Regression + Pinball Loss
- ตั้งใจให้ทำนายค่อนไปทางเผื่อเหลือ เพราะให้งบน้อยเกินไปเจ็บกว่าให้มากเกินไป
- Recurring Bill Detection ด้วย autocorrelation, MCC ย้อนหลัง 3–6 เดือน, spending volatility, future context
- **Model 1B — Seasonal Liquidity Forecaster**: ปฏิทินสภาพคล่อง 12 เดือน สำหรับรายจ่ายที่รู้ล่วงหน้าแต่ไม่สม่ำเสมอ

### Model 2 — Anomaly Transfer Intent Engine (real-time, target < 150 ms)

Dual-Trigger เป็นตัวปลุกแบบเบา ไม่ใช่ตัวตัดสิน

1. High-Stakes Moment — ขอถอนจากเงินสำรอง
2. Micro-transfer Velocity — โอนถี่ผิดปกติไปบัญชีปลายทางใหม่

เมื่อถูกปลุก โมเดลมองธุรกรรมจากหลายมุมพร้อมกัน

| ชั้น | เทคนิค | จับอะไร |
|---|---|---|
| Behavioral | XGBoost | urgency dynamics, clipboard, สถานะสายโทรศัพท์ |
| Sequential | LSTM | laddering — โอนน้อยทดสอบแล้วขยายเป็นก้อนใหญ่ |
| Graph | GraphSAGE / RGCN | ความเชื่อมโยงกับกลุ่มบัญชีม้า, fan-out velocity |
| Narrative | LLM | เหตุผลที่ผู้ใช้ตอบระหว่างหน่วง เทียบกับหลักฐานที่ธนาคารมี |

- Focal Loss / Cost-Sensitive Learning สำหรับ extreme class imbalance
- SHAP แปลงเป็นข้อความเตือนที่ผู้ใช้อ่านเข้าใจ
- Graph embeddings คำนวณล่วงหน้าแบบ batch เก็บใน Feature Store — online layer แค่ดึงมา score

### บันไดมาตรการ 5 ขั้น

1. ปล่อยรายการทันที (ตัดเวลารอที่เหลือ)
2. แจ้งเตือนเป็นข้อความ
3. ยืนยันตัวตนเพิ่ม
4. หน่วง 2 ชั่วโมง + เจ้าหน้าที่ติดต่อกลับ
5. ต้องทำรายการที่สาขา

> LLM ผลิตค่า `narrative_risk` เป็นฟีเจอร์หนึ่งตัวเท่านั้น **ไม่มีอำนาจอนุมัติหรือระงับเงิน** การตัดสินใจยังอยู่ที่ระบบที่ตรวจสอบย้อนหลังได้

---

## ข้อจำกัดที่รู้ตัว

| ความเสี่ยง | แนวทางบรรเทา |
|---|---|
| Cold Start — First Jobber ไม่มีประวัติ 3–6 เดือน | cohort-based baseline ก่อน แล้วค่อย personalize |
| Latency 150 ms ร่วมกับ GNN | precompute graph embeddings แบบ batch, online ดึงมา score เท่านั้น |
| False Positive | risk-based step-up ไม่ใช่ hard block, monitor FPR อย่างจริงจัง |
| iOS จำกัด clipboard / in-call detection | เป็น secondary signal, core detection ใช้ in-app signals ที่เก็บได้ทุก platform |
| Mule account ข้ามธนาคาร | เฟสแรกใช้กราฟภายในเครือ KBank, ข้ามธนาคารเป็น Phase 2 ตามทิศทาง Central Fraud Registry |
| ข้อความเหตุผลเป็นข้อมูลอ่อนไหว | ไม่เก็บ raw text เก็บเฉพาะ category ที่จำแนกได้ ต้องมี consent ตอน onboarding |

---

## ตัวชี้วัด

- **Days-at-Risk ต่อรอบเงินเดือน** (ตัวหลัก)
- Average Savings Rate
- Scam Recall / False Positive Rate
- Fraud Loss Prevented
- Emergency Buffer Untouched Rate
- Feature Adoption

---

## การรัน

ทุกหน้าเป็น HTML ไฟล์เดียวจบ ไม่มี build step ไม่มี dependency เปิดด้วยเบราว์เซอร์ได้ทันที

```bash
git clone <repo-url>
cd k-paced-demo
open index.html
```

### Deploy ด้วย GitHub Pages

1. push ขึ้น repo
2. Settings → Pages → Source เลือก branch `main` โฟลเดอร์ `/ (root)`
3. รอสักครู่แล้วเข้าที่ `https://<username>.github.io/<repo-name>/`

### Deploy ด้วย Netlify

ลากโฟลเดอร์ทั้งหมดไปวางที่ [app.netlify.com/drop](https://app.netlify.com/drop)

---

## หมายเหตุ

ตัวเลขทั้งหมดใน prototype เป็นข้อมูลจำลองเพื่อการนำเสนอ ตัวจำแนกเหตุผลใน demo ใช้ keyword matching เพราะรันฝั่งเบราว์เซอร์ ของจริงจะเรียกผ่าน LLM API

โปรเจคนี้เป็นงานนำเสนอแนวคิด ไม่ใช่ผลิตภัณฑ์จริงของธนาคารกสิกรไทย
