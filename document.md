# รายงาน Mini-Project: การจำแนกประเภทยานพาหนะจากข้อมูลอุบัติเหตุทางถนนในประเทศไทย

---

## บทนำ

### คำอธิบายภาพรวม

อุบัติเหตุทางถนนในประเทศไทยถือเป็นปัญหาวิกฤตระดับชาติที่ส่งผลกระทบอย่างรุนแรงต่อความมั่นคงในชีวิตและทรัพย์สินของประชาชนมาอย่างยาวนาน สถิติการเสียชีวิตและบาดเจ็บจากอุบัติเหตุจราจรยังคงอยู่ในระดับที่น่ากังวล ซึ่งสะท้อนให้เห็นถึงความจำเป็นเร่งด่วนในการหามาตรการป้องกันและแก้ไขปัญหาอย่างเป็นระบบ โครงงานนี้จึงได้นำชุดข้อมูล **Thailand Fatal Road Accident [2011-2022]** มาทำการศึกษา ซึ่งเป็นฐานข้อมูลขนาดใหญ่ที่รวบรวมรายละเอียดของอุบัติเหตุจราจรที่มีผู้เสียชีวิตในประเทศไทยครอบคลุมระยะเวลากว่า 10 ปี โดยได้รับการบูรณาการข้อมูลจากศูนย์ความร่วมมือข้อมูลการบาดเจ็บ (Injury Data Collaboration Center) กรมควบคุมโรค กระทรวงสาธารณสุข ร่วมกับภาคีเครือข่าย

### เรื่องราวของข้อมูลและการใช้ประโยชน์

ข้อมูลชุดนี้ประกอบด้วยรายละเอียดของอุบัติเหตุทางถนนที่มีผู้เสียชีวิตจำนวน **240,924 รายการ** ครอบคลุมตัวแปร 16 คอลัมน์ ทั้งข้อมูลด้านวันเวลาที่เกิดเหตุ สถานที่เกิดเหตุในระดับตำบลและอำเภอ ข้อมูลทางภูมิศาสตร์ในระบบละติจูดและลองจิจูด รวมถึงลักษณะทางประชากรศาสตร์ของผู้ประสบเหตุ เช่น เพศ อายุ และสัญชาติ

วัตถุประสงค์หลักของรายงานฉบับนี้ คือการประยุกต์ใช้กระบวนการวิทยาการข้อมูลและเทคนิคการเรียนรู้ของเครื่อง เพื่อสร้างแบบจำลองการจำแนกประเภท (Classification) สำหรับทำนาย **ประเภทยานพาหนะ (vehicle_type)** ที่เกี่ยวข้องกับอุบัติเหตุ ซึ่งมีทั้งหมด 10 คลาส ได้แก่ moterbike, unidentified, car, pedestrian, small_truck_or_van, bus, tricycle, hevay_truck, bike และ agricultural_vehicle

กระบวนการศึกษาจะเริ่มตั้งแต่การทำความสะอาดข้อมูล การคัดเลือกคุณลักษณะที่สำคัญด้วยเทคนิค Tree-based + Permutation Importance ไปจนถึงการเปรียบเทียบประสิทธิภาพของอัลกอริทึม 3 วิธี ได้แก่ kNN, XGBoost และ Neural Network (MLP) ผลลัพธ์ที่ได้จะสามารถนำไปใช้เป็นข้อมูลประกอบในการวางแผนยุทธศาสตร์ความปลอดภัยทางถนน เช่น การกำหนดมาตรการรณรงค์ให้ตรงกับกลุ่มเสี่ยง การจัดสรรทรัพยากรลงสู่พื้นที่ที่มีแนวโน้มเกิดอุบัติเหตุสูง หรือการออกมาตรการควบคุมพฤติกรรมเสี่ยงตามช่วงเวลาที่โมเดลระบุ

---

## คำอธิบายเกี่ยวกับข้อมูล

จากชุดข้อมูลผู้ประสบอุบัติเหตุทางถนนซึ่งเป็นข้อมูลระดับบุคคล โดยแต่ละแถวเป็นตัวแทนของผู้เสียชีวิตจากอุบัติเหตุ 1 ราย ข้อมูลส่วนใหญ่มีลักษณะเป็นข้อมูลเชิงคุณภาพที่ระบุหมวดหมู่และลักษณะเหตุการณ์ มีเพียงอายุและพิกัดทางภูมิศาสตร์ที่เป็นข้อมูลเชิงปริมาณ เพื่อความชัดเจนในการนำข้อมูลไปวิเคราะห์ต่อยอด ทางคณะผู้จัดทำขอแบ่งกลุ่มตัวแปรออกเป็น 4 หมวดหมู่หลัก ดังนี้:

### หมวดที่ 1: ข้อมูลเกี่ยวกับเวลาและเหตุการณ์ (Temporal & Event Data)

ข้อมูลกลุ่มนี้มีความสำคัญอย่างมากในการวิเคราะห์แนวโน้มช่วงเวลาที่มักเกิดอุบัติเหตุ

- **accident_date**
  - คำอธิบาย: วันที่และเวลาที่เกิดอุบัติเหตุขึ้นจริง (เช่น 2011-01-01 00:25:00)
  - ประเภทข้อมูล: Categorical (ประเภทย่อย Datetime)
  - บทบาท: Feature
  - เชิงลึก (Insight): สามารถนำมาทำ Feature Engineering เพื่อแยกเป็นคอลัมน์ใหม่ได้ เช่น ชั่วโมง (hour), วันในสัปดาห์ (day_of_week), เดือน (month), ปี (year) เพื่อหาปัจจัยเสี่ยงที่สัมพันธ์กับเวลา
  - **หมายเหตุ: มีค่าสูญหาย (NaN) จำนวน 132,022 แถว** — จะเติมด้วย median ของ datetime features ที่สกัดได้

