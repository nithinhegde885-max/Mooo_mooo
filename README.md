# ==========================================================
# JOBSPHERE - JOB MATCHING SYSTEM (SCHOOL PROJECT)
# Dark Theme | Tkinter | CSV Storage | SAFE WINDOW CLOSE
# ==========================================================

import tkinter as tk
from tkinter import messagebox
import csv, os

# ---------- THEME COLORS ----------
BG = "#121212"       # Main background
CARD = "#1e1e1e"     # Card background
FG = "#e5e7eb"       # Text color
BTN = "#2563eb"      # Primary button
BTN2 = "#374151"     # Secondary button
INP = "#2a2a2a"      # Input background

# ---------- FILE NAMES ----------
EMP = "employees.csv"
REC = "recruiters.csv"

# ---------- SKILLS ----------
SKILLS = ["Python","Java","C++","JavaScript","HTML","CSS","SQL"]

# ---------- CSV HEADERS ----------
HDR = {
    "employee": ["username","password","name","number","address","email","gender","age","skills"],
    "recruiter": ["username","password","company","contact","email","address","skills"]
}

# ---------- SESSION ----------
session = {}

# ==========================================================
# CSV FUNCTIONS
# ==========================================================

def ensure(path, headers):
    if not os.path.exists(path):
        with open(path,"w",newline="") as f:
            csv.writer(f).writerow(headers)

def read(path):
    if not os.path.exists(path): return []
    with open(path,newline="") as f:
        return list(csv.DictReader(f))

def write(path, headers, rows):
    with open(path,"w",newline="") as f:
        w = csv.DictWriter(f, fieldnames=headers)
        w.writeheader()
        w.writerows(rows)

def find(username):
    for role, path in [("employee", EMP), ("recruiter", REC)]:
        for row in read(path):
            if row["username"] == username:
                return role, row
    return None, None

# ==========================================================
# GUI SETUP
# ==========================================================

root = tk.Tk()
root.title("Jobsphere")
root.configure(bg=BG)

# ---------- SAFE WINDOW CLOSE (STEP 1) ----------
def on_close():
    """
    Safely close the Tkinter window without crashing.
    """
    root.quit()
    root.destroy()

# ---------- REGISTER CLOSE HANDLER (STEP 2) ----------
root.protocol("WM_DELETE_WINDOW", on_close)

# ==========================================================
# UI HELPERS
# ==========================================================

def clear():
    for w in root.winfo_children():
        w.destroy()

def title(text):
    tk.Label(root, text=text, font=("Arial",18,"bold"),
             bg=BG, fg=FG).pack(pady=10)

def box():
    f = tk.Frame(root, bg=CARD, padx=18, pady=15)
    f.pack(pady=10)
    return f

def entry(parent, hidden=False):
    e = tk.Entry(parent, bg=INP, fg=FG,
                 insertbackground=FG,
                 show="*" if hidden else "")
    e.pack(fill="x", pady=3)
    return e

def button(text, cmd, primary=True):
    tk.Button(root, text=text, command=cmd,
              width=20,
              bg=BTN if primary else BTN2,
              fg="white",
              relief="flat").pack(pady=4)

# ==========================================================
# START SCREEN
# ==========================================================

def start():
    clear()
    title("Jobsphere")
    button("Register", register)
    button("Login", login)
    button("Quit", on_close, False)

# ==========================================================
# REGISTER
# ==========================================================

def register():
    clear()
    title("Register")
    f = box()

    tk.Label(f, text="Username", bg=CARD, fg=FG).pack(anchor="w")
    u = entry(f)

    tk.Label(f, text="Password", bg=CARD, fg=FG).pack(anchor="w")
    p = entry(f, True)

    role = tk.StringVar(value="employee")
    for r in ("employee", "recruiter"):
        tk.Radiobutton(f, text=r.capitalize(),
                       variable=role, value=r,
                       bg=CARD, fg=FG,
                       selectcolor=CARD).pack(anchor="w")

    def next_step():
        if not u.get() or not p.get():
            return messagebox.showerror("Error","All fields required")
        if find(u.get())[0]:
            return messagebox.showerror("Error","Username already exists")
        session.update(username=u.get(), password=p.get(), role=role.get())
        details(True)

    button("Next", next_step)
    button("Back", start, False)

# ==========================================================
# LOGIN
# ==========================================================

def login():
    clear()
    title("Login")
    f = box()

    tk.Label(f, text="Username", bg=CARD, fg=FG).pack(anchor="w")
    u = entry(f)

    tk.Label(f, text="Password", bg=CARD, fg=FG).pack(anchor="w")
    p = entry(f, True)

    def do_login():
        role, user = find(u.get())
        if not user or user["password"] != p.get():
            return messagebox.showerror("Error","Invalid login")
        session.update(username=u.get(), password=p.get(), role=role)
        details(False)

    button("Login", do_login)
    button("Back", start, False)

# ==========================================================
# DETAILS
# ==========================================================

def details(new_user):
    clear()
    title("Details")
    f = box()

    role = session["role"]
    fields = HDR[role][2:-1]
    inputs = {}

    for field in fields:
        tk.Label(f, text=field.capitalize(),
                 bg=CARD, fg=FG).pack(anchor="w")
        inputs[field] = entry(f)

    if not new_user:
        _, user = find(session["username"])
        for k in inputs:
            inputs[k].insert(0, user.get(k,""))

    def save_details():
        row = {"username":session["username"],
               "password":session["password"],
               "skills":""}
        for k in inputs:
            row[k] = inputs[k].get()

        path = EMP if role == "employee" else REC
        rows = read(path)
        rows.append(row)
        write(path, HDR[role], rows)
        skills()

    button("Next", save_details)
    button("Back", start, False)

# ==========================================================
# SKILLS
# ==========================================================

def skills():
    clear()
    title("Select Skills")
    f = box()

    selected = []
    for s in SKILLS:
        v = tk.IntVar()
        tk.Checkbutton(f, text=s, variable=v,
                       bg=CARD, fg=FG,
                       selectcolor=CARD).pack(anchor="w")
        selected.append((s,v))

    def save_skills():
        skill_text = ";".join(s for s,v in selected if v.get())
        path = EMP if session["role"] == "employee" else REC
        rows = read(path)
        for r in rows:
            if r["username"] == session["username"]:
                r["skills"] = skill_text
        write(path, HDR[session["role"]], rows)
        match()

    button("Proceed", save_skills)
    button("Back", start, False)

# ==========================================================
# MATCH RESULTS
# ==========================================================

def match():
    clear()
    title("Matching Results")

    out = tk.Text(root, width=60, height=15,
                  bg=INP, fg=FG, relief="flat")
    out.pack(pady=10)

    _, user = find(session["username"])
    user_skills = set(user["skills"].split(";")) if user["skills"] else set()
    others = read(REC if session["role"] == "employee" else EMP)

    found = False
    for o in others:
        other_skills = set(o["skills"].split(";")) if o["skills"] else set()
        common = user_skills & other_skills
        if common:
            found = True
            out.insert("end", f"{o['username']} → {', '.join(common)}\n")

    if not found:
        out.insert("end", "No matches found.")

    out.config(state="disabled")
    button("Home", start)

# ==========================================================
# PROGRAM START
# ==========================================================

ensure(EMP, HDR["employee"])
ensure(REC, HDR["recruiter"])
start()
root.mainloop()
