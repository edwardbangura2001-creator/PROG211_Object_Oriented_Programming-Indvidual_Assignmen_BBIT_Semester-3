# Citizen Complaint System

**PROG211 – Object-Oriented Programming 1 · Individual Assignment: Real-World Solutions under DPG Standards**
**Domain:** Government · **System:** Citizen Complaint System · **Python 3 (standard library only)**

A free, open-source command-line program that lets citizens report problems with public services: roads, water, electricity, waste collection and health facilities. Every complaint:
- is **sent automatically to the responsible department**;
- gets a **reference number** and a **target resolution time** based on its priority;
- moves through a fixed lifecycle: **Submitted → In progress → Resolved** (or **Rejected**).

Citizens can track their complaint and rate the service once it is resolved. Officials can see overdue complaints and which departments perform best. **Citizens may stay completely anonymous.**

---

## 1. Why this matters

When a road is damaged, water stops or rubbish is not collected, citizens often do not know where to report it, and complaints made by word of mouth are easily lost. Without records, nobody can show how long problems take to fix or which services need improvement. This system gives every complaint a reference number, a responsible department, a deadline and a full history, making public services more **transparent and accountable**.

## 2. Features

| # | Option | What it does |
|---|---|---|
| 1 | View all complaints | Table of every complaint with category, district, priority, status, days open and an **OVERDUE** flag |
| 2 | File a new complaint | Choose a category and priority; the complaint is routed to the right department and given a reference and target |
| 3 | Update a complaint's status | Only valid moves are allowed (e.g. a *Submitted* complaint cannot jump straight to *Resolved*); each change is added to the history |
| 4 | Rate a resolved complaint | 1–5 stars, allowed only once a complaint is resolved |
| 5 | Track a complaint | Full details and dated history, like a tracking slip |
| 6 | Register a citizen | With a first name **or fully anonymous**; only the district is required |
| 7 | **Withdraw (delete) a complaint** | Removes it from the citizen, the department and the register, after typing **YES** |
| 8 | **Delete a citizen** | Removes the citizen **and every complaint they filed**, after typing **YES** |
| 9 | **Search complaints** | By reference, category, district, status or any word in the description |
| 10 | **Best-performing department** | The department with the highest performance score (the "top scorer") |
| 11 | **Department performance report** | For every department: totals, open, resolved, resolution %, average rating, average days, % on time and score |
| 12 | **Overdue complaints** | Every open complaint past its target, and how many days late |
| 13 | System summary | Totals by status, plus the priority targets |
| 0 | Exit | |

**Performance score (out of 100)** = 50% resolution rate + 30% average citizen rating + 20% of complaints resolved on time.

**Priority targets:** High = 3 days, Medium = 7 days, Low = 14 days.

## 3. How to run

You only need Python 3.8 or newer.

```bash
git clone https://github.com/<your-username>/citizen-complaint-system.git
cd citizen-complaint-system

python complaint_system.py          # interactive menu (starts with sample data)
python complaint_system.py --demo   # automatic demonstration of every requirement
python complaint_system.py --test   # run the 10 unit tests
```

## 4. Code structure

All the code is in **one file**, `complaint_system.py`:

| Section | Contents |
|---|---|
| 1. Fixed data | Tuples: `DISTRICTS` (the 16 districts of Sierra Leone), `CATEGORIES`, `PRIORITIES`, `STATUS_FLOW`, `OPEN_STATUSES`, `RATING_RANGE`, plus `target_days()` and `category_details()` |
| 2. Classes | `Citizen`, `Department`, `Complaint` |
| 3. Record functions | `add_record()`, `find_record()`, `withdraw_complaint()`, `delete_citizen()`, `display_complaints()`, `search_complaints()` |
| 4. Sample data | `load_sample_data()`: fictional citizens, complaints, updates and ratings |
| 5. Input helpers | `ask()`, `ask_until_valid()`, `non_empty()`, `valid_district()`, `valid_rating()`: wrong input never crashes the program |
| 6. Menu actions | One function per menu option |
| 7. Menu and demo | `main()` and `run_demo()` |
| 8. Unit tests | 10 tests using `unittest` |

