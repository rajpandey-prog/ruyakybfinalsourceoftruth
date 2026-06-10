# Ruya Business Banking KYB V0 - API Field Source of Truth

**Scope:** Existing Nymcard customers onboarding to Alaan Business Account powered by Ruya.  
**Version:** V0 / Phase 1 MVP.  
**Primary principle:** Keep the customer form lightweight. Pre-fill what exists in Nymcard, ask only for the minimum additional V0 inputs, and let Ops complete deeper Ruya enrichment before final submission to Ruya.

---

## 1. Collection routes

| Collection route | Meaning | V0 handling |
|---|---|---|
| Pre-fill from Nymcard | Data or document metadata already exists from the customer's Nymcard KYB application. | Show fields as editable where useful. Show carried-over documents as metadata only. |
| Ask user for input | Incremental data needed from the customer in V0. | Ask in the customer form using plain language and non-blocking required flags. |
| System mapping | Backend maps user-friendly values to Ruya UUID/master data. | User should never see UUIDs. Engineering maps dropdowns/derived values to Ruya master data. |
| Handle post-submission via Ops | Required by Ruya, but not suitable for the V0 customer form. | Ops completes in Retool/back office using Nymcard records, uploaded docs, or follow-up if needed. |

---

## 2. V0 form decisions

- Required fields are shown as **Required** flags only. They do not block step navigation in the customer form.
- The user can submit the V0 form with missing items. Missing Ruya-required items are resolved by Ops before final Ruya submission.
- Nymcard documents are shown as **metadata only**: document type, holder/company name, expiry date, and status. No preview or download.
- Newly uploaded Ruya documents are previewable and can be deleted/re-uploaded by the initiator and Organisation Admins.
- If a Nymcard document is replaced, both versions are retained. Nymcard remains metadata-only, while the new Ruya upload becomes the current source of truth.
- Near-expiry documents show alerts/comments but do not force renewal in V0. Expired documents must be replaced before final processing.
- Emirates ID and Passport are exclusive identity paths.
- The customer is not asked for ownership percentage in V0. Use Nymcard first, then Ops if missing.

---

## 3. Field matrix by section

### 3.1 Initiator details

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| firstName | String | Mandatory | Always | Pre-fill from Nymcard | Show editable first name. |
| middleName | String | Optional | Optional if available | Pre-fill from Nymcard if available / Ops | Do not ask unless already present or needed by Ops. |
| lastName | String | Mandatory | Always | Pre-fill from Nymcard | Show editable last name. |
| mobileNumber | String | Mandatory | Always | Pre-fill from Nymcard; ask user to confirm/edit | Phone input. Format: UAE mobile starting with 5, 9 digits. |
| email | String | Mandatory | Always | Pre-fill from Nymcard; ask user to confirm/edit | Email input. |
| emiratesId | String | Conditionally mandatory | Initiator uses Emirates ID identity path | Ask user for input | Ask if user is UAE citizen/resident. If Emirates ID path, collect Emirates ID. |
| emiratesIdExpiryDate | Date | Optional in Ruya; V0 required if Emirates ID path | Initiator uses Emirates ID identity path | Ask user for input | Ask Emirates ID expiry date. Flag if expired. |
| roleId | Enum | Mandatory | Always | Ask user for input | Relationship dropdown: Authorised Signatory, Shareholder, Employee. Map to Ruya initiator role/relationship values. |
| companyName | String | Mandatory | Always; must match Company Details companyName | Pre-fill from Nymcard | Show editable company name. |
| countryId | UUID | Optional | If Ruya payload requires initiator country | System mapping / Ops | Do not ask in V0 unless needed by Ops. |

**V0 relationship logic**

| Relationship selected | Follow-up logic | Consent handling |
|---|---|---|
| Authorised Signatory | No POA question. | Collect consent checkbox inside the form. |
| Shareholder | Ask: “Do you have a Power of Attorney from the company?” | If Yes, ask for POA document. If No, collect Authorised Signatory details and route consent. |
| Employee | Ask: “Do you have a Power of Attorney from the company?” | If Yes, ask for POA document. If No, collect Authorised Signatory details and route consent. |

---