- **official_death_date**
  - คำอธิบาย: วันที่ผู้ประสบอุบัติเหตุเสียชีวิตอย่างเป็นทางการ
  - ประเภทข้อมูล: Categorical (ประเภทย่อย Datetime)
  - บทบาท: Feature (ใช้อ้างอิง ไม่นำเข้าโมเดลโดยตรง)

### หมวดที่ 2: ข้อมูลด้านประชากรศาสตร์ (Demographic Data)

ข้อมูลส่วนบุคคลของผู้ประสบเหตุที่สามารถนำไปหาความสัมพันธ์กับประเภทยานพาหนะ

- **age**
  - คำอธิบาย: อายุของผู้เสียชีวิต (เช่น 21.0, 23.0)
  - ประเภทข้อมูล: Numerical (Continuous — ข้อมูลเชิงปริมาณแบบต่อเนื่อง)
  - บทบาท: Feature
  - เชิงลึก (Insight): **มีค่าสูญหาย 29,814 แถว** — เติมด้วยค่า median (38.0 ปี) ซึ่งทนทานต่อ outlier มากกว่า mean

- **gender**
  - คำอธิบาย: เพศของผู้เสียชีวิต (Male / Female)
  - ประเภทข้อมูล: Categorical (Nominal — ข้อมูลเชิงคุณภาพแบบไม่มีลำดับ)
  - บทบาท: Feature
  - การ Encoding: Label Encoding (Male = 1, Female = 0)

- **nationality**
  - คำอธิบาย: สัญชาติของผู้เสียชีวิต (Thai / Unknown)
  - ประเภทข้อมูล: Categorical (Nominal)
  - บทบาท: Feature
  - การ Encoding: Label Encoding (Thai = 1, Unknown = 0)

### หมวดที่ 3: ข้อมูลสาเหตุและยานพาหนะ (Accident Cause & Vehicle Data)

เป็นกลุ่มตัวแปรหลักที่ใช้อธิบายบริบทของเหตุการณ์ว่าเกิดขึ้นได้อย่างไรและเกี่ยวข้องกับอะไร

- **accident_cause_code**
  - คำอธิบาย: รหัสสาเหตุหลักของการเกิดอุบัติเหตุตามมาตรฐาน ICD (เช่น V892, V299) มี 231 ค่าที่แตกต่างกัน
  - ประเภทข้อมูล: Categorical (Nominal)
  - บทบาท: Feature (ต้องแปลงก่อนใช้)
  - **ข้อควรระวัง — Data Leakage**: รหัส ICD มีการ encode ประเภทยานพาหนะไว้ในตัว (เช่น V2xx = มอเตอร์ไซค์, V4xx = รถยนต์) ทำให้การใช้ค่าดิบจะทำให้โมเดลเรียนรู้ได้ง่ายเกินไปจนไม่สะท้อนประสิทธิภาพจริง → **วิธีแก้ไข: แปลงเป็นกลุ่มกว้างๆ (cause_group)** ตามลักษณะเหตุการณ์ ได้แก่ collision, noncollision, unspecified, other, unknown

- **accident_cause**
  - คำอธิบาย: คำอธิบายสาเหตุการเกิดอุบัติเหตุอย่างเป็นทางการแบบเต็มข้อความ (text ยาว)
  - ประเภทข้อมูล: Categorical (Text)
  - บทบาท: Feature (ใช้อ้างอิง — ลบออกเนื่องจากมี accident_cause_code แทน)

- **vehicle_type** ⭐ **ตัวแปรเป้าหมาย (Target Variable)**
  - คำอธิบาย: ประเภทยานพาหนะที่ผู้เสียชีวิตใช้งาน
  - ประเภทข้อมูล: Categorical (Nominal)
  - บทบาท: **Target (y)** — ตัวแปรที่ต้องการทำนาย
  - มี **10 คลาส** ดังนี้:

| คลาส | จำนวน (Samples) | สัดส่วน |
|-------|----------------|---------|
| unidentified | 114,446 | 47.5% |
| moterbike | 94,794 | 39.3% |
| car | 17,450 | 7.2% |
| pedestrian | 7,427 | 3.1% |
| small_truck_or_van | 4,028 | 1.7% |
| bike | 1,126 | 0.5% |
| hevay_truck | 885 | 0.4% |
| tricycle | 376 | 0.2% |
| bus | 298 | 0.1% |
| agricultural_vehicle | 94 | 0.04% |

  - **ปัญหาหลัก: Class Imbalance** — คลาส unidentified และ moterbike ครอบครองสัดส่วนรวมกว่า 86.8% ในขณะที่คลาส bus, tricycle, agricultural_vehicle มีจำนวนน้อยมาก

### หมวดที่ 4: ข้อมูลเชิงพื้นที่และพิกัดภูมิศาสตร์ (Spatial & Geographical Data)

- **province_th / district_th / sub_district_th**
  - คำอธิบาย: ชื่อ จังหวัด / อำเภอ / ตำบล เป็นภาษาไทย
  - ประเภทข้อมูล: Categorical (Nominal)
  - บทบาท: Feature (ลบออก เนื่องจากซ้ำซ้อนกับ _en)

