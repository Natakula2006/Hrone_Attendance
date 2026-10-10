# HROne Employee Attendance & Analytics API

A high-performance RESTful API built with **Python 3.11+**, **FastAPI**, and **MongoDB 6.0+** for managing employee attendance tracking, manual regularization with audit trails, and departmental analytics powered by native MongoDB aggregation pipelines.

---

## 🚀 Features

- **Employee Management**: Create and list employees with unique customer business codes (`emp_code`), custom shift timings, and department organization.
- **Attendance Punch-In/Out**:
  - **Punch-In**: Calculates `late_minutes` (Rule R2, 10-minute grace period) and IST attendance dates (Rule R1, including overnight shifts). Race condition safe via MongoDB unique index on `(emp_code, date)`.
  - **Punch-Out**: Calculates `work_hours` (Rule R4, rounded half-up to 2 decimals), `overtime_minutes` (Rule R3, 30-minute threshold), and `half_day` status (Rule R5, < 4.50 hours). Atomic updates prevent duplicate punch-outs.
- **Manual Attendance Correction (Regularization)**: Recomputes all derived metrics and appends immutable audit entries to `history` storing `at`, `by`, `reason`, and `changes` (`from`/`to` values).
- **MongoDB Aggregation Pipelines**:
  - `GET /analytics/employees/{emp_code}/monthly`: Monthly stats (working days R7, present days R5-R7, late count, OT, attendance %).
  - `GET /analytics/departments/summary`: Cross-collection departmental summary including employees with zero logs (Rule R9).
  - `GET /analytics/leaderboard/late`: Standard competition ranking (1, 2, 2, 4) using `$setWindowFields`.
  - `GET /analytics/departments/{department}/trend`: Daily trend series with gap-filling and 7-day moving averages.
- **Admin Query Diagnostics**:
  - `GET /admin/explain/{endpoint}`: Returns raw MongoDB `explain` output with `executionStats` verbosity for query optimization verification.

---

## 📋 Prerequisites

- **Python**: 3.11 or newer (tested on Python 3.13)
- **MongoDB**: 6.0 or newer (or MongoDB Atlas free tier M0)

---

## ⚙️ Installation & Configuration

1. **Clone Repository**:
   ```bash
   git clone <repository-url>
   cd Hrone
   ```

2. **Create Virtual Environment & Install Dependencies**:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Configure Environment Variables**:
   Create a `.env` file in the root directory (or set environment variables):
   ```env
   MONGO_URI=mongodb://localhost:27017
   MONGO_DB=attendance_db
   ```

4. **Seed Sample Data** (Optional):
   ```bash
   python sample_seed.py
   ```

---

## 🏃 Running the Application

Start the FastAPI application using Uvicorn:
```bash
uvicorn app.main:app --port 8000 --reload
```

- **Interactive API Documentation (Swagger UI)**:  
  Open [http://localhost:8000/docs](http://localhost:8000/docs) in your browser.
- **ReDoc Documentation**:  
  Open [http://localhost:8000/redoc](http://localhost:8000/redoc) in your browser.

---

## 🧪 Running Automated Tests

Run the full test suite using `pytest`:
```bash
pytest
```

---

## 📝 Known Limitations / Implemented Status

- **Implemented Requirements**: 100% of the OpenAPI specification, business rules (R1 - R10), audit history requirements, and aggregation pipelines are fully implemented.
- **Known Limitations**: None. All endpoints are fully optimized with compound database indexes to handle 100,000+ attendance records.
