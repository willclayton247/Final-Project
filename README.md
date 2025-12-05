# Final-Project
All of our code for our final project
CJonestest

import re
import csv

file = ()

with open(file, "r") as f:
    lines = [line.strip() for line in f if line.strip()]

# Your patterns (unchanged)

pattern = {
    "Date" : r"^[0-9._]+/[0-9._]+/[0-9._]+$",
    "Time" : r"^[0-9]{1,2}:[0-9]{2}[A-Z]$",
    "Courtroom" : r"^[a-zA-Z0-9._]+$",
    "Case" : r"^[0-9]+$",
    "File" : r"^[a-zA-Z0-9._]+$",
    "Defendent" : r"^[A-Z]+,[A-Z]+,[A-Z]+$",
    "Complainant" : r"^[A-Z]+,[A-Z]+,[A-Z]+$",
    "Attorney" : r"^[A-Za-z0-9.+]+$",
    "Continuance" : r"^[0-9]+$",
    "Finger" : r"^[a-zA-Z]+$",
    "Bond" : r"^[0-9]+$",
    "BType" : r"^[A-Z]+$",
    "Charge" : r"^[A-Z.() \-0-9]+$",
    "Plea" : r"^[a-zA-Z]+$",
    "Ver" : r"^[a-zA-Z]+$",
    "CLSp" : r"^[a-zA-Z0-9]+$",
    "Pp" : r"^[a-z]+$",
    "Lp" : r"[a-z]+$",
    "Judgement" : r"^[a-zA-Z]+$",
    "ADAp" : r"^[A-Z]+$"
}

cases = []            # final list of cases
current_case = None   # store fields for current case
collecting_charges = False

# Pattern to detect case header line:
# e.g. "1  21CR 892470 ANDERSON,PATRICIA,CHARLES WYATT,DALLAS ATTY:WAIVED 4"
case_header = re.compile(
    r"^(\d+)\s+([A-Z0-9]+)\s+([A-Z,]+)\s+([A-Z,]+)\s+ATTY:([A-Z]+)\s+(\d+)$"
)

# Pattern for charge lines
charge_line = re.compile(
    r"^\(T\)(.+?)\s+PLEA:\s*(.*?)\s+VER:\s*(.*)$"
)

for line in lines:

    # --------------------------
    # Detect start of CASE
    # --------------------------
    m = case_header.match(line)
    if m:
        # Save previous case (if any)
        if current_case:
            cases.append(current_case)

        # Start new case
        current_case = {
            "CaseNumber": m.group(1),
            "File": m.group(2),
            "Defendant": m.group(3),
            "Complainant": m.group(4),
            "Attorney": m.group(5),
            "Continuance": m.group(6),
            "Charges": []     # <- nested list of charges
        }
        collecting_charges = True
        continue

    # --------------------------
    # Detect CHARGE lines
    # --------------------------
    if collecting_charges and current_case:
        c = charge_line.match(line)
        if c:
            charge_dict = {
                "Charge": c.group(1).strip(),
                "Plea": c.group(2).strip(),
                "Ver": c.group(3).strip()
            }
            current_case["Charges"].append(charge_dict)
            continue

# Save last case
if current_case:
    cases.append(current_case)

# --------------------------
# Write Nested CSV
# --------------------------
output_file = file.rsplit(".", 1)[0] + "_nested.csv"

with open(output_file, "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(["CaseNumber", "File", "Defendant", "Complainant",
                     "Attorney", "Continuance", "Charges"])

    for c in cases:
        # Format nested charges as JSON-like text
        charge_text = "; ".join(
            [f"{ch['Charge']} (Plea: {ch['Plea']}, Ver: {ch['Ver']})"
             for ch in c["Charges"]]
        )

        writer.writerow([
            c["CaseNumber"],
            c["File"],
            c["Defendant"],
            c["Complainant"],
            c["Attorney"],
            c["Continuance"],
            charge_text
        ])

print(f"Nested CSV created: {output_file}")







Updated code 12/5

import re
import csv
import os

# Regular expression patterns for parsing
patterns = {
    "Date": r"^\d{1,2}/\d{1,2}/\d{4}$",
    "Time": r"^\d{1,2}:\d{2}\s*[AP]M$",
    "Courtroom": r"^[0-9]{3}[A-Z]$",
    "Case": r"^\d+$",
    "File": r"^\d{2}[A-Z]{2}\s+\d{6}$",
    "Defendant": r"^[A-Z]+,[A-Z]+(?:,[A-Z]+)?$",
    "Complainant": r"^[A-Z\s,]+$",
    "Attorney": r"^(?:ATTY:|BPD APT\.|PD\s+)?[A-Z:\.\s,]+$",
    "Continuance": r"^\d+$"
}