- **province_en / district_en / sub_district_en**
  - คำอธิบาย: ชื่อ จังหวัด / อำเภอ / ตำบล เป็นภาษาอังกฤษ
  - ประเภทข้อมูล: Categorical (Nominal)
  - บทบาท: Feature
  - เชิงลึก (Insight): province_en มี 77 ค่า (จำนวนจังหวัดทั้งหมด) — ใช้ One-Hot Encoding ได้, district_en มี ~1,088 ค่า และ sub_district_en มี ~5,918 ค่า — **cardinality สูงเกินไป → ลบออก** ใช้เฉพาะ `province_en` เท่านั้น

- **latitude** และ **longitude**
  - คำอธิบาย: ค่าละติจูดและลองจิจูดระบุพิกัดที่เกิดอุบัติเหตุ
  - ประเภทข้อมูล: Numerical (Geospatial — ข้อมูลเชิงปริมาณระบุพิกัด)
  - บทบาท: Feature
  - **ปัญหา Noise**: พบค่าผิดปกติ 143 แถว (latitude อยู่นอกช่วง 5°N-21°N ของประเทศไทย) และ 159 แถว (longitude อยู่นอกช่วง 97°E-106°E) → แทนที่ด้วย NaN แล้วเติมด้วย median

### ตารางสรุปโครงสร้างข้อมูล

| ตัวแปร | ประเภท | บทบาท | หมายเหตุ |
|--------|--------|-------|----------|
| accident_date | Datetime | Feature | สกัด hour, day_of_week, month, year (NaN 132,022 rows) |
| official_death_date | Datetime | Feature | ลบออก (ไม่ใช้ในโมเดล) |
| age | Numerical | Feature | เติม NaN ด้วย median = 38.0 |
| gender | Categorical | Feature | Label Encoding (Male=1, Female=0) |
| nationality | Categorical | Feature | Label Encoding (Thai=1, Unknown=0) |
| accident_cause_code | Categorical | Feature | แปลงเป็น cause_group เพื่อลด leakage |
| accident_cause | Categorical (Text) | Feature | ลบออก (ซ้ำกับ accident_cause_code) |
| **vehicle_type** | **Categorical** | **Target** | **10 คลาส — ตัวแปรที่ต้องทำนาย** |
| province_th | Categorical | Feature | ลบออก (ซ้ำกับ _en) |
| district_th | Categorical | Feature | ลบออก (ซ้ำกับ _en) |
| sub_district_th | Categorical | Feature | ลบออก (ซ้ำกับ _en) |
| province_en | Categorical | Feature | One-Hot Encoding (77 จังหวัด) |
| district_en | Categorical | Feature | ลบออก (cardinality 1,088 — สูงเกินไป) |
| sub_district_en | Categorical | Feature | ลบออก (cardinality 5,918 — สูงเกินไป) |
| latitude | Numerical | Feature | กรอง noise แล้วเติม median |
| longitude | Numerical | Feature | กรอง noise แล้วเติม median |

---

## หลักการและขั้นตอน การสกัด การเลือก และการเตรียมลักษณะ (Data Preparation & Feature Selection)

### กระบวนการเตรียมข้อมูล (Data Preparation)

การเตรียมข้อมูลเป็นขั้นตอนที่สำคัญที่สุดในการทำเหมืองข้อมูล เนื่องจากชุดข้อมูลในโลกความเป็นจริงมักมีความไม่สมบูรณ์ การจัดการกับข้อบกพร่องเหล่านี้มีขั้นตอนละเอียดดังนี้:

#### 1. การกำจัดแถวซ้ำ (Duplicates Removal)

จากชุดข้อมูลดั้งเดิม 240,924 แถว พบแถวที่มีข้อมูลเหมือนกันทุกคอลัมน์จำนวน **2,478 แถว** การลบแถวซ้ำเป็นสิ่งจำเป็นเพื่อป้องกัน bias ในการเทรนโมเดล ทำให้เหลือข้อมูล **238,446 แถว**

#### 2. การกำจัดและการจัดการค่าสูญหาย (Missing Value Handling)

| คอลัมน์ | จำนวน NaN | วิธีจัดการ | เหตุผล |
|---------|-----------|-----------|--------|
| accident_date | 132,022 | เติม datetime features ด้วย median | ไม่ลบแถว เพราะจะสูญเสียข้อมูลกว่า 55%; แทนที่สกัด hour/day_of_week/month/year แล้วเติมค่า NaN ใน features เหล่านั้นด้วย median |
| age | 29,814 | เติมด้วย **median = 38.0** | ใช้ median แทน mean เนื่องจาก robust ต่อ outlier ใน distribution ของอายุ |
| gender | 0 | ไม่ต้องจัดการ | — |
| nationality | 0 | ไม่ต้องจัดการ (ค่า Unknown ถือเป็นหมวดหมู่หนึ่ง) | — |
| accident_cause_code | 0 | ไม่ต้องจัดการ | — |
| latitude / longitude | 0 (ก่อนกรอง noise) | กรอง noise → เติม median | ดูหัวข้อถัดไป |

#### 3. การกำจัดสัญญาณรบกวนและข้อมูลผิดปกติ (Noise & Outlier Removal)

สำหรับข้อมูลพิกัดภูมิศาสตร์ ใช้ **ความรู้เฉพาะทาง (Domain Knowledge)** ในการกำหนดขอบเขตที่ถูกต้อง:

