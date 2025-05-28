import tkinter as tk
from tkinter import ttk, messagebox
import os
import json
import smtplib
from email.message import EmailMessage

CONTACTS_FILE = "contacts.json"
selected_contact_name = None

# Load
def load_contacts():
    if os.path.exists(CONTACTS_FILE):
        with open(CONTACTS_FILE, "r") as f:
            return json.load(f)
    return []

# Save
def save_contacts(contacts):
    with open(CONTACTS_FILE, "w") as f:
        json.dump(contacts, f, indent=2)

# Add
def add_contact():
    name = entry_name.get().strip()
    phone = entry_phone.get().strip()
    email = entry_email.get().strip()

    if not name or not phone or not email:
        messagebox.showwarning("Missing Info", "Please fill all fields.")
        return

    contacts = load_contacts()
    for c in contacts:
        if c["name"].lower() == name.lower():
            messagebox.showerror("Duplicate", "Contact with this name already exists.")
            return

    contacts.append({"name": name, "phone": phone, "email": email})
    save_contacts(contacts)
    messagebox.showinfo("Success", "Contact added.")
    clear_entries()
    view_contacts()

# View
def view_contacts():
    contacts = load_contacts()
    listbox.delete(*listbox.get_children())
    for c in contacts:
        listbox.insert('', 'end', values=(c['name'], c['phone'], c['email']))

# Search
def search_contact():
    name = entry_search.get().strip().lower()
    contacts = load_contacts()
    listbox.delete(*listbox.get_children())
    found = False
    for c in contacts:
        if name in c["name"].lower():
            listbox.insert('', 'end', values=(c['name'], c['phone'], c['email']))
            found = True
    if not found:
        messagebox.showinfo("Search", "No contact found.")

# Delete 
def delete_contact():
    name = entry_search.get().strip().lower()
    contacts = load_contacts()
    new_contacts = [c for c in contacts if name not in c["name"].lower()]
    if len(contacts) == len(new_contacts):
        messagebox.showerror("Not Found", "Contact not found.")
    else:
        save_contacts(new_contacts)
        messagebox.showinfo("Deleted", "Contact deleted.")
        view_contacts()

# Update
def update_contact():
    global selected_contact_name
    if selected_contact_name is None:
        messagebox.showwarning("Select Contact", "Please select a contact to update.")
        return

    new_name = entry_name.get().strip()
    new_phone = entry_phone.get().strip()
    new_email = entry_email.get().strip()

    if not new_name or not new_phone or not new_email:
        messagebox.showwarning("Missing Info", "Please fill all fields.")
        return

    contacts = load_contacts()
    updated = False
    for c in contacts:
        if c["name"].lower() == selected_contact_name.lower():
            c["name"] = new_name
            c["phone"] = new_phone
            c["email"] = new_email
            updated = True
            break

    if updated:
        save_contacts(contacts)
        messagebox.showinfo("Updated", "Contact updated.")
        clear_entries()
        view_contacts()
        selected_contact_name = None
    else:
        messagebox.showerror("Not Found", "Selected contact not found.")

# Handle row selection
def on_row_selected(event):
    global selected_contact_name
    selected = listbox.focus()
    if selected:
        values = listbox.item(selected, 'values')
        if values:
            selected_contact_name = values[0]
            entry_name.delete(0, tk.END)
            entry_name.insert(0, values[0])
            entry_phone.delete(0, tk.END)
            entry_phone.insert(0, values[1])
            entry_email.delete(0, tk.END)
            entry_email.insert(0, values[2])

# Clear
def clear_entries():
    global selected_contact_name
    entry_name.delete(0, tk.END)
    entry_phone.delete(0, tk.END)
    entry_email.delete(0, tk.END)
    selected_contact_name = None

def send_email():
    global selected_contact_name
    if selected_contact_name is None:
        messagebox.showwarning("Select Contact", "Please select a contact to email.")
        return

    contacts = load_contacts()
    recipient = None
    for c in contacts:
        if c["name"].lower() == selected_contact_name.lower():
            recipient = c["email"]
            break

    if not recipient:
        messagebox.showerror("Error", "Recipient email not found.")
        return

    sender_email = "seef25theknight@gmail.com"
    sender_password = "xoss kiga ydmf hlfg"  
    subject = "Hello from Contacts App"
    body = f"Hi {selected_contact_name},\n\nThis is a test message from my contact manager app!"

    try:
        msg = EmailMessage()
        msg['From'] = sender_email
        msg['To'] = recipient
        msg['Subject'] = subject
        msg.set_content(body)

        with smtplib.SMTP_SSL('smtp.gmail.com', 465) as smtp:
            smtp.login(sender_email, sender_password)
            smtp.send_message(msg)

        messagebox.showinfo("Email Sent", f"Email sent to {recipient}")
    except Exception as e:
        messagebox.showerror("Email Failed", f"Failed to send email.\n{e}")



# GUI Setup
root = tk.Tk()
root.title("📇 Contacts App")
root.geometry("600x500")
root.configure(bg="#f0f2f5")

style = ttk.Style(root)
style.theme_use("clam")
style.configure("TLabel", font=("Segoe UI", 11), background="#f0f2f5")
style.configure("TButton", font=("Segoe UI", 10), padding=5)
style.configure("Treeview", font=("Segoe UI", 10), rowheight=25)
style.configure("Treeview.Heading", font=("Segoe UI", 11, "bold"))

frame_top = ttk.Frame(root, padding=10)
frame_top.pack(fill='x')

ttk.Label(frame_top, text="Name:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
entry_name = ttk.Entry(frame_top)
entry_name.grid(row=0, column=1, padx=5, pady=5)

ttk.Label(frame_top, text="Phone:").grid(row=1, column=0, padx=5, pady=5, sticky='e')
entry_phone = ttk.Entry(frame_top)
entry_phone.grid(row=1, column=1, padx=5, pady=5)

ttk.Label(frame_top, text="Email:").grid(row=2, column=0, padx=5, pady=5, sticky='e')
entry_email = ttk.Entry(frame_top)
entry_email.grid(row=2, column=1, padx=5, pady=5)

ttk.Button(frame_top, text="Add Contact", command=add_contact).grid(row=3, column=0, columnspan=1, pady=10)
ttk.Button(frame_top, text="Update Contact", command=update_contact).grid(row=3, column=1, columnspan=1, pady=10)

frame_search = ttk.Frame(root, padding=10)
frame_search.pack(fill='x')

ttk.Label(frame_search, text="Search Name:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
entry_search = ttk.Entry(frame_search)
entry_search.grid(row=0, column=1, padx=5, pady=5)

ttk.Button(frame_search, text="Search", command=search_contact).grid(row=0, column=2, padx=5)
ttk.Button(frame_search, text="Delete", command=delete_contact).grid(row=0, column=3, padx=5)
ttk.Button(frame_search, text="View All", command=view_contacts).grid(row=0, column=4, padx=5)
ttk.Button(frame_top, text="Send Email", command=send_email).grid(row=4, column=0, columnspan=2, pady=10)

frame_list = ttk.Frame(root, padding=10)
frame_list.pack(fill='both', expand=True)

columns = ('Name', 'Phone', 'Email')
listbox = ttk.Treeview(frame_list, columns=columns, show='headings')
for col in columns:
    listbox.heading(col, text=col)
    listbox.column(col, anchor='center')

listbox.pack(fill='both', expand=True)
listbox.bind('<<TreeviewSelect>>', on_row_selected)

view_contacts()
root.mainloop()