### 3.2 Company details

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| companyName | String | Mandatory | Always; must match Initiator companyName | Pre-fill from Nymcard | Show editable company name. |
| tradeLicenseNumber | String | Mandatory | Always | Pre-fill from Nymcard | Show editable trade licence number. |
| tradeLicenseIssueDate | Date | Mandatory | Always | Pre-fill from Nymcard if available; otherwise Ops | Do not block user. Ops completes if missing. |
| tradeLicenseExpiryDate | Date | Mandatory | Always | Pre-fill from Nymcard | Show expiry status. Near-expiry warning only; expired must be replaced before final processing. |
| issuingAuthorityId | UUID | Mandatory | Always | System mapping from Nymcard trade licence data; Ops if unmapped | Do not ask user unless mapping fails. |
| dateOfIncorporation | Date | Mandatory | Always | Pre-fill from Nymcard if available; otherwise Ops | Do not ask in V0 unless Ops needs correction. |
| legalTypeId | UUID | Mandatory | Always | System mapping from Nymcard/legal docs; Ops if missing | Do not ask customer in V0. Drives document list. |
| industryId | UUID | Mandatory | Always | System/Ops mapping | Map from licence activity and activity-list answer. Do not ask technical industry dropdown. |
| subIndustryId | UUID | Mandatory | Always | System/Ops mapping | Must belong to selected industry. |
| customerSegmentId | UUID | Mandatory | Always | System mapping | Derive from turnover/segment rules. Ops can override. |
| cbSectorId | UUID | Mandatory | Always | System/Ops mapping | Map from activity/industry. |
| cbSubSectorId | UUID | Mandatory | Always | System/Ops mapping | Must belong to selected CB sector. |
| emirateOfRegistrationId | UUID | Mandatory | Always | System mapping from trade licence / Nymcard; Ops if missing | Do not ask customer unless unavailable. |
| companyProfile | String | Mandatory | Always | Ask user if not available; otherwise Ops drafts | Simple prompt: “Describe what your company does.” Max 4000 chars. |
| licensingActivity | String | Optional | Optional but useful for mapping | Pre-fill from Nymcard if available; ask via activity-list review | Show plain-language business activity list, not technical code field. |
| companyCategoryId | UUID | Optional | Optional | System/Ops | Do not ask in V0. |
| molEstablishmentId | String | Optional | Optional | Ops | Do not ask in V0. |
| dunAndBradstreetNumber | String | Optional | Optional | Ops | Do not ask in V0. |
| dulNumber | String | Optional | Optional | Ops | Do not ask in V0. |
| referralCode | String | Optional | Optional | System/internal | Do not ask in V0. |

---

### 3.3 VAT path

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| vatRegistrationNumber | String | Conditionally mandatory | Required if vatMissingReason is empty / company is VAT registered | Pre-fill from Nymcard/TRN if available; Ops otherwise | V0 does not show VAT block by default. Ask only if Ops needs user follow-up. |
| vatRegistrationDate | Date | Conditionally mandatory | Required if vatMissingReason is empty / company is VAT registered | Ops unless available | Do not ask in V0 by default. |
| vatMissingReason | String | Conditionally mandatory | Required if VAT registration number/date are not provided | Ops; ask user only if needed | Either VAT number + date or missing reason must be ready before Ruya submission. |

---

### 3.4 Business details

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| sourceOfIncomeId | UUID | Mandatory | Always | System/Ops mapping | Map from business activity/company profile. Do not ask in V0. |
| currentTurnover | String | Mandatory | Always | Pre-fill from Nymcard if available; ask only if missing | Show turnover as editable if present. |
| currentTurnoverRangeId | UUID | Mandatory | Always | System mapping | Derive from currentTurnover / Nymcard turnover band. |
| annualIncome | String | Mandatory | Always | System/Ops | Derive from turnover or complete in Ops. |
| avgExpectedTransactionId | UUID | Mandatory | Always | Ops/system | Do not ask in V0 unless later required. |
| expectedMonthlyTransactionsId | UUID | Mandatory | Always | Ops/system | Do not ask in V0 unless later required. |
| expectedMonthlyInwardTransfers | String | Mandatory | Always | Ops | Not in V0 customer form. Ops fills or follows up. |
| expectedMonthlyOutwardTransfers | String | Mandatory | Always | Ops | Not in V0 customer form. Ops fills or follows up. |
| natureOfBusinessId | UUID | Mandatory | Always | System/Ops mapping | Map from business activity. |
| numberOfEmployees | String | Mandatory | Always | Pre-fill from Nymcard if available; Ops if missing | Do not ask in V0 unless unavailable and required by Ops. |
| ownershipStructureId | UUID | Mandatory | Always | System/Ops mapping | Map from Nymcard/shareholder data and legal docs. |
| walkInCustomer | Boolean | Optional | Optional | System/Ops | Do not ask in V0. |
| countryNames | String | Optional | Optional | Ops/system | Derive from business partners/known operations if needed. |
| otherBankAccountTypeId | UUID | Optional | Optional | Ops | Do not ask in V0. |

#### Transaction percentage breakdown

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| cashDepositPercentage | Number | Mandatory | Always for Ruya Business Details | Ops | Not asked in V0. Ops completes before Ruya submission. |
| chequeDepositPercentage | Number | Mandatory | Always for Ruya Business Details | Ops | Not asked in V0. Ops completes before Ruya submission. |
| bankTransferPercentage | Number | Mandatory | Always for Ruya Business Details | Ops | Not asked in V0. Ops completes before Ruya submission. |
| exportLcPercentage | Number | Mandatory | Always for Ruya Business Details | Ops | Not asked in V0. Ops completes before Ruya submission. |
| othersTransactionPercentage | Number | Mandatory | Always for Ruya Business Details | Ops | Not asked in V0. Ops completes before Ruya submission. |

