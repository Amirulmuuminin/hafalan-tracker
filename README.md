# 📖 Qur'an Memorization Tracking Database (Line-Based System - Mushaf Madinah)

---

# 🎯 PURPOSE

This database is designed to track:

- Hafalan baru (new memorization)
- Murojaah (revision cycle)
- Ujian (exam & preparation)
- Target hafalan per student
- Detailed daily activity logs
- Progress and milestone tracking

---

# 🧠 CORE CONCEPT

- **Event-based**: all activities stored in `daily_logs`
- **State-based**: summarized progress in `student_progress`
- **Milestone tracking**: exams and targets handled separately
- **Measurement system**: uses **juz → page → line (baris)**
- **Reference**: Mushaf Madinah (UTHMANI)

---

# 📏 STANDARD UNIT

```text
1 halaman = 15 baris
```

⚠️ Halaman per juz **tidak selalu sama** (Mushaf Madinah).

---

# 📚 JUZ PAGE MAPPING (MUSHAF MADINAH)

```text
Juz 1  : page 1   - 21  (21 pages)
Juz 2  : page 22  - 41  (20 pages)
Juz 3  : page 42  - 61
Juz 4  : page 62  - 81
Juz 5  : page 82  - 101
Juz 6  : page 102 - 121
Juz 7  : page 122 - 141
Juz 8  : page 142 - 161
Juz 9  : page 162 - 181
Juz 10 : page 182 - 201
Juz 11 : page 202 - 221
Juz 12 : page 222 - 241
Juz 13 : page 242 - 261
Juz 14 : page 262 - 281
Juz 15 : page 282 - 301
Juz 16 : page 302 - 321
Juz 17 : page 322 - 341
Juz 18 : page 342 - 361
Juz 19 : page 362 - 381
Juz 20 : page 382 - 401
Juz 21 : page 402 - 421
Juz 22 : page 422 - 441
Juz 23 : page 442 - 461
Juz 24 : page 462 - 481
Juz 25 : page 482 - 501
Juz 26 : page 502 - 521
Juz 27 : page 522 - 541
Juz 28 : page 542 - 561
Juz 29 : page 562 - 581
Juz 30 : page 582 - 604 (23 pages)
```

---

# 📦 TABLES

## 1. users

```sql
users (
  id UUID PRIMARY KEY,
  name VARCHAR,
  role ENUM('student', 'teacher', 'examiner'),
  created_at TIMESTAMP
)
```

---

## 2. student_progress

```sql
student_progress (
  id UUID PRIMARY KEY,
  student_id UUID,

  total_lines INT DEFAULT 0,
  total_pages INT DEFAULT 0,
  total_juz DECIMAL DEFAULT 0,

  last_juz_id INT,
  last_page INT,
  last_line INT,

  created_at TIMESTAMP,
  updated_at TIMESTAMP
)
```

---

## 3. daily_logs (CORE TABLE)

```sql
daily_logs (
  id UUID PRIMARY KEY,
  student_id UUID,
  teacher_id UUID,
  date DATE,

  category ENUM('hafalan_baru', 'murojaah'),
  type ENUM('setoran', 'persiapan_ujian', 'ujian'),

  juz_id INT NOT NULL,

  from_page INT,
  from_line INT,

  to_page INT,
  to_line INT,

  total_lines INT,

  pages INT,

  note TEXT,
  created_at TIMESTAMP
)
```

---

## 4. exam_sessions

```sql
exam_sessions (
  id UUID PRIMARY KEY,
  student_id UUID,
  examiner_id UUID,

  exam_type ENUM(
    'quarter_juz',
    'half_juz',
    'one_juz',
    'five_juz'
  ),

  juz_start INT,
  juz_end INT,

  status ENUM('pending', 'passed', 'failed'),
  exam_date DATE,

  created_at TIMESTAMP
)
```

---

## 5. murojaah_cycles

```sql
murojaah_cycles (
  id UUID PRIMARY KEY,
  student_id UUID,

  current_day INT,
  current_pages INT,

  last_completed_date DATE,
  created_at TIMESTAMP
)
```

---

## 6. target_hafalan

```sql
target_hafalan (
  id UUID PRIMARY KEY,
  student_id UUID,

  target_type ENUM('juz', 'page', 'line'),
  target_value INT,

  deadline DATE,
  created_at TIMESTAMP
)
```

---

# ⚙️ BUSINESS LOGIC

## Hafalan Baru (Line-Based)

```text
total_lines =
(to_line - from_line)
+ ((to_page - from_page) * 15)
```

---

## Murojaah Cycle

```text
3 → 6 → 9 → 12 → 15 → 18 → 20 → reset
```

---

## Progress Calculation

```text
total_pages = total_lines / 15
```

```text
total_juz = total_pages / 20 (approximation for UI only)
```

⚠️ Gunakan mapping juz untuk akurasi real.

---

## Exam Rules (Accurate Mushaf-Based)

```text
1/4 juz  ≈ 5 pages
1/2 juz  ≈ 10 pages
1 juz    = dynamic (based on mapping above)
5 juz    = cumulative based on mapping
```

---

# ✅ VALIDATION RULES

```text
1. page must be within juz range (based on mapping)
2. from_page <= to_page
3. from_line <= 15
4. to_line <= 15
5. if same page → from_line <= to_line
6. no cross-juz in single log
```

---

# 📊 QUERY PATTERNS

```sql
SELECT * FROM daily_logs
WHERE student_id = ?
ORDER BY date DESC;
```

```sql
SELECT * FROM exam_sessions
WHERE student_id = ?;
```

```sql
SELECT * FROM target_hafalan
WHERE student_id = ?;
```

```sql
SELECT * FROM student_progress
WHERE student_id = ?;
```

---

# 🧩 DESIGN PRINCIPLES

- Line-based as source of truth
- Page & juz derived from mapping
- Real-world Mushaf Madinah accurate
- Scalable & consistent tracking
- Avoid ayah-based ambiguity

---

# 🚀 NOTES FOR FRONTEND

- Always validate page using juz mapping
- Use lines for precise progress
- Use pages for UI display
- Use juz mapping for milestone logic

---

END OF DOCUMENT
