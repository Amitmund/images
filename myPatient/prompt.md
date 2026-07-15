# Prompt for mypacent website.

I want to build a Progressive Web App with Django for doctors for to maintain the details about patient. In this case what are the features should be having? what would you recommond for doctors sinup, payments approval plan for now, manual where we can approve for 7 days, 3 months and 1 year active plan, which should have start and end date prefilled, after the end date. 7 days before it should prompt the docor for the payment, if not payment done till last day, just deactivate on login, but all the record should be kept. 

And what are the features we should be having for patient features.

Features:

Pagination.

Slug

Secutiry check on special character and binary payload protectation.

(Safty on image upload, or should we just have google drive share image upload link or https://github.com/SH20RAJ/picser


```
Can I have a django custom store, where user will get the option to upload a image, and internally, it should upload the image in my github(following setting) and update the database with the jsdelivr_commit link. For end user they have feel like uploading the image, but internally, it updated the jsdelivr link, which will use in rest of the app. Internally I want to have my own hosted " https://picser.pages.dev/api/public-upload" or 

# Replace with your actual GitHub credentials
curl -X POST \
  -F "file=@screenshot.png" \
  -F "github_token=ghp_your_github_token" \
  -F "github_owner=your_username" \
  -F "github_repo=your_repo" \
  https://picser.pages.dev/api/public-upload


Response (JSON):

{
  "success": true,
  "filename": "screenshot-1704123456789.png",
  "url": "https://cdn.jsdelivr.net/gh/user/repo@abc123/uploads/screenshot-1704123456789.png",
  "urls": {
    "jsdelivr_commit": "https://cdn.jsdelivr.net/gh/user/repo@abc123/uploads/screenshot-1704123456789.png",
    "raw_commit": "https://raw.githubusercontent.com/user/repo/abc123/uploads/screenshot-1704123456789.png",
    "github_commit": "https://github.com/user/repo/blob/abc123/uploads/screenshot-1704123456789.png"
  },
  "size": 142857,
  "type": "image/png"
}
```

Followup questions:
- After the doctor signin, we need to give a open a page, where we will be having a youtube video link, how to use.
- Also in 7 days free trial, the date should also calcualte the last day of expire login. ( and till that the PWA starts should be showing...)
- For payment, contact details of owner of the website. And once the payment is done, on the Admin, panel, just to have enable just by selecting 3 months and 1 year activation options (With correction options for admin) and once its selected the date plan, its should auto update new expire date.
- Should we keep the payment record details (fields ) just for record, and where we should keep these records.

---


# Core Doctor Management & Access Flow

For a MVP (Minimum Viable Product), a manual, admin-controlled activation workflow keeps things lightweight while ensuring you retain full control over access.

### 1. Sign-Up & Verification Workflow
Registration: Doctors sign up using basic professional details (Name, Medical License Number/Registration ID, Clinic/Hospital Name, Email, and Phone).

Pending State: Upon registration, the doctor's account is created in a "Pending Verification" state. They cannot access the patient dashboard yet.

Admin Notification: You (the system admin) receive an alert to verify their credentials.

Date Filter: (If doctor select a given date, it should display all the pacent name, contact_number and its dignostic type, having a link to the pacent name, opening to the pacent details. This list also include the pacent came on followup date with its note too. (The operator need to write note, if the paceent came on the followup date.)


Q) Doctor, key filter what they are looking (By date, By name and By dignostic) 


### 2. Manual Access & Subscription Approval
Instead of integrating a complex automated payment gateway on day one, you will use an Admin Panel to manually manage access tiers:

### Plan Tiers:

```

7-Day Trial

3-Month Access

1-Year Access

```

Prefilled Validation: When you approve a plan, the system automatically calculates the Start Date (Current Timestamp) and End Date based on the selected tier.

### Grace Period & Prompts:

T-Minus 7 Days: The PWA starts showing a non-intrusive warning banner upon login (e.g., "Your access expires in X days. Please contact support to renew.").

Expiration Day: If no manual renewal is recorded by the end date, the system flips the account status to "Expired".

### Soft Deactivation (Data Retention):

When an expired doctor attempts to log in, they are blocked by a screen stating their account is inactive and directing them to payment/renewal support.

Crucial: Their database records, patient histories, and uploaded documents remain completely untouched and securely encrypted. Nothing is deleted.


---

# Core Features for the Doctor Portal

Since this is a PWA, these features should be optimized for quick mobile/tablet use while doctors are moving between patients.

Offline Patient Search & View: Because it's a PWA, doctors should be able to view cached patient list summaries and basic contact info even if the hospital Wi-Fi drops out temporarily.

Patient Profiles & Demographics: Standard digital charts capturing name, age, blood group, allergies, emergency contacts, and insurance details.


### Electronic Health Records (EHR) & Encounters:

Timeline view of past visits/consultations.

Subjective notes (symptoms), Objective findings JSON field, so we can add new fields: (Optional) (vitals like BP, heart rate, weight), Assessment, and Plan (SOAP notes format). (What is SOAP format mean? )


Digital Prescriptions: A quick interface to add medications, dosages, frequencies, and duration, which can be generated into a downloadable or shareable summary.

Lab & Report Uploads: The ability to snap a photo using a phone camera (via the PWA) or upload PDFs of lab results directly into the patient's file.

Appointment Scheduler: A simple calendar view showing daily or weekly patient slots.



---

# Core Features for the Patient Portal




