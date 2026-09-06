# Accenture OA --- MS Office Quick Revision

> Brief, high-yield concepts, examples, formulas, shortcuts, and MCQ
> traps.

## 1. Microsoft Word

**Purpose:** Word processor for letters, reports, resumes, documents.

  Concept         Meaning / Example
  --------------- ---------------------------------------------------
  Font            Text appearance, e.g. Arial
  Bold            Thick text --- `Ctrl+B`
  Italic          Slanted text --- `Ctrl+I`
  Underline       Line below text --- `Ctrl+U`
  Alignment       Left, Center, Right, Justify
  Styles          Consistent formatting such as Heading 1
  Header          Content at top of page
  Footer          Content at bottom of page
  Page Break      Starts content on a new page --- `Ctrl+Enter`
  Section Break   Allows different formatting/layout in sections
  Mail Merge      Creates personalized documents from a data source
  Find            Search text --- `Ctrl+F`
  Replace         Find and change text --- `Ctrl+H`

### Word alignment

-   `Ctrl+L` → Left
-   `Ctrl+E` → Center
-   `Ctrl+R` → Right
-   `Ctrl+J` → Justify

### Common Word shortcuts

-   `Ctrl+N` New
-   `Ctrl+O` Open
-   `Ctrl+S` Save
-   `Ctrl+P` Print
-   `Ctrl+A` Select All
-   `Ctrl+C` Copy
-   `Ctrl+X` Cut
-   `Ctrl+V` Paste
-   `Ctrl+Z` Undo
-   `Ctrl+Y` Redo

**MCQ trap:** Page Break = new page; Section Break = separate section
formatting.

------------------------------------------------------------------------

## 2. Microsoft Excel

**Purpose:** Spreadsheet application for calculations, data analysis,
tables, charts, and reporting.

### Structure

`Workbook → Worksheet → Rows/Columns → Cells`

Example: `B2` = column B, row 2.

### Cell references

  Reference   Meaning
  ----------- -------------------------------------------------
  `A1`        Relative; row and column can change when copied
  `$A$1`      Absolute; row and column locked
  `$A1`       Column locked; row can change
  `A$1`       Row locked; column can change

### Basic formulas

Assume `A1:A5 = 10,20,30,40,50`.

  Formula        Purpose                  Example/result
  -------------- ------------------------ ------------------------
  `SUM`          Adds values              `=SUM(A1:A5)` → 150
  `AVERAGE`      Calculates average       `=AVERAGE(A1:A5)` → 30
  `MAX`          Largest value            `=MAX(A1:A5)` → 50
  `MIN`          Smallest value           `=MIN(A1:A5)` → 10
  `COUNT`        Counts numeric cells     `=COUNT(A1:A5)`
  `COUNTA`       Counts non-empty cells   `=COUNTA(A1:A5)`
  `COUNTBLANK`   Counts empty cells       `=COUNTBLANK(A1:A5)`

**Remember:** `COUNT` = numbers, `COUNTA` = non-empty, `COUNTBLANK` =
empty.

### Conditional formulas

**IF**

``` excel
=IF(A1>=40,"Pass","Fail")
```

If A1 is 50 → `Pass`.

**AND**

``` excel
=AND(A1>=40,B1>=40)
```

TRUE only when all conditions are TRUE.

**OR**

``` excel
=OR(A1>=40,B1>=40)
```

TRUE when at least one condition is TRUE.

**NOT**

``` excel
=NOT(A1=10)
```

Reverses TRUE/FALSE.

### COUNTIF

Counts cells matching a condition.

``` excel
=COUNTIF(A1:A10,">50")
```

Example: values `30,60,70,40` → result `2`.

### SUMIF

Adds values meeting a condition.

``` excel
=SUMIF(A1:A10,"Apple",B1:B10)
```

Adds B values where A contains `Apple`.

### AVERAGEIF

``` excel
=AVERAGEIF(A1:A10,">50")
```

Average of values satisfying the condition.

### Text formulas

  Formula    Example                    Result
  ---------- -------------------------- ----------------------
  `LEFT`     `=LEFT("ACCENTURE",3)`     `ACC`
  `RIGHT`    `=RIGHT("ACCENTURE",3)`    `URE`
  `MID`      `=MID("ACCENTURE",2,3)`    `CCE`
  `LEN`      `=LEN("Excel")`            `5`
  `TRIM`     `=TRIM(A1)`                Removes extra spaces
  `UPPER`    `=UPPER("excel")`          `EXCEL`
  `LOWER`    `=LOWER("EXCEL")`          `excel`
  `PROPER`   `=PROPER("hello world")`   `Hello World`
  `CONCAT`   `=CONCAT(A1," ",B1)`       Combines text

