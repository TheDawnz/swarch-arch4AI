# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** อัครชัย ยงสุวรรณกุล

**Student ID / รหัสนิสิต:** 6630357821

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 512 | 611 | 93.5% | 100% |
| t0-small | 53 | 118 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> Large model จะให้ผลลัพท์ที่แม่นยำมากกว่า แต่จะประมวลผลช้ากว่า

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 28.9 s | 243 | 356 | 20 | 0 | 92.8% |
| t1-after | 9.8 s | 402 | 1022 | 0 | 26 | 93.1% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

>p50 วัดเฉพาะคำขอที่ตอบกลับเร็ว จึงไม่สะท้อนประสบการณ์จริงของผู้ใช้ ค่า errors จะบอกปัญหาได้

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

>เพราะการค้างแค่ 10% อาจทำให้เกิดลูกโซ่ที่ทำให้ระบบช้าและกว้างขึ้นได้

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

>บันทึกรีวิวไว้ก่อน แล้วส่งไปให้ batch job ประมวลผลใหม่ภายหลัง เรียกว่า Feature store
 
**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

>0.1 วิ น่าจะตัดเร็วเกินไป ทำให้เกิด pending และลด accuracy แต่ 10 วิ จะลด pending ได้แต่การตอบสนองจะช้าลง

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 23 | 528 | 91.5% | 38% |
| t2-0.7 | 0.7 | 399 | 582 | 93.0% | 53% |
| t2-0.8 | 0.8 | 452 | 601 | 93.5% | 64% |
| t2-0.9 | 0.9 | 482 | 604 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

>0.8 น่าจะดีสุดเพราะ accuracy ที่เท่ากับ large cell และลดเวลาลงได้ แต่หากลองเทสหลายรอบความแม่นยำอาจลดลงเล็กน้อย

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

>p50 ได้จากรีวิวที่ small ตอบ ทำให้ถ้า small มาก p50 จะน้อย

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> กรณีรีวิวมีคำประชดหรือปฏิเสธซับซ้อน เช่น ไม่แย่เลย

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 17 | 559 | 93.5% | 40% | 34% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

>รีวิวที่ซ้ำๆจะได้ประโยชน์สูง ในทางกลับกัน รีวิวที่แปลกๆจะไม่ได้ประโยชน์

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

>ต้องล้าง cache เดิมออก ควรออกแบบ ให้มี model/version รวมอยู่ด้วย

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

>ได้รู้เกี่ยวกับการแบ่งงานโดยใช้ประโยชน์จากตัวประมวลผลที่เล็กและใหญ่ให้คุ้มค่า และการใช้ cache ร่วมเพื่อให้การประมวลผลและตอบกลับดีที่สุด