- **ละติจูดประเทศไทย**: 5°N – 21°N
- **ลองจิจูดประเทศไทย**: 97°E – 106°E

ค่าที่อยู่นอกขอบเขตดังกล่าวถือเป็น Noise จากการบันทึกข้อมูลผิดพลาด:
- พบ **latitude ผิดปกติ 143 แถว** (เช่น ค่า -74° ซึ่งเป็นไปไม่ได้สำหรับประเทศไทย)
- พบ **longitude ผิดปกติ 159 แถว** (เช่น ค่า -92°)

วิธีจัดการ: แทนที่ค่าผิดปกติด้วย NaN แล้วเติมด้วย **median** ของคอลัมน์ทั้งหมด

#### 4. Feature Engineering

- **สกัด Datetime Features** จาก `accident_date`:
  - `hour` (0-23): ชั่วโมงที่เกิดเหตุ
  - `day_of_week` (0=จันทร์, 6=อาทิตย์): วันในสัปดาห์
  - `month` (1-12): เดือนที่เกิดเหตุ
  - `year` (2011-2022): ปีที่เกิดเหตุ

- **แปลง accident_cause_code เป็น cause_group** เพื่อลด Data Leakage:
  รหัส ICD ดั้งเดิม (เช่น V234, V892) มีการ encode ประเภทยานพาหนะไว้ในตัว ถ้านำไปใช้ตรงๆ โมเดลจะ "โกง" ได้ จึงแปลงเป็นกลุ่มกว้างๆ ตามลักษณะเหตุการณ์:
  - `collision` — อุบัติเหตุแบบชน
  - `noncollision` — อุบัติเหตุแบบไม่ชน (เช่น พลิกคว่ำ, ตกถนน)
  - `unspecified` — ไม่ระบุลักษณะ
  - `other` — อื่นๆ
  - `unknown` — ไม่ทราบสาเหตุ

  ผลการกระจายตัวของ cause_group:
  - unspecified: 100,684 แถว
  - collision: 65,005 แถว
  - other: 43,456 แถว
  - unknown: 18,474 แถว
  - noncollision: 10,827 แถว

#### 5. การลบคอลัมน์ที่ไม่ใช้

| คอลัมน์ที่ลบ | เหตุผลที่ลบ |
|--------------|-------------|
| accident_date | สกัด datetime features แล้ว |
| official_death_date | ไม่เกี่ยวข้องกับการทำนาย vehicle_type |
| accident_cause_code | แปลงเป็น cause_group แล้ว |
| accident_cause | text ยาว ซ้ำซ้อนกับ cause_code |
| province_th, district_th, sub_district_th | ซ้ำกับ _en |
| district_en, sub_district_en | cardinality สูงเกินไป (1,088 และ 5,918 ค่า ตามลำดับ) |

#### 6. Encoding

| ตัวแปร | วิธี Encoding | รายละเอียด |
|--------|--------------|------------|
| gender | Label Encoding | Male = 1, Female = 0 |
| nationality | Label Encoding | Thai = 1, Unknown = 0 |
| province_en | One-Hot Encoding (drop_first=True) | 76 คอลัมน์ใหม่ (77 จังหวัด ลบ 1 reference) |
| cause_group | One-Hot Encoding (drop_first=True) | 4 คอลัมน์ใหม่ (5 กลุ่ม ลบ 1 reference) |
| vehicle_type (Target) | LabelEncoder | 10 คลาส → 0-9 |

**ผลลัพธ์หลัง Feature Engineering:** X shape = **238,446 แถว × 90 คุณลักษณะ**

---

### การลดทอนมิติข้อมูลด้วยวิธี Tree-based + Permutation Importance

เมื่อชุดข้อมูลมีจำนวนลักษณะ (Features) สูงถึง 90 มิติ จะทำให้เกิดปัญหา "Curse of Dimensionality" ซึ่งส่งผลให้โมเดลซับซ้อนเกินไป (Overfitting) และใช้เวลาประมวลผลนาน การเลือกคุณลักษณะที่สำคัญจึงดำเนินการผ่าน 2 ขั้นตอนหลัก:

**1. Tree-based Importance (Gain):** เริ่มต้นจากการสร้างโมเดล XGBoost เบื้องต้น (`n_estimators=300, max_depth=4, learning_rate=0.05`) เพื่อดึงค่า 'Gain' ซึ่งเป็นการวัดว่าแต่ละคุณลักษณะสามารถช่วยลดความไม่บริสุทธิ์ของข้อมูล (Impurity) ในการแตกกิ่ง (Split) ได้มากน้อยเพียงใด

**2. กฎ 2/3 (2/3 Rule):** จากคุณลักษณะทั้งหมด 90 ตัว เลือก top ⌈2/3⌉ = **60 คุณลักษณะ** ที่มี GAIN สูงที่สุด แล้ว Retrain โมเดลใหม่

**3. Permutation Importance:** นำคุณลักษณะที่ผ่านการคัดกรองมาทำการสลับค่าแบบสุ่ม (Shuffle) ทีละตัว แล้ววัดว่าค่า Accuracy ลดลงไปเท่าไร (n_repeats=30) หากคุณลักษณะใดเมื่อถูกสลับค่าแล้วทำให้โมเดลแย่ลงมาก แปลว่าคุณลักษณะนั้นมีความสำคัญสูง