Rule: all five percentages must total exactly 100 before Ruya submission.

#### Business relationship fallback reasons

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| reasonNotDealingSupplierId | UUID | Conditionally mandatory | Required if no suppliers are added | Ops | Not asked in V0 unless Ops needs customer follow-up. |
| reasonNotDealingCustomerId | UUID | Conditionally mandatory | Required if no customers are added | Ops | Not asked in V0 unless Ops needs customer follow-up. |
| reasonNotDealingBankId | UUID | Conditionally mandatory | Required if no other bank accounts are added | Ops | Not asked in V0 unless Ops needs customer follow-up. |

---

### 3.5 Address

V0 asks only whether the operating address is the same as the address on the trade licence. If it is different, collect the operating address. Ruya supports up to three address records.

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| addressType | Enum | Mandatory | Always for each address record | System mapping | Default to COMMUNICATION for V0 customer-entered address unless Ops adds more records. |
| addressLine1 | String | Mandatory | Always for each address record | Pre-fill from Nymcard if available; ask if different/missing | Ask only if operating address differs or Nymcard address missing. |
| addressLine2 | String | Mandatory per Ruya address table | Always for each address record | Ask if different/missing; Ops if not needed by UX | Can be optional-looking in UI but Ops completes if Ruya requires. |
| emirateId | UUID | Mandatory | Always for UAE address | System mapping from selected Emirate | Dropdown mapped to EMIRATES. |
| cityId | UUID | Mandatory | Always | System mapping from selected City | City dropdown mapped to CITIES and filtered by Emirate where possible. |
| poBox | String | Mandatory | Always | Ask if different/missing; Ops if unavailable | Max 6 chars for address record. |
| nearestLandmark | String | Mandatory | Always | Ask if different/missing; Ops if unavailable | Keep user copy simple: “Nearest landmark.” |
| primaryMobileNumber | String | Mandatory | Always | Pre-fill from Nymcard / ask if missing | UAE mobile format. |
| secondaryMobileNumber | String | Optional | Optional; can default to primaryMobileNumber | System default / Ops | Do not ask in V0. |
| primaryEmail | String | Mandatory | Always | Pre-fill from Nymcard / ask if missing | Use corporate email where appropriate. |
| secondaryEmail | String | Optional | Optional; can default to primaryEmail | System default / Ops | Do not ask in V0. |

---

### 3.6 Officials and shareholders - core fields

For V0, the prototype asks the user whether any listed activities apply. If yes, collect/shareholders with **10% or more**. If no, collect/shareholders with **25% or more**. The form does not expose “high-risk” wording to the user.

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| relationshipType | Enum | Mandatory | Always for each official | Pre-fill from Nymcard; ask/edit if missing | Show role as plain label: Director, Shareholder, Authorised Signatory, etc. Map to Ruya relationship types. |
| shareholderType | Enum | Conditionally mandatory | Required when relationshipType = SHAREHOLDER | Pre-fill from Nymcard; Ops if missing | V0 supports individual/company shareholder distinction if already known. |
| ownershipPercentage | Number | Conditionally mandatory | Required when relationshipType = SHAREHOLDER | Pre-fill from Nymcard; Ops if missing | Do not ask customer in V0. Used to decide threshold eligibility. |
| parentOfficialId | UUID | Optional | Only allowed for sub-shareholders under a company shareholder | Ops | Out of V0 customer form. |
| emirateId | UUID | Mandatory | Always for official address where applicable | Pre-fill from Nymcard if available; Ops/user if missing | Map to EMIRATES. |
| cityId | UUID | Mandatory | Always for official address where applicable | Ask/Ops if missing | Map to CITIES. |
| poBox | String | Mandatory | Always for official address where applicable | Ask/Ops if missing | Max 50 chars. |
| nearestLandmark | String | Mandatory | Always for official address where applicable | Ask/Ops if missing | Keep as simple address helper. |
| addressLine1 | String | Optional in Ruya table | Optional | Pre-fill from Nymcard if available | Show editable residential address if already present. |
| addressLine2 | String | Optional in Ruya table | Optional | Pre-fill from Nymcard if available | Do not force user. |

---

