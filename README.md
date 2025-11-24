# Final-Project
All of our code for our final project
CJonestest

import re

file = input("Enter file name: ")

f = open(file, "r")

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
    "Charge" : r"^[A-Z.]+$",
    "Plea" : r"^[a-zA-Z]+$",
    "Ver" : r"^[a-zA-Z]+$",
    "CLSp" : r"^[a-zA-Z0-9]+$",
    "Pp" : r"^[a-z]+$",
    "Lp" : r"^[a-z]+$",
    "Judgement" : r"^[a-zA-Z]+$",
    "ADAp" : r"^[A-Z]+$"

}

extracted = {}

for line in f:
    matches = re.findall(pattern, f)
    extracted = matches
