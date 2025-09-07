# Clinical Trial Dataset (Auralin vs Novodra)

## 📌 Dataset Summary
This dataset contains records of **500 patients**, out of which **350 participated in a clinical trial**.  
- None of the patients were using **Novodra** (injectable insulin) or **Auralin** (oral insulin being researched) as their primary insulin before.  
- All participants had **elevated HbA1c levels**.  
- For baseline: all 350 patients were treated with **Novodra for 4 weeks**.  
- Then, patients were split:  
  - **175 switched to Auralin** (for 24 weeks)  
  - **175 continued Novodra** (for 24 weeks)  
- Adverse reaction data was also collected.  

The trial aims to test whether **Auralin is noninferior to Novodra**, defined as the **upper bound of the 95% CI of mean HbA1c difference < 0.4%**.

---

## 📊 Table & Column Descriptions

### **patients**
- `patient_id` → Unique identifier in Master Patient Index  
- `assigned_sex` → Sex at birth (male/female)  
- `given_name` → First name  
- `surname` → Last name  
- `address` → Main address  
- `city`, `state`, `zip_code`, `country` → Location fields (all US)  
- `contact` → Phone + Email together  
- `birthdate` → Date of birth (MM/DD/YYYY); all ≥ 18 years  
- `weight` → In pounds (lbs)  
- `height` → In inches (in)  
- `bmi` → Body Mass Index (kg/m²); trial inclusion: **16 ≤ BMI ≤ 38**

### **treatments / treatment_cut**
- `given_name`, `surname` → Patient identifiers  
- `auralin` → Baseline–End median daily insulin dose (units)  
- `novodra` → Same for Novodra  
- `hba1c_start` → HbA1c level at treatment start (%)  
- `hba1c_end` → HbA1c level at treatment end (%)  
- `hba1c_change` → Difference (start − end); lower = improvement  

### **adverse_reactions**
- `given_name`, `surname` → Patient identifiers  
- `adverse_reaction` → Reported reaction  

---

## ℹ️ Additional Information
- Insulin resistance varies per person → baseline & end dose both required.  
- Trial includes diverse patients (sex, age, race, ethnicity) for better generalization.  
- **Noninferiority check**: mean HbA1c change difference (Novodra − Auralin) CI upper bound < 0.4%.  

---

## ⚠️ Dataset Issues

### **Dirty Data**

**patients**
- `patient_id = 9` → Misspelled name *"Dsvid"* instead of *David* (**accuracy**)  
- `state` → Mixed formats (full name vs abbreviation) (**consistency**)  
- `zip_code` → Some 4-digit values (**validity**)  
- Missing data for 12 patients in (`address`, `city`, `state`, `zip_code`, `country`, `contact`) (**completion**)  
- Wrong datatypes in (`assigned_sex`, `zip_code`, `birthdate`) (**validity**)  
- Duplicate entries (*John Doe*) (**accuracy**)  
- One patient weight = **48 lbs** (implausible) (**accuracy**)  
- One patient height = **27 in** (implausible) (**accuracy**)  

**treatments / treatment_cut**
- `given_name`, `surname` all lowercase (**consistency**)  
- `auralin` & `novodra` contain `"u"` (**validity**)  
- `"-"` values treated as NaN (**validity**)  
- Missing values in `hba1c_change` (**completion**)  
- Duplicate entry (*Joseph Day*) (**accuracy**)  
- `hba1c_change` has **9 instead of 4** (**accuracy**)  

**adverse_reactions**
- `given_name`, `surname` all lowercase (**consistency**)  

---

### **Messy Data**

**patients**
- `contact` contains both phone + email → should be split  

**treatments / treatment_cut**
- `auralin` & `novodra` should be split into **start_dose** and **end_dose**  
- `treatments` and `treatment_cut` should be merged  

**adverse_reactions**
- Should be linked to treatments, not independent  

---

## 🧹 Data Cleaning Plan (Step-by-Step)

1. **patients**
   - Fix name typos (e.g., *Dsvid → David*)  
   - Standardize `state` (USPS 2-letter codes)  
   - Normalize `zip_code` to 5 digits  
   - Handle missing values (imputation / leave blank with flag)  
   - Convert datatypes (sex → category, birthdate → datetime, zip → string)  
   - Remove duplicates (John Doe, etc.)  
   - Flag outliers (weight 48 lbs, height 27 in)  
   - Recompute `bmi` and check inclusion criteria (16 ≤ BMI ≤ 38)  
   - Split `contact` into `phone`, `email`  

2. **treatments / treatment_cut**
   - Title-case names  
   - Remove `"u"` from doses  
   - Split into `start_dose`, `end_dose`  
   - Replace `"-"` with NaN  
   - Merge `treatments` + `treatment_cut`  
   - Recompute `hba1c_change = hba1c_start − hba1c_end`  
   - Fix incorrect values (e.g., 9 → 4)  
   - Drop duplicate (Joseph Day)  

3. **adverse_reactions**
   - Title-case names  
   - Merge with patients & treatments for complete clinical dataset  

---

## ✅ Final Outputs
- Clean tables: `patients_clean.csv`, `treatments_clean.csv`, `adverse_reactions_clean.csv`  
- Merged master file: `master_clinical.csv`  
- Analysis script generates: `reports/summary.md` with HbA1c statistics & noninferiority check  
