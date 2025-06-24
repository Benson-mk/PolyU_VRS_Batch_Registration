# PolyU VRS Batch Registration 📋

Batch-upload **up to 10 visitors per event** to The Hong Kong Polytechnic University’s Visitor Registration System (VRS) through the (undocumented) **AddRequestv2** API.  
Everything is wrapped in a single Jupyter notebook for maximum transparency.

---

## ✨  What this notebook does

1. **Reads three local files** you supply  
   * `Visitor.csv` – a list of individual visitors  
   * `Event.csv`   – event-level details (dates, sponsor info, etc.)  
   * `Cookies.txt` – four session cookies exported from your logged-in VRS browser session  
2. Splits visitors into batches of ≤ 10 (a VRS hard limit).  
3. Sanitises visit dates (caps stay at 13 days, no past dates).  
4. POSTs each batch to  https://fmovrs.polyu.edu.hk/vrs-ajax/ShortVisiting/AddRequestv2 and prints the HTTP status + JSON response.

---

## 🏃 Quick start

### Option A — Google Colab (zero install)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Benson-mk/PolyU_VRS_Batch_Registration/blob/main/polyu_vrs_batch_registration.ipynb)

1. Click the badge, then **Runtime → Run all**.  
2. When prompted, upload **Visitor.csv**, **Event.csv**, **Cookies.txt**.  
3. Watch the output pane for per-batch results.

### Option B — Local Jupyter

```bash
git clone https://github.com/Benson-mk/PolyU_VRS_Batch_Registration.git
cd PolyU_VRS_Batch_Registration

python -m venv .venv && source .venv/bin/activate

jupyter notebook polyu_vrs_batch_registration.ipynb
```

---

## 📑 Preparing the input files

### Event.csv

| accessDate | exitDate   | organization | purposeOfVisit | accessLocation | sponsorDepartment | sponsorName   | sponsorPhone | sponsorEmail           | sponsorNetID |
|------------|------------|--------------|----------------|----------------|-------------------|---------------|--------------|------------------------|--------------|
| 2025-07-01 | 2025-07-05 | ABC Ltd      | Demo Day       | AAB G/F        | COMP              | Dr Chan C.W.  | 12345678     | cw.chan@polyu.edu.hk   | cwchan       |

* Dates **must** be `YYYY-MM-DD`  —  *but* the notebook will self-heal if you leave a date blank or mistype it:

  | Situation                 | What the notebook does                       |
  | ------------------------- | -------------------------------------------- |
  | `accessDate` empty / bad  | Uses **today’s date**                        |
  | `accessDate` in the past  | Bumps it forward to **today**                |
  | `exitDate` empty / bad    | Sets it to **accessDate + 13 days**          |
  | Visit longer than 13 days | Trims `exitDate` to **accessDate + 13 days** |

  (13 days is hard-coded as `max_stay`; change the constant near the top of the notebook if VRS updates its rule.)

	
### Visitor.csv

| LName | FName | Email                  | Mobile   |
| ----- | ----- | ---------------------- | -------- |
| Smith | John  | john.smith@example.com | 91234567 |
| Doe   | Jane  | jane.doe@example.com   |          |
- `Mobile` is optional; blank cells are okay.
    
### Cookies.txt
Export **these four** cookies from an authenticated VRS tab:

```
BIGipServerPACI_VISITOR_44301_POOL
BIGipServerWAF_CM_PROD_HTTPS_POOL
.AspNetCore.Session
.AspNetCore.Cookies
```

Browser extensions such as _[Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)_ let you Export cookies in Netscape format; save that as `Cookies.txt`.

---

## 🔧 How it works, in 20 seconds

```text
┌── Visitor.csv (n rows) ─┐      ┌── Event.csv (m rows) ──┐
│ … visitor details …     │      │ … event details …      │
└─────────┬───────────────┘      └─────────┬──────────────┘
          │                                │
          └─▶ Notebook slices visitors ◀──┘ (≤ 10 per batch)
                    │
                    ├─ Creates JSON payload
                    └─ POST /ShortVisiting/AddRequestv2
```

- `max_visitors` and `max_stay` are set to **10** and **13** days respectively – change the constants near the top if VRS updates its rules.
    

---

## 🛠 Troubleshooting

| Symptom                             | Likely cause & fix                                        |
| ----------------------------------- | --------------------------------------------------------- |
| `HTTP 401` / `HTTP 403`             | Cookies expired → export a fresh `Cookies.txt`.           |
| `HTTP 400` “visitor count exceeded” | More than 10 visitors in a batch → check for blank lines. |
| `HTTP 500`                          | VRS hiccup → re-run that cell in a few minutes.           |

---

## 🤝 Contributing

Pull requests welcome – especially for:

- Handling future API schema changes
    
- Adding CLI support (notebook → script)
    
- Improving error handling / logging
    

---

## 📄 License

Released under the **MIT License**.  
This project is **not** affiliated with, endorsed by, or supported by The Hong Kong Polytechnic University. Use at your own risk.