## 5. How the assignment tasks are met

### Task 1 – Classes, attributes and objects

| Class | Attributes | Methods |
|---|---|---|
| `Citizen` | `citizen_id`, `first_name`, `district`, `complaints` (dict); class attributes `total_citizens`, `next_number` | `add_complaint()`, `open_complaints()`, `display_info()`, `register()` [class], `register_anonymous()` [class], `get_total_citizens()` [class], `record_removal()` [class], `normalise_district()` [static], `is_valid_district()` [static] |
| `Department` | `code`, `name`, `complaints` (dict) | `receive()`, `count_by_status()`, `resolution_rate()`, `average_rating()`, `average_days_to_resolve()`, `on_time_rate()`, `performance_score()`, `statistics()`, `from_category()` [class], `find_best()` [static] |
| `Complaint` | `reference`, `citizen`, `department`, `category`, `description`, `priority`, `date_filed`, `status`, `date_closed`, `rating`, `history` (tuple); class attributes `total_complaints`, `next_number` | `is_open()`, `days_open()`, `is_overdue()`, `update_status()`, `rate()`, `display_row()`, `display_details()`, `from_text()` [class], `get_total_complaints()` [class], `record_removal()` [class], `normalise_priority()` [static], `is_valid_rating()` [static], `parse_date()` [static] |

**How the objects interact:**
- A `Complaint` object holds the real `Citizen` and `Department` objects.
- When it is created, it registers itself with **both**: `citizen.add_complaint(self)` and `department.receive(self)`.
- The citizen can then count its open complaints.
- The department calculates its own resolution rate, rating, on-time rate and score from the complaint objects it holds.
- Withdrawing a complaint removes it from all three places.

```python
citizen = Citizen.register_anonymous("Kono")
department = Department("DEL", "Electricity Department")
complaint = Complaint(citizen, department, "POWER", "Transformer sparking", "High", "2026-10-10")
complaint.update_status("In progress", "Engineer assigned", "2026-10-11")
```

### Task 2 – Data structure: dictionaries

The program uses **one data structure, the dictionary**, to store objects. The unique ID is the key:

```python
citizens    = {"C001": <Citizen>, "C002": <Citizen>, ...}
departments = {"DPW": <Department>, "DWS": <Department>, ...}
complaints  = {"CMP-0001": <Complaint>, "CMP-0002": <Complaint>, ...}
citizen.complaints = {"CMP-0001": <Complaint>, ...}
```

A citizen tracking their complaint only knows the reference number, so a dictionary keyed by reference finds it instantly. Duplicate references are impossible.

Functions in Section 3:
- **Add:** `add_record()` adds an object and refuses duplicate keys.
- **Find:** `find_record()` looks an object up by its ID, in any case.
- **Display:** `display_complaints()` prints every complaint.
- **Search:** `search_complaints()` finds complaints by reference, category, district, status or any word.
- **Delete:** `withdraw_complaint()` and `delete_citizen()`.

**Optional tuples (bonus):** all fixed rules are tuples, so they cannot be changed by accident:

```python
PRIORITIES  = (("High", 3), ("Medium", 7), ("Low", 14))
STATUS_FLOW = (("Submitted", "In progress"), ("Submitted", "Rejected"),
               ("In progress", "Resolved"), ("In progress", "Rejected"))
DISTRICTS   = ("Bo", "Bombali", "Bonthe", ..., "Western Area Urban")    # all 16
```

Tuples are also used in these places:
- **Complaint history:** each complaint's `history` is a tuple of `(date, status, note)` tuples. A new tuple is built for each update, so earlier entries can never be edited, which gives an honest audit trail.
- **Returned values:** `department.statistics()` returns an 8-value tuple, and `category_details()` returns one too.
- **The menu.**
- **Tuple unpacking**, for example `for when, status, note in self.history:`.

### Task 3 – Methods