### 3.7 Individual official / shareholder fields

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| firstName | String | Mandatory | Individual official/shareholder | Pre-fill from Nymcard; ask if adding new person | Editable. |
| middleName | String | Optional | Optional | Pre-fill if available / Ops | Do not force in V0. |
| lastName | String | Mandatory | Individual official/shareholder | Pre-fill from Nymcard; ask if adding new person | Editable. |
| dateOfBirth | Date | Mandatory | Individual official/shareholder | Pre-fill from Nymcard if available; Ops/user follow-up if missing | Not asked upfront in V0 unless adding a new person. Must be 18+. |
| countryOfBirthId | UUID | Mandatory | Individual official/shareholder | Ops/user follow-up | Do not ask in V0 unless missing and needed. |
| nationalityId | UUID | Mandatory | Individual official/shareholder | Pre-fill from Nymcard if available; Ops/user follow-up | Map to NATIONALITIES. |
| residentialStatusId | UUID | Mandatory | Individual official/shareholder | Ask user via simple UAE resident/citizen question if needed | Drives Emirates ID vs Passport. |
| mobileNumber | String | Mandatory | Individual official/shareholder | Pre-fill from Nymcard; ask if missing/new person | UAE mobile format where applicable. |
| email | String | Mandatory | Individual official/shareholder | Pre-fill from Nymcard; ask if missing/new person | Valid email. |
| isUsPassportOrGreenCardHolder | Boolean | Optional | Optional | Ops/user follow-up if CRS/FATCA review requires | Do not ask in V0 by default. |
| hasOtherCitizenship | Boolean | Mandatory | Individual official/shareholder | Ops/user follow-up if needed | Not in V0 lightweight form unless required by Ops. |
| taxResidencyCountryId | UUID | Conditionally mandatory | Required when hasOtherCitizenship = true | Ops/user follow-up | Map to COUNTRIES. |
| hasTin | Boolean | Mandatory | Individual official/shareholder | Ops/user follow-up | Not in V0 lightweight form unless required by Ops. |
| tinNumber | String | Conditionally mandatory | Required when hasTin = true | Ops/user follow-up | Exactly 15 alphanumeric characters. |
| tinUnavailableReasonId | UUID | Conditionally mandatory | Required when hasTin = false | Ops/user follow-up | Map to REASON_TIN_NOT_AVAILABLE. |
| multipleUsersRequired | Boolean | Optional | Admin/internet banking only | Product/Ops | Out of Ruya KYB V0 customer form. |
| loginId | String | Optional | Admin/internet banking only | Product/Ops | Out of Ruya KYB V0 customer form. |

---

### 3.8 Identity document fields

| Residential status / path | Required API fields | Ruya status | V0 source plan | How we get it in V0 |
|---|---|---|---|---|
| Resident / UAE National / GCC National | emiratesId, emiratesIdIssueDate, emiratesIdExpiryDate | Mandatory for this identity path | Pre-fill from Nymcard metadata if available; ask/upload if missing or expired | Emirates ID and Passport are exclusive. User selects one path. |
| Non-Resident | passportNumber, passportIssueDate, passportExpiryDate | Mandatory for this identity path | Pre-fill from Nymcard metadata if available; ask/upload if missing or expired | Passport path only. |

---

### 3.9 Company shareholder fields

Applies only when `shareholderType = COMPANY`.

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| name | String | Mandatory | Company shareholder exists | Pre-fill from Nymcard if available; Ops otherwise | Do not ask customer unless adding a company shareholder. |
| companyCountryId | UUID | Mandatory | Company shareholder exists | Ops/user follow-up | Map to COUNTRIES. |
| companyTypeId | UUID | Mandatory | Company shareholder exists | Ops/system | Map to COMPANY_TYPES. |
| companyStartDate | Date | Mandatory | Company shareholder exists | Ops/user follow-up | YYYY-MM-DD. |
| tradeLicenseNumber | String | Mandatory | Company shareholder exists | Pre-fill from Nymcard if available; Ops/user otherwise | Max 100 chars. |
| tradeLicenseIssueDate | Date | Mandatory | Company shareholder exists | Ops | Must be before today. |
| tradeLicenseExpiryDate | Date | Mandatory | Company shareholder exists | Ops/user if missing or expired | Must be at least 3 months from today before Ruya submission. |
| annualTurnOverInAed | Number | Mandatory | Company shareholder exists | Ops/user follow-up | Minimum 0. |
| noOfEmployees | Number | Mandatory | Company shareholder exists | Ops/user follow-up | Minimum 0. |
| emirateId | UUID | Mandatory | Company shareholder exists and UAE address applies | Ops/system | Map to EMIRATES. |
| cityId | UUID | Mandatory | Company shareholder exists | Ops/system | Map to CITIES. |
| poBox | String | Mandatory | Company shareholder exists | Ops/user follow-up | Max 50 chars. |
| nearestLandmark | String | Mandatory | Company shareholder exists | Ops/user follow-up | Max 100 chars. |

---

### 3.10 Business partners

Business partners are not part of the V0 customer-facing form. Ops completes these before Ruya submission or follows up if needed.