def parse_court_calendar(file_path):
    """Parse a court calendar text file and return structured data"""
    
    with open(file_path, 'r') as f:
        lines = [line.strip() for line in f if line.strip()]
    
    cases = []
    current_case = {}
    
    i = 0
    while i < len(lines):
        line = lines[i].strip()
        
        # Check for date (start of new session)
        if re.match(patterns["Date"], line):
            current_date = line
            i += 1
            continue
        
        # Check for time
        if re.match(patterns["Time"], line):
            current_time = line
            i += 1
            continue
        
        # Check for courtroom
        if re.match(patterns["Courtroom"], line):
            current_courtroom = line
            i += 1
            continue
        
        # Check for case number (start of new case)
        if re.match(r"^\d+$", line) and len(line) <= 3:
            # Save previous case if exists
            if current_case:
                cases.append(current_case)
            
            # Start new case
            current_case = {
                "Court Date": current_date,
                "Time": current_time,
                "Courtroom No": current_courtroom,
                "Case No": line,
                "File Number": "",
                "Defendant Name": "",
                "Complainant": "",
                "Attorney": "",
                "Continuance": ""
            }
            i += 1
            continue
        
        # Check for file number
        if re.match(patterns["File"], line):
            current_case["File Number"] = line
            i += 1
            continue
        
        # Check for defendant name
        if re.match(patterns["Defendant"], line):
            current_case["Defendant Name"] = line
            i += 1
            continue
        
        # Check for complainant
        if line and not re.match(patterns["Defendant"], line) and "," in line and line.isupper():
            # Check if next line might be attorney
            if i + 1 < len(lines) and ("ATTY:" in lines[i + 1] or "BPD" in lines[i + 1] or "PD" in lines[i + 1]):
                current_case["Complainant"] = line
            elif not current_case["Complainant"]:
                current_case["Complainant"] = line
            i += 1
            continue
        
        # Check for attorney
        if "ATTY:" in line or "BPD APT" in line or "PD " in line or re.match(r"^[A-Z\.\s,]+$", line):
            if not current_case["Attorney"]:
                current_case["Attorney"] = line
            i += 1
            continue
        
        # Check for continuance (usually at the end)
        if re.match(r"^\d+$", line) and len(line) <= 2 and current_case.get("Defendant Name"):
            current_case["Continuance"] = line
            i += 1
            continue
        
        i += 1
    
    # Add last case
    if current_case:
        cases.append(current_case)
    
    return cases

def write_to_csv(cases, output_file):
    """Write parsed cases to CSV file"""
    
    headers = ["Court Date", "Time", "Courtroom No", "Case No", "File Number", 
               "Defendant Name", "Complainant", "Attorney", "Continuance"]
    
    with open(output_file, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=headers)
        writer.writeheader()
        writer.writerows(cases)
    
    print(f"Successfully wrote {len(cases)} cases to {output_file}")

def main():
    """Main function to process court calendar files"""
    
    file_map = {
        "nov1": "court_calendar_Nov1.txt",
        "nov10": "court_calendar_Nov10.txt",
        "nov11": "court_calendar_Nov11.txt"
    }
    
    print("Court Calendar Parser")
    print("=" * 50)
    
    # Ask user which files to process
    user_choices = {}
    for key in file_map:
        choice = input(f"Would you like to process {key}? (yes/no): ").strip().lower()
        user_choices[key] = choice
    
    opened_any = False
    
    for key, txt_file in file_map.items():
        if user_choices[key] == "yes":
            opened_any = True
            csv_file = f"{key}.csv"
            
            try:
                # Check if file exists
                if not os.path.exists(txt_file):
                    print(f"Warning: {txt_file} not found. Skipping...")
                    continue
                
                # Parse the court calendar
                cases = parse_court_calendar(txt_file)
                
                # Write to CSV
                write_to_csv(cases, csv_file)
                
                # Try to open the CSV (Windows only)
                try:
                    os.startfile(csv_file)
                except AttributeError:
                    print(f"CSV file created: {csv_file} (auto-open not available on this OS)")
                
            except Exception as e:
                print(f"Error processing {txt_file}: {e}")
    
    if not opened_any:
        print("No files selected.")

if __name__ == "__main__":
    main()