**ผลลัพธ์ประสิทธิภาพเบื้องต้น:**
- Baseline Accuracy (90 features ทั้งหมด): **0.7435**
- Accuracy หลัง 2/3 Rule (60 features): **0.7434** — แทบไม่ลดลงเลย แสดงว่าหลาย features ไม่มีประโยชน์

### การวิเคราะห์ผลลัพธ์ Feature Importance — Top 3 Features

จากการรันอัลกอริทึม XGBoost ควบคู่กับ Permutation Importance พบว่าคุณลักษณะที่มีความสำคัญสูงสุด 3 อันดับแรก (Top 3) ได้แก่:

| อันดับ | Feature | Importance Mean | Importance Std |
|--------|---------|----------------|----------------|
| 1 | **cause_unknown** | 0.1728 | ± 0.0010 |
| 2 | **year** | 0.1656 | ± 0.0013 |
| 3 | **nationality** | 0.1031 | ± 0.0008 |

#### ความเกี่ยวเนื่องกับคลาส (Target Class) — อธิบายเชิงลึก

**1. cause_unknown (Importance = 0.1728):**
คุณลักษณะนี้บ่งบอกว่าสาเหตุอุบัติเหตุ "ไม่ทราบ" หรือไม่ เมื่อสาเหตุไม่ถูกระบุ มักเกิดกับกรณีที่ผู้เสียชีวิตถูกพบในสภาพที่ไม่สามารถระบุรายละเอียดได้ ซึ่งสัมพันธ์อย่างยิ่งกับคลาส **"unidentified"** (ยานพาหนะไม่ระบุ) จึงเป็นตัวแยกหลักระหว่างคลาส unidentified กับคลาสอื่นๆ

**2. year (Importance = 0.1656):**
ปีที่เกิดอุบัติเหตุมีความสำคัญสูงเนื่องจากสัดส่วนของประเภทยานพาหนะเปลี่ยนแปลงตามเวลา ตัวอย่างเช่น ในช่วงปีหลังๆ (2018-2022) สัดส่วนของ moterbike อาจเพิ่มขึ้นตามจำนวนรถจักรยานยนต์ที่เพิ่มขึ้น หรือระบบบันทึกข้อมูลที่ดีขึ้นทำให้สัดส่วน unidentified ลดลง

**3. nationality (Importance = 0.1031):**
สัญชาติของผู้เสียชีวิตเป็นตัวแยกที่สำคัญ เนื่องจากผู้ที่มีสัญชาติ "Unknown" มีแนวโน้มจะเป็นคลาส unidentified สูง (ไม่สามารถระบุรายละเอียดได้ทั้งตัวบุคคลและยานพาหนะ) นอกจากนี้ รูปแบบการใช้ยานพาหนะอาจแตกต่างกันระหว่างกลุ่มสัญชาติ

### การหาจำนวนคุณลักษณะที่เหมาะสม (Feature Selection ด้วย Nested Cross-Validation)

ใช้เทคนิค **Nested Cross-Validation** (Outer CV 5-fold, Inner CV 3-fold) ควบคู่กับโมเดล XGBoost เพื่อค้นหาจำนวน $k$ ที่ทำให้โมเดลมีค่า Accuracy ดีที่สุดในแต่ละ Fold โดยทดสอบค่า $k$ ตั้งแต่ 5 ถึง 90 (ทุก 10 features)

**ผลลัพธ์ Nested CV:**

| Outer Fold | Best k | Outer Fold Accuracy |
|------------|--------|-------------------|
| Fold 1 | 85 | 0.7426 |
| Fold 2 | 85 | 0.7414 |
| Fold 3 | 85 | 0.7448 |
| Fold 4 | 85 | 0.7449 |
| Fold 5 | 90 | 0.7421 |

- **Mean Nested CV Accuracy:** 0.7432
- **Std Nested CV Accuracy:** 0.0014
- **Best k per fold:** [85, 85, 85, 85, 90]
- **Mean best k:** 86.00
- **Median best k:** 85
- **Std of best k:** 2.00

**สรุปจำนวนที่เลือก:** ระบบทำการคำนวณค่า **มัธยฐาน (Median)** และสรุปผลว่า จำนวนคุณลักษณะที่เหมาะสมที่สุด (Recommended number of features) คือ **85 คุณลักษณะ** จากทั้งหมด 90 คุณลักษณะ (ลบออก 5) การใช้ค่ามัธยฐานช่วยลดความลำเอียงจากโฟลด์ที่มีค่า $k$ แกว่ง

**ข้อสังเกต:** ค่า best k ค่อนข้างสูง (85-90 จาก 90) แสดงว่าคุณลักษณะส่วนใหญ่ล้วนมีส่วนช่วยในการจำแนกประเภท แม้จะน้อยก็ตาม ทั้งนี้เป็นเพราะ One-Hot Encoding ของ province_en สร้างคุณลักษณะจังหวัดจำนวนมาก ซึ่งแต่ละจังหวัดอาจมีรูปแบบการเกิดอุบัติเหตุที่แตกต่างกัน

### คุณลักษณะที่เหลืออยู่หลัง Feature Selection

หลังจากตัดคุณลักษณะที่ไม่จำเป็นออก 5 ตัว คงเหลือ **85 คุณลักษณะ** สำคัญ แบ่งเป็น:

- **คุณลักษณะหลัก (Non-province):** cause_unknown, cause_unspecified, cause_other, cause_noncollision, year, nationality, age, gender, latitude, longitude, hour, day_of_week, month
- **คุณลักษณะจังหวัด (Province One-Hot):** prov_Trang, prov_Udon Thani, prov_Kamphaeng Phet, prov_Ubon Ratchathani, prov_Buri Ram, prov_Nakhon Sawan, prov_Suphan Buri, prov_Lop Buri, prov_Kanchanaburi, prov_Krabi ฯลฯ (รวม 72 จังหวัด)