### ROUND

``` excel
=ROUND(12.567,2)
```

Result → `12.57`.

### VLOOKUP

Searches vertically in the **first column** of a table.

``` excel
=VLOOKUP(101,A2:C10,3,FALSE)
```

Meaning: - Find `101` - Search first column - Return column 3 - `FALSE`
= exact match

**VLOOKUP:** vertical search.\
**HLOOKUP:** horizontal search.

``` excel
=HLOOKUP(101,A1:F3,3,FALSE)
```

### XLOOKUP

Modern, flexible lookup:

``` excel
=XLOOKUP(A1,B:B,C:C)
```

### Sort vs Filter

-   **Sort:** changes the order of records.
-   **Filter:** displays records matching criteria.

### PivotTable

Summarizes/analyzes large datasets.

Example: total sales by department or month.

### Charts

-   Column/Bar → compare categories
-   Line → trends over time
-   Pie → proportions
-   Scatter → relationship between variables

### Excel MCQ traps

-   `FALSE` in VLOOKUP = exact match
-   `$A$1` = fully absolute
-   Sort ≠ Filter
-   PivotTable = summarize/analyze
-   Line chart = trend

------------------------------------------------------------------------

## 3. Microsoft PowerPoint

**Purpose:** Create slide-based presentations.

  Concept         Meaning
  --------------- --------------------------------------
  Slide           One presentation page
  Theme           Overall visual design
  Layout          Arrangement of placeholders
  Transition      Effect **between slides**
  Animation       Effect on **objects within a slide**
  Slide Master    Controls common design/formatting
  Speaker Notes   Presenter notes

### Shortcuts

-   `Ctrl+N` New presentation
-   `Ctrl+S` Save
-   `Ctrl+M` New slide
-   `Ctrl+D` Duplicate
-   `F5` Start slideshow from beginning
-   `Shift+F5` Start from current slide

**MCQ trap:** Transition = slide-to-slide; Animation = object-level.

------------------------------------------------------------------------

## 4. Microsoft Outlook

**Purpose:** Email, calendar, meetings, contacts, and tasks.

### Email fields

-   **To:** Primary recipients
-   **CC:** Visible copy recipients
-   **BCC:** Hidden copy recipients
-   **Reply:** Reply to sender
-   **Reply All:** Reply to sender and other recipients
-   **Forward:** Send an existing email to another person

### Calendar

Used for appointments, meetings, events, and scheduling.

**Appointment:** mainly your own calendar entry.\
**Meeting:** includes invited participants.

### Rules

Automate email handling.

Example: `If sender = Manager → move email to Important folder`

**MCQ trap:** CC is visible; BCC hides recipient addresses from other
recipients.

------------------------------------------------------------------------

## 5. Microsoft Access

**Purpose:** Relational database application.

### Main objects

  Object   Purpose
  -------- ----------------------------
  Table    Stores data
  Query    Retrieves/manipulates data
  Form     User interface/data entry
  Report   Formatted output

Example query:

``` sql
SELECT * FROM Employees;
```

### Primary Key

Uniquely identifies each record.

Example: `EmployeeID`

### Foreign Key

References a key in another table.

Example: `Employee.DepartmentID → Department.DepartmentID`

### Relationships

-   **One-to-One:** one record ↔ one record
-   **One-to-Many:** one record ↔ many records
-   **Many-to-Many:** many ↔ many; usually uses a junction/bridge table

**MCQ traps:** - Table → stores data - Query → retrieves/manipulates
data - Form → data entry/interface - Report → formatted output - Primary
Key → unique identifier - Foreign Key → reference to another table

------------------------------------------------------------------------

## 6. Microsoft Publisher

**Purpose:** Desktop publishing and page-layout design.

### Common uses

-   Brochures
-   Flyers
-   Posters
-   Newsletters
-   Business cards
-   Invitations

**Publisher vs Word** - Word → document/word processing - Publisher →
visual page layout/desktop publishing

