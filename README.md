# CS371-MiniProject

สรุปโครงงาน Mini-Project สำหรับรายวิชา CS371: การจำแนกประเภทยานพาหนะจากชุดข้อมูลอุบัติเหตุทางถนนในประเทศไทย (Thailand Fatal Road Accident 2011–2022)

## ภาพรวม

โครงการนี้ใช้ชุดข้อมูลอุบัติเหตุทางถนนที่มีผู้เสียชีวิตจำนวน 240,924 ราย (2011–2022) เพื่อสร้างท่อข้อมูล (data pipeline) และโมเดลการจำแนกประเภท (classification) ที่พยายามทำนาย `vehicle_type` (10 คลาส) โดยทำขั้นตอนตั้งแต่การทำความสะอาดข้อมูล การสกัดคุณลักษณะ (feature engineering) การคัดเลือกคุณลักษณะที่สำคัญ (feature selection) และการเทรน/ประเมินผลของโมเดลหลายแบบ (kNN, XGBoost, MLP)

## โครงสร้างไฟล์ในรีโป

- `thailand_fatal_raod_accident_2011_2022.csv` — ชุดข้อมูลดิบ
- `miniproject.ipynb` / `pakorn-miniproject.ipynb` — โน้ตบุ๊กหลัก: โค้ดทั้งหมดของ pipeline และผลการรัน
- `feature_importance.ipynb` — การวิเคราะห์ความสำคัญของ feature (XGBoost + permutation importance)
- `feature_selection.ipynb` — การเลือกคุณลักษณะด้วย nested cross-validation
- `knn_example.ipynb`, `mlp_example.ipynb` — ตัวอย่าง/ทดลองโมเดล
- `document.md` — รายงานฉบับเต็มที่สรุปผล (อัปเดตตามผลจาก `miniproject.ipynb`)

## ขั้นตอนหลักในงานวิจัย (Pipeline)

1. Exploratory Data Analysis และ Data Cleaning
	- ลบแถวซ้ำ (2,478 แถว) → เหลือ 238,446 แถว
	- จัดการค่าสูญหาย: `age` เติม median (38.0), `accident_date` มี NaN ~132K แถว → สกัดฟีเจอร์ datetime แล้วเติมค่า median สำหรับฟีเจอร์ที่สกัด
	- กรองค่าพิกัดนอกขอบเขตประเทศไทย และเติม median
	- แปลง `accident_cause_code` เป็นกลุ่มกว้าง (`cause_group`) เพื่อลด data leakage
	- ใช้ `province_en` (One-Hot) เท่านั้น—ลบ district/sub_district ที่มี cardinality สูง

2. Feature Engineering
	- สกัด `hour`, `day_of_week`, `month`, `year` จาก `accident_date`
	- Label/One-Hot Encoding สำหรับตัวแปรหมวดหมู่
	- ผลลัพธ์หลัง FE: 90 features

3. Feature Importance & Selection
	- ใช้ XGBoost + Permutation Importance
	- Nested CV เพื่อหาจำนวน feature ที่เหมาะสม → แนะนำ `k = 85` (คงเหลือ 85 features)

4. Modeling & Hyperparameter Tuning
	- เปรียบเทียบ 3 อัลกอริธึม: kNN, XGBoost, MLP (sklearn)
	- ใช้ GridSearchCV กับ Stratified CV (k-fold)
	- Best params (สรุปจากการรัน):
	  - kNN: `metric='manhattan'`, `n_neighbors=15`, `weights='uniform'`
	  - XGBoost: `learning_rate=0.1`, `max_depth=6`, `n_estimators=300`, `colsample_bytree=0.8`, `subsample=0.8`
	  - MLP: `alpha=0.01`, `hidden_layer_sizes=(128, 64)`, `learning_rate='constant'`, `early_stopping=True`

5. Evaluation
	- Train/Test split: 80/20 stratified → Train 190,756 / Eval 47,690 (ด้วย 85 features)
	- Metrics (Accuracy, Precision/Recall/F1 weighted)
	- ผลสรุป Accuracy (Evaluation set): kNN 72.19%, XGBoost 75.84%, MLP 74.32%
	- XGBoost ให้ผลดีที่สุดและไม่มีสัญญาณ overfitting (gap CV–Eval ≈ -0.08%)

## ข้อจำกัดและข้อเสนอแนะ

- ชุดข้อมูลมี class imbalance รุนแรง (เช่น `unidentified` และ `moterbike` รวม > 86% ของข้อมูล) — คลาสย่อยหลายตัวมีตัวอย่างน้อยมาก (bus, tricycle, agricultural_vehicle) ผลคือ recall ของคลาสเหล่านี้ ≈ 0
- ข้อควรระวังเรื่อง data leakage: `accident_cause_code` ถูกแปลงเป็น `cause_group` เพื่อป้องกันโมเดล "เรียนรู้" ตัวบ่งชี้ยานพาหนะโดยตรง
- หากต้องการปรับปรุง: ใช้เทคนิค oversampling (SMOTE/ADASYN), class-weighting, หรือการรวบรวม/สร้าง features เพิ่มเติม (จากพิกัดเชิงพื้นที่หรือข้อมูลสภาพอากาศ/เวลา)

## วิธีรัน (Quick start)

1. เปิด `miniproject.ipynb` ใน Jupyter / VSCode และรันทุกเซลล์ (Notebook มีผลลัพธ์ที่บันทึกไว้)
2. หรือ ถ้าต้องการรันเป็นสคริปต์ ให้ตั้ง environment ด้วย Python 3.10–3.11 และติดตั้ง dependencies จาก `requirements.txt` (ถ้ามี)

ตัวอย่างคำสั่งติดตั้ง dependencies (ถ้ามีไฟล์ requirements.txt):
```bash
python -m venv .venv
source .venv/bin/activate   # หรือ: .venv\\Scripts\\activate (Windows)
pip install -r requirements.txt
jupyter lab
```

## แหล่งอ้างอิง

- ชุดข้อมูล: Thailand Fatal Road Accident (2011–2022)
- เอกสารโน้ตบุ๊ก: `miniproject.ipynb`, `feature_importance.ipynb`, `feature_selection.ipynb`

## ติดต่อ

หากต้องการให้ช่วยสรุปเป็นสไลด์ หรือเตรียมโค้ดสำหรับ deployment (API / model serving) บอกหัวข้อที่ต้องการได้เลย
