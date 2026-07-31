# Metro Ticket Booking & Smart Card Recharge System

A ServiceNow Service Catalog project that allows passengers to either book a QR-based metro ticket or recharge their smart travel card — all through a single catalog item with dynamic, branching form logic.

---

## 🔗 Project Links

- **Demo Video:** https://www.image2url.com/r2/default/videos/1785522313620-a26240dc-1d70-4dc7-820b-b587d65a8b34.mp4
- **Project Documentation (PDF/DOCX):** [(https://drive.google.com/drive/u/0/folders/1JFnw9kAjuz5oT_pmoLIhdYS6gfS84dcw)]
- **ServiceNow PDI Instance:** `(https://dev386209.service-now.com/)`

---

## 📌 Project Title

Metro Ticket Booking & Smart Card Recharge System

## 👤 Submitted By

- **Name:** Shaik Zuveriya Tabassum
- **Role:** ServiceNow Developer (Individual Project) — responsible for end-to-end design, catalog build, scripting, testing, and documentation.

---

## 🎯 Purpose

This project implements a metro e-ticketing system as a ServiceNow Service Catalog item. It replicates the real-world dual-service counter experience — buy a ticket vs. top up a travel card — inside the ServiceNow Service Portal, allowing passengers to:

- Book a QR-based metro ticket for a single or return journey, **or**
- Recharge an existing metro smart card

---

## ✨ Features

- Single catalog item offering two branching services: **Book a Metro Ticket** and **Recharge Smart Card**
- Dynamic form fields shown/hidden based on selected service, using UI Policies
- Journey details capture: Starting Station, Destination, Number of Passengers, Journey Type (Single/Return)
- Automatic fare calculation based on passengers and journey type via a Client Script
- Multiple payment modes (Card, Cash, UPI, Others) with conditional free-text field for "Others"
- Smart card recharge capture: Card Number, Card Name, Recharge Amount
- QR code generation for the recharged smart card, displayed to the user on submission
- Automatic Requested Item (RITM) and Request (REQ) generation with full request tracking
- Additional Details view on the Requested Item form showing all submitted values

---

## 🏗️ Architecture

### Platform
Built entirely on the ServiceNow platform using native Service Catalog and Service Portal capabilities — no external frontend/backend stack is used.

### Service Portal (Frontend)
- Service Portal (`/sp`) used as the end-user facing interface
- Catalog item rendered via the standard Service Portal "Catalog Item" widget
- Catalog Client Scripts (onChange, onSubmit) used for dynamic behavior in the browser
- UI Policies used to show/hide and mandate fields based on user selection

### Platform Layer (Backend)
- **Catalog Item:** `sc_cat_item` — "Book A Metro Ticket"
- **Variables:** `sc_item_option` / `item_option_new` — capture journey, passenger, and recharge details
- **Requested Item / Request:** `sc_req_item`, `sc_request` — auto-generated on order submission
- **Question Choices:** define the two branching service options (`recharge_card` / `book_ticket`)

### Database
Uses ServiceNow's native platform tables (`sc_cat_item`, `sc_req_item`, `sc_request`, `sc_item_option_mst`, `sc_item_option_mst_value`) — no external database required.

### Third-Party Integration
- **QR Server API** (`api.qrserver.com`) — generates the QR code image for the recharged smart card, called directly from the client script.

---

## ⚙️ Setup Instructions

### Prerequisites
- A ServiceNow Personal Developer Instance (PDI) — Australia data residency instance used for this project
- Admin (or `catalog_admin`) role access
- Access to **Service Catalog > Catalog Definitions** module

### Installation / Configuration Steps
1. Log in to the ServiceNow PDI as System Administrator.
2. Copy the instance domain, e.g. `https://dev190678.service-now.com`
3. Append `/sp` to the URL to access the Service Portal, e.g. `https://dev190678.service-now.com/sp`
4. Navigate to **Service Catalog > Catalog Definitions > Maintain Items > New** to create the catalog item "Book A Metro Ticket."
5. Add variables, UI Policies, and Catalog Client Scripts as detailed below.

---

## 🗂️ Catalog Structure

### Catalog Item
- **Book A Metro Ticket** — Category: Office > Services

### Variables (12 total)
- `what_do_you_want_to_do_today` (Multiple Choice) — branching control
- `starting_from`, `going_to` (Reference/String)
- `no_of_passengers` (Integer)
- `type_of_journey` (Choice: Single Journey / Return Journey)
- `amount_for_single_journey`, `amount_including_return`
- `mode_of_payment` (Choice: Card / Cash / Others / UPI)
- `enter_payment_mode` (String, conditional on "Others")
- `enter_smart_card_number`, `enter_smart_card_name`, `recharge_amount`

### Catalog Client Scripts (3 total)
| Name | Type | Purpose |
|---|---|---|
| FareCalculation | onChange | Auto-calculates fare based on passengers/journey type |
| FieldValidation | onChange | Validates form input |
| QR Generation | onSubmit | Generates and displays the QR code for smart card recharge |

### UI Policies
- Show ticket-booking fields when "Book a Metro Ticket" is selected
- Show recharge fields when "Recharge Smart Card" is selected
- Show "Amount Including Return" when Journey Type = Return
- Show "Enter payment mode" when Mode of Payment = Others

---

## ▶️ Running the Application

As a ServiceNow catalog item, there is no separate frontend/backend server to start.

1. Open the Service Portal:
   ```
   https://<your-instance>.service-now.com/sp
   ```
2. Search for **"Book A Metro Ticket"** in the catalog search bar.
3. Fill in the required details and click **Order Now** to submit.

---

## 🔌 API Documentation

### External API Used — QR Server API
Generates a scannable QR code image representing the recharged smart card.

```
GET https://api.qrserver.com/v1/create-qr-code/
    ?size=250x250
    &data=<encoded card number, amount, and reference ID>
```

Called from the **QR Generation** Catalog Client Script (`onSubmit`) when the user selects Recharge Smart Card.

### Internal Platform APIs Used
| Function | Description |
|---|---|
| `g_form.getValue()` | Reads the current value of a catalog variable |
| `g_form.setValue()` | Sets a catalog variable value (used for auto fare calculation) |
| `g_form.getUniqueValue()` | Retrieves the RITM sys_id used as a QR reference number |

---

## 🔐 Authentication

Handled natively by the ServiceNow platform:
- Users authenticate via standard ServiceNow login (username/password or SSO, depending on instance configuration)
- Service Portal session maintained via ServiceNow's native session cookies
- Catalog item visibility and submission rights controlled through the item's Availability and Permission tabs (roles/ACLs)
- No custom token-based authentication implemented — the entire solution runs within the authenticated ServiceNow session

---

## 🧪 Testing

### Testing Strategy
Manual functional testing performed directly on the ServiceNow PDI Service Portal, covering both branches of the catalog item.

### Test Cases Executed
| # | Test Case | Result |
|---|---|---|
| 1 | Book a Metro Ticket — Single Journey fare calculation | Pass |
| 2 | Book a Metro Ticket — Return Journey fare calculation | Pass |
| 3 | Payment Mode = Others reveals free-text field | Pass |
| 4 | Order submission generates RITM with correct Additional Details | Pass |
| 5 | Recharge Smart Card — UI Policy hides ticket fields | Fail → Fixed |
| 6 | QR code generation on Recharge Smart Card submission | Fail → Fixed |

### Tools Used
- Browser DevTools (Console & Elements tabs) — used to diagnose the GlideModal reference error
- ServiceNow PDI native testing — direct Service Portal order submission

---

## 🐛 Known Issues

### Resolved During Development
- **UI Policy mismatch:** "Recharge Smart Card" did not hide ticket-booking fields — root cause was a mismatch between the UI Policy condition and the actual Question Choice Value; corrected by verifying the exact Value column in Question Choices.
- **QR popup "Page not found":** Root cause was use of `GlideModal` referencing a non-existent UI Page (`external_qr_api_page`) — this API is Classic UI-only and not available in Service Portal.
- **`ReferenceError: GlideModal is not defined`:** Resolved by replacing GlideModal with a native `window.open()` popup rendering the QR Server API image directly.

### Outstanding
- Browser popup blockers may prevent the QR window from opening on first attempt — requires the user to allow popups for the instance domain.
- QR Server API is an external third-party domain; outbound access must be confirmed on restricted/enterprise instances.

---

## 🚀 Future Enhancements

- Replace the third-party QR Server API with an in-platform QR generation library to remove the external dependency
- Display the QR code inline within the Requested Item's Additional Details view (via a Service Portal widget) instead of a separate popup window
- Add Flow Designer automation to route Recharge Smart Card requests to a fulfillment/approval flow distinct from ticket bookings
- Add mandatory attachment validation (e.g., payment proof) only when Mode of Payment is UPI or Others
- Add real fare-matrix lookup (via a Data Lookup Definition) instead of a flat per-passenger rate

---

## 📁 Repository / File Contents

| File | Description |
|---|---|
| `Metro_Ticket_Project_Documentation.pdf` | Full project documentation with screenshots |
| `Metro_Ticket_Project_Documentation.docx` | Editable version of the documentation |
| `README.md` | This file |

---

## 📄 License

This project was built for academic/training purposes as part of a ServiceNow development learning project.
