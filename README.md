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

