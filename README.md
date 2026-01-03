import sqlite3
from tkinter import *
from tkinter import messagebox, ttk

# ---------------- Database ---------------- #

conn = sqlite3.connect("students.db")
cursor = conn.cursor()
cursor.execute("""
CREATE TABLE IF NOT EXISTS students(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    gender TEXT,
    age INTEGER,
    course TEXT,
    contact TEXT
)
""")
conn.commit()

# ---------------- Functions ---------------- #

def add_student():
    name = name_entry.get()
    gender = gender_var.get()
    age = age_entry.get()
    course = course_entry.get()
    contact = contact_entry.get()

    if name == "" or age == "" or course == "" or contact == "":
        messagebox.showerror("Error", "All fields are required")
        return

    cursor.execute("INSERT INTO students(name, gender, age, course, contact) VALUES (?,?,?,?,?)",
                   (name, gender, age, course, contact))
    conn.commit()
    messagebox.showinfo("Success", "Student added successfully")
    clear_fields()
    show_students()


def show_students():
    cursor.execute("SELECT * FROM students")
    rows = cursor.fetchall()

    for row in tree.get_children():
        tree.delete(row)

    for row in rows:
        tree.insert("", END, values=row)


def delete_student():
    selected = tree.focus()
    if selected == "":
        messagebox.showerror("Error", "Select a student to delete")
        return

    data = tree.item(selected)['values']
    student_id = data[0]

    cursor.execute("DELETE FROM students WHERE id=?", (student_id,))
    conn.commit()
    messagebox.showinfo("Deleted", "Student deleted successfully")
    show_students()


def update_student():
    selected = tree.focus()
    if selected == "":
        messagebox.showerror("Error", "Select a student to update")
        return

    data = tree.item(selected)['values']
    student_id = data[0]

    cursor.execute("""
        UPDATE students SET 
        name=?, gender=?, age=?, course=?, contact=? WHERE id=?
    """, (name_entry.get(), gender_var.get(), age_entry.get(),
          course_entry.get(), contact_entry.get(), student_id))

    conn.commit()
    messagebox.showinfo("Updated", "Student updated successfully")
    show_students()


def search_student():
    keyword = search_entry.get()
    cursor.execute("SELECT * FROM students WHERE name LIKE ?", ('%' + keyword + '%',))
    rows = cursor.fetchall()

    for row in tree.get_children():
        tree.delete(row)

    for row in rows:
        tree.insert("", END, values=row)


def clear_fields():
    name_entry.delete(0, END)
    age_entry.delete(0, END)
    course_entry.delete(0, END)
    contact_entry.delete(0, END)
    gender_var.set("Male")

# ---------------- GUI ---------------- #

root = Tk()
root.title("Student Management System")
root.geometry("800x500")
root.config(bg="#f2f2f2")

title = Label(root, text="Student Management System", font=("Arial", 18, "bold"), bg="#f2f2f2")
title.pack(pady=10)

frame = Frame(root, bg="#f2f2f2")
frame.pack()

# Labels & Entry Fields
Label(frame, text="Name:", bg="#f2f2f2").grid(row=0, column=0)
name_entry = Entry(frame)
name_entry.grid(row=0, column=1)

Label(frame, text="Gender:", bg="#f2f2f2").grid(row=1, column=0)
gender_var = StringVar(value="Male")
gender_box = ttk.Combobox(frame, textvariable=gender_var, values=["Male", "Female"], width=17)
gender_box.grid(row=1, column=1)

Label(frame, text="Age:", bg="#f2f2f2").grid(row=2, column=0)
age_entry = Entry(frame)
age_entry.grid(row=2, column=1)

Label(frame, text="Course:", bg="#f2f2f2").grid(row=3, column=0)
course_entry = Entry(frame)
course_entry.grid(row=3, column=1)

Label(frame, text="Contact:", bg="#f2f2f2").grid(row=4, column=0)
contact_entry = Entry(frame)
contact_entry.grid(row=4, column=1)

# Buttons
Button(frame, text="Add", command=add_student, width=12).grid(row=5, column=0, pady=10)
Button(frame, text="Update", command=update_student, width=12).grid(row=5, column=1)
Button(frame, text="Delete", command=delete_student, width=12).grid(row=6, column=0)
Button(frame, text="Clear", command=clear_fields, width=12).grid(row=6, column=1)

# Search Bar
Label(root, text="Search by Name:", bg="#f2f2f2").pack()
search_entry = Entry(root)
search_entry.pack()

Button(root, text="Search", command=search_student).pack(pady=5)

# Table
tree = ttk.Treeview(root, columns=("ID", "Name", "Gender", "Age", "Course", "Contact"), show="headings")
tree.pack(fill=BOTH, expand=True)

for col in ("ID", "Name", "Gender", "Age", "Course", "Contact"):
    tree.heading(col, text=col)

show_students()
root.mainloop()