---

## เทคนิคที่ใช้ (Modeling Techniques and Hyperparameter Tuning)

### การจัดเตรียมข้อมูลสำหรับการสร้างแบบจำลอง (Data Splitting)

เพื่อให้การประเมินประสิทธิภาพมีความน่าเชื่อถือและป้องกันปัญหาการจำข้อมูล (Overfitting) ข้อมูลได้ถูกแบ่งออกเป็น 2 ชุดหลักในสัดส่วน **80/20 (Train/Test Split)** แบบ **Stratified Sampling** เพื่อให้แน่ใจว่าสัดส่วนของทุกคลาส (10 คลาส) ในทั้งสองชุดเท่ากัน:

1. **ชุดข้อมูลสำหรับการสร้างและทวนสอบ (Cross Validation Set — 80%):** จำนวน **190,756 ตัวอย่าง × 85 features**
   - ข้อมูลส่วนนี้ถูกนำไปใช้ในกระบวนการฝึกสอน (Training) ร่วมกับ **k-Fold Stratified Cross Validation** เพื่อหาค่า Hyperparameter ที่เหมาะสมที่สุดด้วยวิธี Grid Search

2. **ชุดข้อมูลทดสอบ (Evaluation Set / Hold-out Set — 20%):** จำนวน **47,690 ตัวอย่าง × 85 features**
   - ชุดข้อมูลนี้ถูกเก็บไว้เป็น **Unseen Data** และจะถูกเรียกใช้เพียงครั้งเดียวในขั้นตอนสุดท้าย เพื่อเปรียบเทียบว่าโมเดลสามารถทำงานกับข้อมูลที่ไม่เคยเห็นมาก่อนได้ดีเพียงใด และเพื่อตรวจจับปัญหา Overfitting

### เทคนิคการจำแนกประเภท (Classification Algorithms) ที่เลือกใช้

#### 1. การจำแนกประเภทโดยอาศัยเพื่อนบ้าน (k-Nearest Neighbors — kNN)

- **หลักการ:** เป็นขั้นตอนวิธีแบบ Instance-based Learning โดยจัดกลุ่มข้อมูลใหม่ตามเสียงข้างมากของข้อมูลใกล้เคียงจำนวน $k$ ตัว (อิงตามระยะทาง เช่น Euclidean หรือ Manhattan Distance) ใช้ร่วมกับ `StandardScaler` ใน Pipeline เพื่อปรับสเกลข้อมูลก่อน
- **ข้อดี:** แนวคิดเรียบง่าย ไม่ต้องมีสมมติฐานเกี่ยวกับการแจกแจงของข้อมูล (Non-parametric) และฝึกสอนโมเดลได้อย่างรวดเร็ว
- **ข้อเสีย:** อ่อนไหวต่อสเกลข้อมูล (ต้องทำ Scaling เสมอ), มีปัญหา "Curse of Dimensionality" หาก features เยอะ, ใช้เวลา Inference นานกับข้อมูลขนาดใหญ่ (190K+ ตัวอย่าง), และจัดการ class imbalance ไม่ดี

#### 2. XGBoost (Extreme Gradient Boosting)

- **หลักการ:** เป็นเทคนิค Ensemble แบบ Boosting ที่สร้าง Decision Tree หลายๆ ต้นเรียงต่อกัน โดยต้นไม้ต้นถัดไปจะพยายามแก้ไขข้อผิดพลาด (Residuals) ของต้นไม้ก่อนหน้า พร้อมทั้งมีระบบ Regularization ใช้ `multi:softprob` สำหรับ multiclass classification (10 คลาส)
- **ข้อดี:** ให้ความแม่นยำสูง (State-of-the-Art สำหรับ Tabular Data), จัดการความสัมพันธ์แบบ Non-linear ได้ดี, ไม่ต้อง scale ข้อมูล, จัดการ missing values ได้ในตัว, และให้ feature importance ที่เป็นประโยชน์
- **ข้อเสีย:** มี Hyperparameter จำนวนมากที่ต้อง tune, อาจ overfit ถ้า depth สูงเกินไป

#### 3. เครือข่ายประสาทเทียม (Neural Network — MLPClassifier)

- **หลักการ:** ใช้สถาปัตยกรรมแบบ Multi-Layer Perceptron (MLP) จาก scikit-learn ที่ประกอบด้วย Input Layer, Hidden Layer(s) และ Output Layer พร้อม L2 Regularization (alpha) และ Early Stopping เพื่อป้องกัน Overfitting ใช้ร่วมกับ `StandardScaler` ใน Pipeline
- **ข้อดี:** สามารถเรียนรู้ความสัมพันธ์ที่ซับซ้อนและมีมิติสูงได้ดี, มี Regularization หลายวิธี (L2/alpha, EarlyStopping), ยืดหยุ่นในการออกแบบสถาปัตยกรรม
- **ข้อเสีย:** ต้องการทรัพยากรสูง (Computationally Expensive), ต้อง scale ข้อมูลอย่างเคร่งครัด, เสี่ยงต่อ Overfitting หากไม่มี Regularization, และไม่สามารถตีความได้ง่าย (Black Box)

### การค้นหาค่าพารามิเตอร์ที่ดีที่สุด (Grid Search Optimization)