#### Suppliers

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| companyName | String | Mandatory | Supplier record is added | Ops | Not asked in V0. |
| countryId | UUID | Mandatory | Supplier record is added | Ops | Map to COUNTRIES. |
| percentage | Number | Mandatory | Supplier record is added | Ops | Total supplier percentages must not exceed 100. |

#### Customers

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| companyName | String | Mandatory | Customer record is added | Ops | Not asked in V0. |
| countryId | UUID | Mandatory | Customer record is added | Ops | Map to COUNTRIES. |
| percentage | Number | Mandatory | Customer record is added | Ops | Total customer percentages must not exceed 100. |

#### Other bank accounts

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| bankCodeId | UUID | Mandatory | Other bank account record is added | Ops | Map to BANK_CODES. |
| bankAccountTypeId | UUID | Mandatory | Other bank account record is added | Ops | Map to BANK_ACCOUNT_TYPES. |
| iban | String | Optional | Optional | Ops/user follow-up if needed | Max 23 chars. |
| bankAccountSince | String | Optional | Optional | Ops/user follow-up if needed | YYYY-MM-DD. |

---

### 3.11 Consent fields

| API field | Type | Ruya status | Conditionally required when | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| consentMasterId | UUID | Conditionally mandatory | Provide either consentMasterId or consentMasterIds | System mapping | Single consent ID when one consent is captured. |
| consentMasterIds | UUID[] | Conditionally mandatory | Provide either consentMasterId or consentMasterIds | System mapping | Batch consent IDs when multiple consents apply. |

---

## 4. Consolidated document section

### 4.1 Document API fields

Every document uploaded to Ruya requires these API fields.

| API field | Type | Ruya status | When required | V0 source plan | How we get it in V0 |
|---|---:|---|---|---|---|
| documentMasterId | UUID | Mandatory | Every uploaded document | System mapping | Map selected document type to Ruya Document Master. |
| fileIntentId | UUID | Mandatory | Every uploaded document | System generated | Generated from presigned S3 upload response. |

### 4.2 Document handling rules

| Rule | V0 decision |
|---|---|
| Who can view uploaded documents | Initiator and Organisation Admins can view Ruya-uploaded documents. Nymcard documents are metadata-only for everyone. |
| Existing Nymcard documents | Show document type, holder/company name, expiry date, and status. No preview/download. |
| Updating existing documents | User updates via Documents & Certificates. Keep old Nymcard metadata; add new Ruya upload as current source of truth. |
| New Ruya uploads | User can upload, view, delete, and re-upload. |
| Near-expiry documents | Show alert/comment. Do not force update in V0. |
| Expired documents | Flag for replacement before final processing/Ruya submission. |
| Requirement types | Use Ruya Document Master: Mandatory, Optional, Conditional, Not Applicable. |

### 4.3 Shared documents across legal entity types

| Document | Ruya status | Applies to | Condition / when needed | V0 source plan |
|---|---|---|---|---|
| Passport Copy | Mandatory | All legal entity types | Required for relevant individuals, especially non-residents. | Pre-fill metadata from Nymcard if available; ask upload if missing/expired. |
| Emirates ID Copy | Mandatory | All legal entity types | Required for UAE/GCC/resident identity path. | Pre-fill metadata from Nymcard if available; ask upload if missing/expired. |
| Valid Trade License - Company Shareholder | Optional | All legal entity types | Needed when shareholder is a company. | Nymcard metadata if available; ask/Ops if company shareholder exists. |
| Additional Form | Optional | All legal entity types | If Ruya/Ops requests extra information. | Ops-driven; ask user only if requested. |

### 4.4 Entity-specific documents