| Type | Example | What it does |
|---|---|---|
| Instance | `complaint.update_status("Resolved", "Fixed")` | Moves one complaint along the lifecycle, checking `STATUS_FLOW` |
| Instance | `complaint.rate(5)` | Records satisfaction (only when resolved) |
| Instance | `complaint.is_overdue()` | True if still open after its target days |
| Instance | `department.performance_score()` | One department's score out of 100 |
| Class | `Citizen.register("Aminata", "Bo")` | Creates a citizen with an automatic ID |
| Class | `Citizen.register_anonymous("Kono")` | Creates an anonymous citizen |
| Class | `Complaint.from_text("C001, ROAD, Pothole, High, 2026-09-01", ...)` | Builds a complaint from one line of text and routes it to the right department |
| Class | `Department.from_category(entry)` | Builds a department from a `CATEGORIES` tuple |
| Class | `get_total_citizens()`, `get_total_complaints()`, `record_removal()` | Read and update the shared counters |
| Static | `Citizen.is_valid_district("bo")` | Checks a district, in any case |
| Static | `Complaint.is_valid_rating(6)` | Checks a rating is 1–5 |
| Static | `Complaint.parse_date("2099-01-01")` | Refuses badly formatted and future dates |
| Static | `Department.find_best(departments)` | Finds the best-performing department |

## 6. How the solution meets DPG standards

| DPG principle | How this project meets it |
|---|---|
| **Open-source** | Public GitHub repository under the **MIT licence**. It uses only the Python standard library, so councils, ministries and civil-society groups can use, study, change and share it for free. |
| **Inclusive and accessible design** | Works offline in any text terminal on a low-cost computer, for example at a council office or community centre. Plain-language numbered menu, and every prompt shows an example or the list of choices. Districts, categories, priorities and references are accepted in any capitalisation. Every answer is checked straight away and re-asked with a message starting `Problem:`, so the program never crashes. Status, overdue flags and ratings are shown in words and `*` symbols, not colours. Deleting always needs a typed **YES**. |
| **Privacy-respecting** | Citizens can be **fully anonymous**. Otherwise only a **first name** and **district** are stored. **No** surname, phone number, email, home address or national ID number is collected. Data stays on the local machine. Deleting a citizen removes all their complaints. All sample data is fictional. |
| **Modular and reusable** | Clearly separated sections, with one small function per feature. A new category or department is added with **one line** in `CATEGORIES`, and priority targets or allowed status changes are edited in their tuples, without touching the classes. The classes could power a web form, SMS service or mobile app. |

## 7. Example usage

### Filing a complaint (option 2): wrong answers are re-asked

```
  Option: 2
  Your citizen ID (e.g. C002): c002
  Categories:
    ROAD    Roads and potholes        -> Public Works Department
    WATER   Water supply              -> Water Services Department
    POWER   Electricity supply        -> Electricity Department
    WASTE   Waste collection          -> Sanitation Department
    HEALTH  Health facility service   -> Health Services Department
  Category code: park
  Problem: please type one of the category codes shown.
  Category code: road
  Describe the problem:
  Problem: the description cannot be empty.
  Describe the problem: Collapsed culvert on Dama Road
  Priority (High/Medium/Low): urgent
  Problem: please type High, Medium or Low.
  Priority (High/Medium/Low): high
  [OK] Complaint CMP-0009 sent to the Public Works Department.
       Target: resolved within 3 days. Keep your reference number.
```

### Updating status (option 3): the lifecycle is enforced

```
  Option: 3
  Complaint reference (e.g. CMP-0003): cmp-0003
  Current status: In progress
  New status (Resolved, Rejected): Resolved
  Note for the citizen: Pipe fixed
  CMP-0003: In progress -> Resolved

  Option: 3
  Complaint reference (e.g. CMP-0003): CMP-0005
  Current status: Submitted
  New status (In progress, Rejected): Resolved
  Note for the citizen:
  Problem: A complaint that is 'Submitted' cannot become 'Resolved'.
```

### Rating (option 4)

```
  Option: 4
  Complaint reference: CMP-0003
  How satisfied are you? (1 = very poor, 5 = excellent): seven
  Problem: please type a whole number from 1 to 5.
  How satisfied are you? (1 = very poor, 5 = excellent): 4
  [OK] Thank you. CMP-0003 rated **** (4/5).
```