ในชุด Cross Validation Set มีการใช้ฟังก์ชัน **GridSearchCV** เพื่อกวาดหาชุดค่าพารามิเตอร์ที่เป็นไปได้ (Exhaustive Search) พร้อม **Stratified k-Fold Cross Validation**

#### Parameter Grid ของแต่ละเทคนิค:

**kNN** (Pipeline: StandardScaler + KNeighborsClassifier, 10-Fold CV):

| Parameter | ค่าที่ทดสอบ |
|-----------|------------|
| n_neighbors | 3, 5, 7, 9, 11, 13, 15 |
| weights | uniform, distance |
| metric | euclidean, manhattan |

รวม 7 × 2 × 2 = **28 combinations** × 10 folds = 280 fits

**XGBoost** (10-Fold CV):

| Parameter | ค่าที่ทดสอบ |
|-----------|------------|
| n_estimators | 100, 300 |
| max_depth | 3, 4, 6 |
| learning_rate | 0.05, 0.1 |
| subsample | 0.8 |
| colsample_bytree | 0.8 |

รวม 2 × 3 × 2 = **12 combinations** × 10 folds = 120 fits

**Neural Network (MLP)** (Pipeline: StandardScaler + MLPClassifier, 5-Fold CV):

| Parameter | ค่าที่ทดสอบ |
|-----------|------------|
| hidden_layer_sizes | (128, 64), (64, 32) |
| alpha (L2 regularization) | 0.001, 0.01 |
| learning_rate | constant, adaptive |

รวม 2 × 2 × 2 = **8 combinations** × 5 folds = 40 fits
(ใช้ `early_stopping=True`, `validation_fraction=0.15`, `n_iter_no_change=15`)

#### ผลลัพธ์ Best Parameter Set ของแต่ละเทคนิค:

| เทคนิค | Best Parameters |
|--------|----------------|
| **kNN** | `metric='manhattan'`, `n_neighbors=15`, `weights='uniform'` |
| **XGBoost** | `learning_rate=0.1`, `max_depth=6`, `n_estimators=300`, `colsample_bytree=0.8`, `subsample=0.8` |
| **MLP** | `alpha=0.01`, `hidden_layer_sizes=(128, 64)`, `learning_rate='constant'` |

**วิเคราะห์ Best Parameters:**
- **kNN** เลือก $k=15$ (ค่าสูงสุดในเซต) เพราะข้อมูลมี noise สูงและ class imbalance ทำให้ต้องดูเพื่อนบ้านมากขึ้นเพื่อให้เกิด smoothing effect, ใช้ manhattan distance ซึ่งมักดีกว่า euclidean กับ high-dimensional data
- **XGBoost** เลือก max_depth=6 (ลึกที่สุด) และ n_estimators=300 (มากที่สุด) แสดงว่าต้องการโมเดลที่มี capacity สูงเพื่อจับ pattern ที่ซับซ้อนของ 10 คลาส
- **MLP** เลือก hidden_layer_sizes=(128, 64) ซึ่งเป็น architecture ที่ใหญ่กว่า (64, 32) และ alpha=0.01 ซึ่งมี regularization มากกว่า แสดงว่า architecture ใหญ่พร้อม regularization สูงให้ผลลัพธ์ดีกว่า architecture เล็กที่ไม่มี regularization

---

## การประเมินประสิทธิภาพของแบบจำลอง (Model Performance Evaluation)

### วิเคราะห์ผล เปรียบเทียบ และอภิปรายผลลัพธ์เชิงตัวเลข

จากการนำโมเดลทั้ง 3 เทคนิค (ด้วย Best Parameters) ไปทดสอบในกระบวนการ Cross Validation และทดสอบกับ Evaluation Set (20% Hold-out) โดยใช้มาตรวัดความแม่นยำ (Accuracy Score) ได้ผลลัพธ์ดังนี้:

| เทคนิคการเรียนรู้ (Model) | ประสิทธิภาพจาก CV Set (Best CV Accuracy) | ประสิทธิภาพจาก Evaluation Set (Eval Accuracy) | ค่าความต่าง (Gap = CV - Eval) | Overfitting? |
|---------------------------|------------------------------------------|-----------------------------------------------|------------------------------|-------------|
| kNN | 71.92% | 72.19% | -0.27% | ไม่เกิด |
| **XGBoost** | **75.76%** | **75.84%** | **-0.08%** | **ไม่เกิด** |
| Neural Network (MLP) | 74.38% | 74.32% | +0.05% | ไม่เกิด |

### การวินิจฉัยปัญหา Underfitting และ Overfitting

#### พิจารณาเรื่อง Underfitting (การเข้ากันน้อยเกินไป):

ไม่พบปัญหา Underfitting อย่างรุนแรง เนื่องจากทุกโมเดลทำคะแนนได้สูงกว่า 70% ทั้งบน CV Set และ Evaluation Set ซึ่งดีกว่าการสุ่มทาย (baseline ~47.5% จากคลาส majority = unidentified) อย่างมีนัยสำคัญ อย่างไรก็ตาม Accuracy ระดับ 72-76% แสดงว่ายังมี room for improvement ซึ่งอาจเกิดจากข้อจำกัดของ features ที่มีอยู่ (หลายคุณลักษณะถูกลดลงเพื่อป้องกัน data leakage) และ class imbalance ที่รุนแรง

#### พิจารณาเรื่อง Overfitting (การเข้ากันมากเกินไป):