| Document | Ruya status | Applies to / legal entity codes | Condition / when needed | V0 source plan |
|---|---|---|---|---|
| Valid Trade License | Mandatory | Freelancer 16, Mainland LLC 48, Simple Partnership 50, and most operating entities | Always required for operating company. | Pre-fill Nymcard metadata; ask update if expired. |
| Trade License & Certificate of Incorporation issued by Free Zone Authority | Mandatory | FZE 44, Free Zone LLC 47 | Required for free zone entities. | Pre-fill Nymcard metadata if available; ask/Ops if missing. |
| Trade License / Certificate of Incorporation | Mandatory | Offshore 49 | Required for offshore company. | Ask/Ops if not in Nymcard. |
| Valid trade/manufacturing/commercial/professional license | Mandatory | Private Joint Stock 51 | Required by legal entity type. | Ask/Ops if applicable. |
| Valid Trade / Commercial License | Mandatory | Public Joint Stock 52 | Required by legal entity type. | Ask/Ops if applicable. |
| Commercial Registration Certificate | Optional | Freelancer 16, Mainland LLC 48, Simple Partnership 50 | Optional by Ruya document master. | Nymcard if available; ask only if Ops requests. |
| Certificate of Incorporation | Mandatory | FZE 44, Private Joint Stock 51, Public Joint Stock 52; also some free zone/offshore cases | Required by legal entity type. | Nymcard if available; otherwise ask/Ops. |
| Certificate of incorporation of parent company | Mandatory | Branch of Foreign Company 40 | Required for branch parent structure. | Ask/Ops when parent organisation applies. |
| Valid License to operate in UAE | Mandatory | Branch of Foreign Company 40 | Required for UAE branch. | Nymcard if available; otherwise ask/Ops. |
| Memorandum / Articles of Association | Mandatory | Branch parent 40, Free Zone LLC 47, Mainland LLC 48, Offshore 49, Joint Stock 51/52 | Required by entity type. | Pre-fill Nymcard metadata; ask update if stale/missing. |
| Article of Association / Memorandum of Association | Mandatory | Public Joint Stock 52 | Required by entity type. | Ask/Ops if applicable. |
| Memorandum and Article of Association | Mandatory | Private Joint Stock 51 | Required by entity type. | Ask/Ops if applicable. |
| Article of Association / Article or Memorandum of Association | Optional | NPO 41 | Optional by document master. | Ask only if Ops requests. |
| Share Certificate | Mandatory | FZE 44, Offshore 49 | Required by entity type. | Nymcard/MoA if available; otherwise ask/Ops. |
| Share Certificates / Ownership Structure | Mandatory | Free Zone LLC 47 | Required by entity type. | Nymcard/MoA if available; otherwise ask/Ops. |
| Shareholder Register / Ownership Structure of Parent Company | Mandatory | Branch of Foreign Company 40 | Required for parent company. | Ask/Ops when branch/parent path applies. |
| Register of Shareholders & Ownership Structure | Mandatory | Offshore 49 | Required for offshore. | Ask/Ops if applicable. |
| Share Register / Shareholder Register | Mandatory | Private Joint Stock 51, Public Joint Stock 52 | Required by entity type. | Ask/Ops if applicable. |
| Register of Directors / Officers | Mandatory | Offshore 49 | Required for offshore. | Ask/Ops if applicable. |
| List of major shareholders / directors / key management personnel | Mandatory | Private Joint Stock 51, Public Joint Stock 52 | Required by entity type. | Ask/Ops if applicable. |
| List of trustees / board members with ID / Passports | Mandatory | NPO 41 | Required for NPO. | Ask/Ops if applicable. |
| Details / identification of authorised signatories | Mandatory | Federal Gov 45, Emirates Gov 46, Joint Stock 51/52, some NPO/branch cases | Required by entity type or authority path. | V0 collects AS details; Ops completes formal list if needed. |
| Valid Emirates ID and Passport of shareholders, directors and signatories | Mandatory | Mainland LLC 48, Free Zone LLC 47, Partnerships 50, Joint Stock 51/52 and others | Required for relevant people. | Nymcard metadata if available; ask upload if missing/expired. |
| Passport / ID of UAE branch GM / authorised signatories | Mandatory | Branch of Foreign Company 40 | Required for branch GM/signatories. | Ask/Ops if applicable. |
| Passport copy / Emirates ID and resident visa copies of authorised signatories / POA | Mandatory | NPO 41 | Required for authorised signatories/POA. | Ask/Ops if applicable. |
| Business Plan / Business Summary | Mandatory | Freelancer 16 | Required by legal entity type. | Ask/Ops if applicable. |
| Business Plan / Business Profile | Mandatory | FZE 44, Free Zone LLC 47, Mainland LLC 48, Offshore 49, Simple Partnership 50 | Required by legal entity type. | Ask user/company profile or Ops drafts from existing data. |
| Audited financials / 6 months company bank statement / personal bank statement | Mandatory | Freelancer 16, Branch 40, FZE 44, Free Zone LLC 47, Mainland LLC 48, Offshore 49, Simple Partnership 50, Joint Stock 51/52 | Usually required for Ruya KYB. | Ask user upload in V0. |
| Proof of office address / flexidesk address / lease / Ejari / utility bill | Mandatory | Branch 40, NPO 41, Cooperatives 42, FZE 44, Government 46, LLCs 47/48, Offshore 49, Partnerships 50, Joint Stock 51/52 | Required by legal entity/document master. | Nymcard if available; ask if operating address differs or missing. |
| Proof of address of parent company | Mandatory | Branch of Foreign Company 40 | Required for parent company. | Ask/Ops when branch/parent applies. |
| Proof of Address of UBO / Shareholder / Authorised signer | Mandatory when condition applies | Branch 40, NPO 41, FZE 44, LLCs 47/48, Partnerships 50, Joint Stock 51/52 | Required if high-risk national/non-resident condition applies. | Ops determines condition; ask user if triggered. |
| Tax Residency Certificates or equivalent | Optional / Mandatory by entity | Optional for many; Mandatory for Mainland LLC 48 per provided list | Required if document master says mandatory or CRS/FATCA review needs it. | Ops/user follow-up if required. |
| FATCA & CRS Self-Certification Forms | Mandatory for many entities | Branch 40, NPO 41, FZE 44, Government 45/46, LLCs 47/48, Partnerships 50, Joint Stock 51/52 | Required by document master. | Ops/Retool generated or user e-sign/upload depending implementation. |
| Regulatory approvals | Mandatory if applicable | Public Joint Stock 52 and regulated entities | Required when regulated approval applies. | Ask regulated question; Ops validates. |
| Board resolution to open and operate account / POA | Optional / Mandatory by context | Free Zone LLC 47 optional; Mainland LLC 48 optional; Joint Stock mandatory; V0 POA path | Required when user is not AS but has POA, or legal entity requires mandate. | Ask user upload in V0 when POA = Yes; Ops validates. |
| Resolution for registration of branch by Board of Directors | Optional | Branch 40 | Optional by document master. | Ask only if Ops requests. |
| Resolution / mandate to open and operate account / POA | Optional | Simple Partnership 50 | Optional by document master. | Ask only if Ops requests or POA path applies. |
| Resolution of governing body authorising opening account | Mandatory | Cooperatives 42 | Required by entity type. | Ask/Ops if applicable. |
| Board of Directors resolution to open and operate account | Mandatory | Private/Public Joint Stock 51/52 | Required by entity type. | Ask/Ops if applicable. |
| Letter from Free Zone Authority for account opening | Mandatory for under-formation condition | FZE 44, Free Zone LLC 47 | Required when under-formation entity. | Ops/user upload if condition applies. |
| Certificate of Incumbency / Good Standing | Optional / Mandatory by entity | Optional for Branch 40/FZE 44/Free Zone LLC 47; mandatory for Offshore 49 | Required when document master marks mandatory or foreign parent/offshore path. | Ask when parent organisation/outside-UAE path requires it; Ops validates. |
| MOFA attested constitutional documents of immediate parent company | Optional | Offshore 49 | Required if shareholder is another legal entity per condition text. | Ops/user follow-up if company shareholder/parent path applies. |
| Commercial Registration or Trade License of Parent Company | Mandatory | Branch 40 | Required for parent company. | Ask/Ops if parent organisation applies. |
| Partnership Deed / Agreement | Mandatory | Simple Partnership 50 | Required by entity type. | Ask/Ops if applicable. |
| Private Joint Stock establishment certificate | Mandatory | Private Joint Stock 51 | Required by entity type. | Ask/Ops if applicable. |
| Organisational Structure | Mandatory | Public Joint Stock 52 | Required by entity type. | Ask/Ops if applicable. |
| Charter / Constitution | Mandatory | NPO 41 | Required for NPO. | Ask/Ops if applicable. |
| No Objection from Community Development Authority / Ministry | Mandatory | NPO 41 | Required for NPO banking with Ruya. | Ask/Ops if applicable. |
| Minutes document / general assembly validation | Mandatory | NPO 41, Cooperatives 42 | Required for current office bearers. | Ask/Ops if applicable. |
| Decrees for establishment | Optional | NPO 41, Cooperatives 42 | Optional by document master. | Ask only if Ops requests. |
| Letter from Ministry / competent authority to open accounts | Mandatory | Cooperatives 42, Government-like entities | Required by legal entity/document master. | Ask/Ops if applicable. |
| Constitutions and By-Laws | Mandatory | Cooperatives 42 | Required for societies/association/club. | Ask/Ops if applicable. |
| Approval certificate or letter from Ministry of Finance | Mandatory | Federal Gov 45, Emirates Gov 46 | Required by entity type. | Ask/Ops if applicable. |
| Copy of decree establishing ministry / department | Mandatory | Federal Gov 45, Emirates Gov 46 | Required by entity type. | Ask/Ops if applicable. |
| Ministerial Order / documents governing business relationship | Mandatory | Federal Gov 45 | Required by entity type. | Ask/Ops if applicable. |
| Official government license / mandate | Optional | Federal Gov 45 | Optional if commercial arm. | Ask only if Ops requests. |
| Valid license / registration from Community Development Ministry or competent authority | Mandatory / Optional by entity | Mandatory for NPO 41; optional for Cooperatives 42 | Required for NPO; optional for Cooperatives. | Ask/Ops if applicable. |
| Copy of company stamp | Optional | Freelancer 16, Simple Partnership 50 | Optional by document master. | Ask only if Ops requests. |

