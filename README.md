Overview of the program's capabilities:

1) Password Protection: The program secures access to the diary with a 6-digit PIN. Upon first run or if no PIN is set, it prompts the user to create a new PIN. The PIN is then hashed and stored in a SQLite database for security purposes.
2) Diary Entry Management:
-  Adding Entries: Users can add new diary entries which include a date, a numerical score (out of 10), and a text-based feedback. The date is automatically set to the current date.
-  Image Attachment: Alongside textual feedback, users have the option to attach an image to each diary entry by providing the file path to the image. The program copies the image to a dedicated diary_images directory and saves the path in the database.
-  Retrieving Entries: Users can retrieve past diary entries by inputting the date of the entry they wish to view. The program will then display the score, feedback, and the path to any associated image.
3) Database Interaction: The script uses SQLite, a lightweight database engine, to store and manage data. All entries, images, and user credentials are stored in the diary.db database file. The schema defines three tables for diary entries, images, and user data.
4) User Interface: The application runs in the console, providing a text-based interface where users can interact with the diary through a menu system. Options include adding entries, viewing past entries, changing the PIN, or exiting the program.
5) Data Validation: The script includes basic validation, such as checking the length and content of the PIN.
6) Error Handling: There is minimal error handling, primarily for image file operations. The program will inform the user if there was an error saving an image.
7) File Operations: Images are managed by copying them into a specific directory, renaming them according to the date of the diary entry.
