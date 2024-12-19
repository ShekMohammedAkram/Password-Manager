# Password-Manager
import tkinter as tk
from tkinter import messagebox
from tkinter import simpledialog
import secrets
import string
from cryptography.fernet import Fernet
import json
import os

# Set up the key for encryption/decryption (generated once and saved for later use)
def load_or_generate_key():
    if not os.path.exists("key.key"):
        key = Fernet.generate_key()
        with open("key.key", "wb") as key_file:
            key_file.write(key)
    else:
        with open("key.key", "rb") as key_file:
            key = key_file.read()
    return key

key = load_or_generate_key()
cipher_suite = Fernet(key)

# Password Manager Class
class PasswordManager:
    def __init__(self):
        self.data_file = "passwords.json"
        self.load_data()

    def load_data(self):
        try:
            with open(self.data_file, "r") as f:
                self.data = json.load(f)
        except (FileNotFoundError, json.JSONDecodeError):
            self.data = {}

    def save_data(self):
        with open(self.data_file, "w") as f:
            json.dump(self.data, f, indent=4)

    def add_password(self, website, username, password):
        encrypted_password = cipher_suite.encrypt(password.encode()).decode()
        self.data[website] = {"username": username, "password": encrypted_password}
        self.save_data()

    def retrieve_password(self, website):
        entry = self.data.get(website)
        if entry:
            decrypted_password = cipher_suite.decrypt(entry["password"].encode()).decode()
            return entry["username"], decrypted_password
        else:
            return None

# Generate a secure random password
def generate_password(length=12):
    characters = string.ascii_letters + string.digits + string.punctuation
    return ''.join(secrets.choice(characters) for _ in range(length))

# GUI
def add_password():
    website = website_entry.get()
    username = username_entry.get()
    password = password_entry.get()

    if website and username and password:
        manager.add_password(website, username, password)
        messagebox.showinfo("Success", f"Password for {website} saved!")
    else:
        messagebox.showwarning("Input Error", "Please fill out all fields.")

def generate_and_fill_password():
    generated_password = generate_password()
    password_entry.delete(0, tk.END)
    password_entry.insert(0, generated_password)

def retrieve_password():
    website = website_entry.get()
    if website:
        result = manager.retrieve_password(website)
        if result:
            username, password = result
            messagebox.showinfo("Password Found", f"Username: {username}\nPassword: {password}")
        else:
            messagebox.showwarning("Not Found", "No entry found for this website.")
    else:
        messagebox.showwarning("Input Error", "Please enter a website.")

# Set up the manager and GUI
manager = PasswordManager()

root = tk.Tk()
root.title("Password Manager")

tk.Label(root, text="Website:").grid(row=0, column=0, padx=5, pady=5)
website_entry = tk.Entry(root, width=30)
website_entry.grid(row=0, column=1, padx=5, pady=5)

tk.Label(root, text="Username:").grid(row=1, column=0, padx=5, pady=5)
username_entry = tk.Entry(root, width=30)
username_entry.grid(row=1, column=1, padx=5, pady=5)

tk.Label(root, text="Password:").grid(row=2, column=0, padx=5, pady=5)
password_entry = tk.Entry(root, width=30, show="*")
password_entry.grid(row=2, column=1, padx=5, pady=5)

generate_button = tk.Button(root, text="Generate Password", command=generate_and_fill_password)
generate_button.grid(row=2, column=2, padx=5, pady=5)

add_button = tk.Button(root, text="Add Password", command=add_password)
add_button.grid(row=3, column=1, padx=5, pady=5)

retrieve_button = tk.Button(root, text="Retrieve Password", command=retrieve_password)
retrieve_button.grid(row=3, column=2, padx=5, pady=5)

root.mainloop()
