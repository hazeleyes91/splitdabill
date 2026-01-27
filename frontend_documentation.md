# Frontend Structure & UX/UI Overview

## **1. Application Structure**
The application is a standard HTML/CSS/JS web app served via FastAPI templates.
- **Entry Points**:
    - **Home**: `templates/home.html` (Served at `/`)
    - **App**: `templates/index.html` (Served at `/session/{id}`)
- **Styling**: Vanilla CSS (embedded in `<style>`).
- **State Management**: Local JavaScript object `state` synced with the backend via API calls.

---

## **2. User Interface (UI) Components**
The UI is separated into two distinct pages:

1.  **Home Screen** (`home.html`):
    -   Landing page.
    -   "Create New Bill Link" button (POSTs to `/sessions` -> Redirects).

2.  **App Screen** (`index.html`):
    -   The main workspace, loaded only when a valid Session ID is present in the URL.
    -   **Loading Screen** (`#loadingRoot`): Spinner shown during initial fetch.

### **Main Workspace Sections**
| Section | ID | Description |
| :--- | :--- | :--- |
| **Header** | N/A | Contains "Bill Name" input, "New Bill" / "Copy Link" buttons, and **Mode Toggle**. <br> **Features**: <br> - **Mode Toggle**: Switches between "Basic" and "Advanced". <br> - **Bill Name**: Text input with "Bill Name" placeholder. Can be cleared (stays empty). <br> - **New Bill**: Redirects to Home to start fresh. <br> - **Copy Link**: Copies the current session URL. |
| **Bill Details** | `#s1` | Matrix table for Dishes/People inputs. **Features**: <br> - **Basic Mode**: <br> - "Paid By" dropdown in Dish cell. <br> - Toggle buttons (✔) for eating.<br> - **Advanced Mode**: <br> - Numeric +/- inputs. |
| **Payments** | `#s2` | List of payers. **Basic Mode**: HIDDEN (Auto-calculated from Sec 1). **Advanced Mode**: Visible manual entry. |
| **Covers / Treats** | `#s3` | List of covers. **Basic Mode**: HIDDEN. **Advanced Mode**: Visible. |
| **Final Results** | `#s4` | "Calculate Settlements" button and the results list showing who owes whom. |
| **Backup** | `#s5` | JSON dump/load area for debugging or manual backup. |

### **General Features**
- **Live Warning System**: 
    - Automatically checks difference between Total Bill (Section 1) and Total Paid (Section 2).
    - If difference > 0.1, displays a warning in Section 2.
    - **Blocks Calculation**: Disables the "Calculate Settlements" button until the discrepancy is resolved.
- **Input Guards**: Numeric fields (Prices, Amounts) block non-numeric keys to prevent invalid data.
- **Clear Buttons ("x")**: 
    - Custom overlay button appearing on hover inside inputs (Bill Name, Person Name, Dish Price, etc.).
    - Clears the field immediately and updates the UI.
- **Delete Buttons**: 
    - Distinct square button with faint gray background to differentiate from Clear buttons.
    - Removes the item (Person, Dish, Payment row) entirely.

---

## **3. User Experience (UX) Flow**

### **Session Management**
- **New User**: Visits `/` (Home) -> Clicks "Create New Bill Link" -> `POST /sessions` -> Redirects to `/session/{uuid}`.
- **Existing Link**: User visits `/session/{uuid}` -> App loads session data via `GET /sessions/{uuid}`.
- **Auto-Save**: Any change to data models (dishes, people, ratios, payments) triggers an `autoSave()` function (debounced 1s) to `PUT /sessions/{id}`.

### **Core Workflows**
1.  **Adding Data**:
    - Users add dishes and people in Section 1.
    - Matrix cells (`modRatio`) increment/decrement consumption.
    - **Input Validation & UX for Section 1 (Consumption)**:
        - **Dish Name**: 
            - Textarea with 2-line visual limit (27px).
            - Auto-shrinks font size (12px -> 9px) to fit text.
        - **Dish Price**: Number input with `validateNumberInput` guard.
        - **Add Dish**: Adds a new row and **auto-scrolls** the window down.
2.  **Calculations**:
    - Users enter payment info in Section 2.
    - Clicks "Calculate Settlements" in Section 4.
    - UI validates `Total Dish Cost` vs. `Total Paid`. Warnings shown if mismatch > 0.1.
3.  **Settlement**:
    - Displays a list of debts (e.g., "Alice pays Bob $50").
    - Users can add bank details/notes for creditors (synced across all debts to that creditor).

## **5. Dual Mode Logic**
The app supports two modes, stored in `state.mode` (Bill-Based state):

### **Basic Mode** (Default)
- **Philosophy**: Simple "I paid for this, you ate that".
- **Consumption**: Binary (Eat/Don't Eat) via toggle buttons.
- **Payments**: "Paid By" dropdown located directly in the Dish cell. Payments are derived solely from this.
- **Covers**: Disabled.
- **Hidden Sections**: Section 2 (Who Paid) and Section 3 (Covers).

### **Advanced Mode**
- **Philosophy**: Granular control.
- **Consumption**: Numeric ratios (e.g., 0.5, 2x portions).
- **Payments**: Manual entry in Section 2.
- **Covers**: Full support in Section 3.

### **Switching Logic**
- **Basic -> Advanced**: Safe. Preserves data.
- **Advanced -> Basic**: **Destructive**.
    - Triggers a **Custom CSS Modal** warning.
    - Resets `state.covers` to empty.
    - Recalculates payments based on Dish Payers (overwriting manual payments).

---

## **4. API Interactions**

### **Backend Endpoints Used**

| Method | Endpoint | Payload / Params | Functionality |
| :--- | :--- | :--- | :--- |
| **POST** | `/sessions` | `{ state: {...} }` | Creates a new session. Returns `{ id: "..." }`. |
| **GET** | `/sessions/{id}` | N/A | Fetch session state. Returns `{ id, state }`. |
| **PUT** | `/sessions/{id}` | `{ state: {...} }` | Updates/Saves session state. |
| **POST** | `/calculate` | `{ section1: {...}, payments: [...], covers: [...] }` | Performs split logic. Returns `{ settlements: [...] }`. |