**MCQ tip:** brochure, flyer, poster, or business card → Publisher.

------------------------------------------------------------------------

# 7. Which Tool Should You Choose?

  Task                    Tool
  ----------------------- ------------
  Write report            Word
  Calculate marks         Excel
  Analyze data            Excel
  Create presentation     PowerPoint
  Send email              Outlook
  Schedule meeting        Outlook
  Store relational data   Access
  Run database query      Access
  Create brochure         Publisher
  Create business card    Publisher

------------------------------------------------------------------------

# 8. Master Shortcut Sheet

### General

``` text
Ctrl+N → New
Ctrl+O → Open
Ctrl+S → Save
Ctrl+P → Print
Ctrl+A → Select All
Ctrl+C → Copy
Ctrl+X → Cut
Ctrl+V → Paste
Ctrl+Z → Undo
Ctrl+Y → Redo
Ctrl+F → Find
```

### Word

``` text
Ctrl+B → Bold
Ctrl+I → Italic
Ctrl+U → Underline
Ctrl+L → Left
Ctrl+E → Center
Ctrl+R → Right
Ctrl+J → Justify
Ctrl+H → Replace
Ctrl+Enter → Page Break
```

### PowerPoint

``` text
Ctrl+M → New Slide
Ctrl+D → Duplicate
F5 → Slideshow
Shift+F5 → Current Slide
```

------------------------------------------------------------------------

# 9. Excel Formula Cheat Sheet

``` text
=SUM(A1:A5)                    Add
=AVERAGE(A1:A5)               Average
=MAX(A1:A5)                   Largest
=MIN(A1:A5)                   Smallest
=COUNT(A1:A5)                 Count numbers
=COUNTA(A1:A5)                Count non-empty
=COUNTBLANK(A1:A5)            Count blanks
=COUNTIF(A:A,">50")           Conditional count
=SUMIF(A:A,"Apple",B:B)       Conditional sum
=AVERAGEIF(A:A,">50")         Conditional average
=IF(A1>=40,"Pass","Fail")     Condition
=AND(A1>0,B1>0)               All conditions
=OR(A1>0,B1>0)                Any condition
=NOT(A1=10)                   Reverse result
=LEFT(A1,3)                   Left characters
=RIGHT(A1,3)                  Right characters
=MID(A1,2,3)                  Middle characters
=LEN(A1)                      Character count
=TRIM(A1)                     Remove extra spaces
=UPPER(A1)                    Uppercase
=LOWER(A1)                    Lowercase
=PROPER(A1)                   Title case
=ROUND(A1,2)                  Round number
=VLOOKUP(A1,B:D,3,FALSE)      Vertical lookup
=HLOOKUP(A1,B:D,3,FALSE)      Horizontal lookup
=XLOOKUP(A1,B:B,C:C)          Flexible lookup
```

------------------------------------------------------------------------

# 10. Last-Minute Accenture OA Revision

Memorize these pairs:

``` text
Word       → Documents
Excel      → Spreadsheets / Calculations
PowerPoint → Presentations
Outlook    → Email / Calendar
Access     → Database
Publisher  → Desktop Publishing

Page Break    → New page
Section Break → Different section formatting

Transition → Between slides
Animation  → Object on slide

COUNT   → Numeric cells
COUNTA  → Non-empty cells
COUNTBLANK → Empty cells

Sort   → Reorders data
Filter → Shows matching data

Primary Key → Unique record identifier
Foreign Key → References another table

CC  → Visible
BCC → Hidden

VLOOKUP → Vertical
HLOOKUP → Horizontal
```

## Priority Order If You Have Very Little Time

1.  Excel formulas
2.  Relative vs absolute references
3.  VLOOKUP/HLOOKUP/XLOOKUP
4.  COUNT vs COUNTA vs COUNTBLANK
5.  COUNTIF/SUMIF
6.  IF/AND/OR
7.  Sort vs Filter
8.  PivotTable and charts
9.  Word shortcuts
10. Page Break vs Section Break
11. Mail Merge
12. PowerPoint Transition vs Animation
13. Slide Master
14. Outlook To/CC/BCC
15. Outlook Rules and Calendar
16. Access Primary/Foreign Keys
17. Access relationships
18. Publisher use cases

## One-Line Memory Trick

**Word writes, Excel calculates, PowerPoint presents, Outlook
communicates, Access stores, Publisher designs.**