---

## 5. Consent logic

| Case | Condition | Customer experience | Backend / Ops handling |
|---|---|---|---|
| Initiator is Authorised Signatory | Relationship = Authorised Signatory | Collect consent checkbox in review step. | Map accepted consent to consentMasterId/consentMasterIds. |
| Initiator is Shareholder or Employee with POA | Relationship = Shareholder/Employee and POA = Yes | Ask for POA upload in Documents step; collect consent in review. | Store POA as Ruya upload. Ops validates authority before submission. |
| Initiator is Shareholder or Employee without POA | Relationship = Shareholder/Employee and POA = No | Collect AS name, email, phone. Show routed consent explanation. | Send consent request to AS. If AS is not an Alaan user, Ops onboards AS as Organisation Admin and triggers consent from Retool. |

---

## 6. Core validation rules before Ruya submission

| Rule | Fields involved | V0 handling |
|---|---|---|
| Company name must match | initiator.companyName == companyDetails.companyName | Enforce before Ruya submission. Do not block V0 draft progression. |
| Transaction percentages total 100 | cash/cheque/bank transfer/export LC/others percentages | Ops/system completes before Ruya submission. |
| VAT either/or | vatRegistrationNumber + vatRegistrationDate OR vatMissingReason | Ops completes before Ruya submission. |
| TIN either/or | tinNumber if hasTin = true OR tinUnavailableReasonId if hasTin = false | Ops/user follow-up before Ruya submission. |
| Other citizenship -> tax country | taxResidencyCountryId required when hasOtherCitizenship = true | Ops/user follow-up if required. |
| Residential status -> Emirates ID | Emirates ID fields required for Resident/UAE National/GCC National | V0 asks plain identity path. Emirates ID and Passport are exclusive. |
| Residential status -> Passport | Passport fields required for Non-Resident | V0 asks plain identity path. Emirates ID and Passport are exclusive. |
| Minimum age | dateOfBirth | Official must be at least 18 before Ruya submission. |
| Issue dates before today | tradeLicenseIssueDate, emiratesIdIssueDate, passportIssueDate | Validate before Ruya submission. |
| Expiry dates future | tradeLicenseExpiryDate, emiratesIdExpiryDate, passportExpiryDate | Alert user. Expired docs must be replaced before final processing. |
| No suppliers -> reason | reasonNotDealingSupplierId | Ops handles if supplier records are not collected. |
| No customers -> reason | reasonNotDealingCustomerId | Ops handles if customer records are not collected. |
| No other banks -> reason | reasonNotDealingBankId | Ops handles if other bank records are not collected. |
| Sub-shareholders | parentOfficialId | Only allowed when parent is a Company Shareholder. Ops handles. |
| Ownership percentage | ownershipPercentage | Required by Ruya when relationshipType = SHAREHOLDER. Pre-fill from Nymcard; Ops if missing. |