### Tracking a complaint (option 5)

```
################################################################
  COMPLAINT CMP-0001
################################################################
  Filed by    : C001 (Aminata, Western Area Urban)
  Category    : Roads and potholes
  Department  : Public Works Department
  Problem     : Large pothole on main road near the market
  Priority    : High (target 3 days)
  Status      : Resolved
  Days open   : 3
  Rating      : *****
  History:
    2026-09-01  Submitted    Complaint received
    2026-09-02  In progress  Inspection team sent
    2026-09-04  Resolved     Pothole filled
################################################################
```

### Registering anonymously (option 6)

```
  Option: 6
  Stay anonymous? (Y/N): y
  Districts:
    Bo                  Bombali             Bonthe              Falaba
    Kailahun            Kambia              Karene              Kenema
    Koinadugu           Kono                Moyamba             Port Loko
    Pujehun             Tonkolili           Western Area Rural  Western Area Urban
  District: kono
  [OK] Registered C005 (Anonymous, Kono). Use ID C005 to file complaints.
```

### Withdrawing a complaint (option 7) and deleting a citizen (option 8)

```
  Option: 7
  Reference of the complaint to withdraw: cmp-0008
  CMP-0008: Public tap leaking (Submitted)
  Type YES to withdraw and delete it: YES
  [OK] CMP-0008 withdrawn. 7 complaint(s) remain.

  Option: 8
  Citizen ID to delete: C003
  Citizen ID : C003
  Name       : Anonymous
  District   : Kenema
  Complaints : 2 filed, 2 still open
  Type YES to delete this citizen and ALL their complaints: yes
  [OK] Deleted C003 and 2 complaint(s).
```

### Best department (option 10) and performance report (option 11)

```
  BEST-PERFORMING DEPARTMENT
  --------------------------------------------------
  Public Works Department (DPW)
  Performance score : 75/100
  Resolved          : 1 of 2 (50%)
  Citizen rating    : 5.0/5
  Average time      : 3.0 days to resolve, 100% on time

  DEPARTMENT PERFORMANCE REPORT
  --------------------------------------------------------------------------------
  Department                   Total Open Done  Rate Stars  Days OnTime  Score
  --------------------------------------------------------------------------------
  Public Works Department          2    1    1   50%   5.0   3.0   100%     75
  Water Services Department        2    2    0    0%   0.0   0.0     0%      0
  Electricity Department           1    0    1  100%   3.0  16.0     0%     68
  Sanitation Department            2    0    1   50%   4.0   4.0   100%     69
  Health Services Department       1    1    0    0%   0.0   0.0     0%      0
  Score = 50% resolution rate + 30% citizen rating + 20% resolved on time
```

### Overdue complaints (option 12)

```
  OVERDUE COMPLAINTS (open longer than their target, as of 2026-10-12)
  ------------------------------------------------------------------------
  CMP-0003  Water Services Department   High    24 day(s) late
  CMP-0005  Health Services Department  Medium  10 day(s) late
  CMP-0006  Public Works Department     High    8 day(s) late
  3 complaint(s) need urgent action.
```

> "Days open" and "days late" are counted up to **today's date**, so these numbers grow each day you run the program.

## 8. Testing

```
$ python complaint_system.py --test
...
Ran 10 tests in 0.002s

OK
```

The tests check:
- privacy, anonymous registration and district validation;
- that a complaint registers itself with both its citizen and its department;
- the status lifecycle: valid moves work, while skipping steps or going backwards is refused;
- that ratings are allowed only for resolved complaints and must be 1–5;
- overdue detection;
- department statistics and the best department;
- that deleting a citizen also deletes their complaints everywhere;
- `from_text()` and the validation of priority and dates;
- the tuple lookups.

## 9. Future improvements

- Save data to a file or database so it remains after the program closes.
- Add staff log-in so only department officers can change a complaint's status.
- Accept complaints by SMS or USSD for citizens without internet, and offer the menu in Krio.
- Produce monthly reports for each district.

## 10. Licence

MIT – see [LICENSE](LICENSE).