| โมเดล | Gap (CV - Eval) | การวินิจฉัย |
|--------|----------------|-------------|
| kNN | -0.27% | **ไม่เกิด Overfitting** — Gap น้อยกว่า 2% และ Eval สูงกว่า CV เล็กน้อย แสดงว่าโมเดล generalize ได้ดี |
| XGBoost | -0.08% | **ไม่เกิด Overfitting** — Gap แทบเป็น 0 แสดงว่า Generalization ดีเยี่ยม |
| MLP | +0.05% | **ไม่เกิด Overfitting** — Gap ต่ำกว่า 0.1% ซึ่งถือว่าดีมาก Early Stopping ทำหน้าที่ป้องกัน Overfitting ได้อย่างมีประสิทธิภาพ |

**สรุป:** ทั้ง 3 โมเดลมีค่า Gap ระหว่าง CV กับ Evaluation น้อยกว่า 1% ทั้งหมด ซึ่ง:
- Gap < 2% ถือว่า **ไม่เกิด Overfitting**
- Gap ≈ 0% แสดงว่า **Generalization ดีมาก**
- กรณี Eval > CV เล็กน้อย (kNN, XGBoost มี Gap ติดลบ) เป็นเรื่องปกติที่เกิดจาก randomness ในการสุ่มแบ่ง partition

**เหตุผลที่ไม่เกิด Overfitting:**
1. **kNN:** ใช้ $k=15$ (ค่าสูง) ซึ่งทำให้โมเดล smooth ไม่ sensitive ต่อ noise
2. **XGBoost:** มี Regularization ในตัวผ่าน subsample=0.8 และ colsample_bytree=0.8 ที่ทำ Random Subsampling
3. **MLP:** ใช้ Early Stopping (patience=15), L2 Regularization (alpha=0.01), และ validation_fraction=0.15

### แสดงผลตัวเลขประเมินประสิทธิภาพโดยรวมจากข้อมูลทดสอบ (Evaluation Set) — เปรียบเทียบสรุป

| Metric | kNN | XGBoost | MLP |
|--------|-----|---------|-----|
| **Eval Accuracy** | 72.19% | **75.84%** | 74.32% |
| Gap (CV - Eval) | -0.27% | **-0.08%** | +0.05% |

---

### การสรุปเพื่อเลือกใช้แบบจำลอง (Final Model Selection)

เมื่อพิจารณาภาพรวมของตัวเลขประสิทธิภาพจากชุดข้อมูลทดสอบ (Evaluation Set) ซึ่งเป็นเครื่องบ่งชี้ความสามารถที่แท้จริงของแบบจำลอง พบว่า **XGBoost** ให้ผลลัพธ์ดีที่สุดใน **ทุกเมตริก** และห่างจากอันดับ 2 (MLP) ประมาณ 1.5%

**ขอเสนอให้เลือกใช้ "XGBoost" ด้วยเหตุผลประกอบดังนี้:**

1. **Accuracy สูงสุด:** XGBoost ทำ Eval Accuracy ได้ **75.84%** ซึ่งสูงกว่า MLP (74.32%) และ kNN (72.19%)

2. **Generalization ดีเยี่ยม:** ค่า Gap ระหว่าง CV กับ Eval เพียง **-0.08%** (แทบเป็นศูนย์) แสดงว่าโมเดลไม่เกิด Overfitting และสามารถใช้ได้จริงกับข้อมูลที่ไม่เคยเห็น

3. **ความสามารถในการตีความ (Interpretability):** XGBoost มีกลไก Feature Importance ที่บอกได้ว่าตัวแปรใดสำคัญที่สุด (เช่น cause_unknown, year, nationality) ซึ่งช่วยให้ผู้ใช้เข้าใจเหตุผลเบื้องหลังการทำนาย ในขณะที่ Neural Network ทำตัวเป็นกล่องดำ (Black Box)

4. **ไม่ต้อง Scale ข้อมูล (Scale Invariance):** XGBoost อิงจาก Decision Tree จึงไม่ต้องการ StandardScaler ทำให้ Data Pipeline เรียบง่ายกว่า MLP และ kNN

5. **ทนทานต่อ Class Imbalance:** แม้จะมี 10 คลาสที่ imbalance อย่างรุนแรง XGBoost ก็ยังจัดการได้ดีกว่าโมเดลอื่นทุกตัว

6. **เวลาในการ Inference:** ต่างจาก kNN ที่ต้องคำนวณระยะทางกับทุกจุด (190K+ ตัวอย่าง) XGBoost ทำนายได้รวดเร็วมากเนื่องจากใช้เพียงการ traverse ตาม tree

**ข้อจำกัดที่พบร่วมกันของทั้ง 3 โมเดล:**
- คลาสที่มีจำนวนน้อย (agricultural_vehicle: 94, bus: 298, tricycle: 376, bike: 1,126) ทุกโมเดลทำนายได้ไม่ดีเนื่องจาก class imbalance ที่รุนแรงมาก
- Accuracy ระดับ 72-76% แสดงว่ายังมีพื้นที่ในการปรับปรุง ซึ่งอาจเกิดจากข้อจำกัดของ features (ลบ accident_cause_code เพื่อป้องกัน data leakage) และ class imbalance
- หากต้องการปรับปรุงในอนาคต ควรพิจารณาใช้เทคนิค SMOTE/Oversampling หรือ class_weight เพื่อช่วยให้โมเดลเรียนรู้คลาสที่มีจำนวนน้อย