---

## 7. Master data dependencies

All UUID fields must be mapped to Ruya master data before final API submission.

| Master data type | Used in |
|---|---|
| COUNTRIES | Company, official birth/nationality/tax country, business partners |
| NATIONALITIES | Officials |
| EMIRATES | Company registration, address, officials |
| CITIES | Address, officials |
| ISSUING_AUTHORITIES | Company trade licence |
| LEGAL_TYPES | Company legal structure |
| INDUSTRIES | Company industry |
| SUB_INDUSTRIES | Company sub-industry, filtered by industry |
| CUSTOMER_SEGMENTS | Company/customer segment |
| CB_SECTORS | Company central bank sector |
| CB_SUB_SECTORS | Company central bank sub-sector, filtered by CB sector |
| COMPANY_TYPES | Company category, company shareholder type |
| SOURCES_OF_INCOME | Business details |
| CURRENT_TURNOVER | Business details |
| AVERAGE_EXPECTED_TRANSACTION | Business details |
| MONTHLY_TRANSACTIONS | Business details |
| NATURE_OF_BUSINESS | Business details |
| OWNERSHIP_STRUCTURE | Business details |
| REASON_NOT_DEALING_SUPPLIER | Business relationship fallback |
| REASON_NOT_DEALING_CUSTOMER | Business relationship fallback |
| REASON_NOT_DEALING_BANK | Business relationship fallback |
| BANK_ACCOUNT_TYPES | Business details, other bank accounts |
| RESIDENTIAL_STATUS | Officials |
| REASON_TIN_NOT_AVAILABLE | Officials |
| BANK_CODES | Other bank accounts |

---

## 8. V0 customer-facing sections

| Form section | Purpose | Main customer asks |
|---|---|---|
| Initiator details | Confirm applicant identity and authority. | Relationship with company, POA path if needed, AS details if no POA. |
| Company details | Confirm Nymcard company data and activity. | Review company data, activity-list Yes/No, operating address if different. |
| Shareholders and officials | Confirm who must be included for Ruya. | Review pre-filled people; add missing people if needed. Threshold is 10% or 25% based on activity-list answer. |
| Documents | Collect incremental or replacement documents. | Financial statement/bank statement, POA if applicable, expired document replacements, conditional parent/regulatory docs. |
| Review and consent | Final review and consent handling. | Consent checkbox if AS/POA path; routed AS consent if no POA. |

