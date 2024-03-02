import hashlib
import os
import sqlite3
import shutil
from datetime import datetime
from getpass import getpass

# Set up the database connection
conn = sqlite3.connect('diary.db')
cursor = conn.cursor()

# Create the tables if they don't exist
cursor.execute('''
CREATE TABLE IF NOT EXISTS diary_entries (
    id INTEGER PRIMARY KEY,
    date TEXT NOT NULL,
    score INTEGER NOT NULL,
    feedback TEXT NOT NULL
)''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS images (
    id INTEGER PRIMARY KEY,
    date TEXT NOT NULL,
    image_path TEXT NOT NULL,
    FOREIGN KEY(date) REFERENCES diary_entries(date)
)''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS user (
    id INTEGER PRIMARY KEY,
    password_hash TEXT NOT NULL
)''')

conn.commit()

# Function to hash the password
def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# Function to set a new password
def set_new_password():
    new_password = getpass("Enter a new 6-digit PIN: ")
    if len(new_password) == 6 and new_password.isdigit():
        password_hash = hash_password(new_password)
        cursor.execute('DELETE FROM user')  # Remove any existing password
        cursor.execute('INSERT INTO user (password_hash) VALUES (?)', (password_hash,))
        conn.commit()
        print("New PIN set successfully.")
    else:
        print("Invalid PIN. The PIN must be 6 digits.")

# Function to check the password
def check_password():
    saved_password_hash = cursor.execute('SELECT password_hash FROM user LIMIT 1').fetchone()
    if saved_password_hash:
        password = getpass("Enter your 6-digit PIN to continue: ")
        password_hash = hash_password(password)
        return password_hash == saved_password_hash[0]
    else:
        print("No PIN set. Please create a new PIN.")
        set_new_password()
        return True

# Function to add new diary entry with image
def add_diary_entry():
    score = input("Enter your score (out of 10): ")
    feedback = input("Enter your feedback for the day: ")
    image_path = input("Enter the path to your image (leave empty if none): ")
    date_str = datetime.now().strftime("%Y-%m-%d")
    
    cursor.execute('INSERT INTO diary_entries (date, score, feedback) VALUES (?, ?, ?)',
                   (date_str, score, feedback))
    conn.commit()
    
    if image_path:
        if not os.path.exists('diary_images'):
            os.makedirs('diary_images')
        try:
            image_filename = f"{date_str}.jpg"
            dest_path = os.path.join('diary_images', image_filename)
            shutil.copy(image_path, dest_path)
            cursor.execute('INSERT INTO images (date, image_path) VALUES (?, ?)',
                           (date_str, dest_path))
            conn.commit()
            print(f"Image saved as {image_filename}")
        except IOError as e:
            print(f"An error occurred while saving the image: {e}")
    
    print("Diary entry saved.")

# Function to retrieve and display the image path associated with a diary entry
def show_diary_entry_image(date_str):
    cursor.execute('SELECT image_path FROM images WHERE date = ?', (date_str,))
    image_entry = cursor.fetchone()
    if image_entry:
        print(f"Image path for the entry: {image_entry[0]}")
    else:
        print("No image found for this entry.")

# Function to retrieve diary entry by date
def retrieve_diary_entry():
    date_str = input("Enter the date for the entry you wish to retrieve (YYYY-MM-DD): ")
    cursor.execute('SELECT * FROM diary_entries WHERE date = ?', (date_str,))
    entry = cursor.fetchone()
    if entry:
        print(f"Date: {entry[1]}, Score: {entry[2]}, Feedback: {entry[3]}")
        show_diary_entry_image(date_str)
    else:
        print("No entry found for this date.")

# Main function with menu
def main():
    if not check_password():
        print("Incorrect PIN. Exiting program.")
        return  # Exit if the PIN is incorrect

    while True:
        print("\nDiary Menu:")
        print("1. Add new diary entry")
        print("2. Retrieve diary entry")
        print("3. Change PIN")
        print("4. Exit")
        choice = input("Choose an option: ")

        if choice == '1':
            add_diary_entry()
        elif choice == '2':
            retrieve_diary_entry()
        elif choice == '3':
            if check_password():
                set_new_password()
            else:
                print("Incorrect PIN. You cannot change the PIN.")
        elif choice == '4':
            break
        else:
            print("Invalid option. Please try again.")

    conn.close()

if __name__ == "__main__":
    main()