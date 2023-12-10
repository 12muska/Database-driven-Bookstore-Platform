# Database-driven-Bookstore-Platform

Python and MySQL. This project will include functionalities like viewing books, registering customers, placing orders, filtering books by genre, and sorting books by title, author, or price.

Step 1: Database Setup

Create a MySQL database and tables for books, customers, orders, and reviews. You can use a MySQL client like MySQL Workbench or a command-line interface.

sql


-- Create the bookstore database
CREATE DATABASE bookstore;

-- Use the bookstore database
USE bookstore;

-- Create the books table
CREATE TABLE books (
    book_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    genre VARCHAR(100),
    price DECIMAL(8, 2) NOT NULL,
    quantity INT NOT NULL
);

-- Create the customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL
);

-- Create the orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Create the order_items table
CREATE TABLE order_items (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    book_id INT,
    quantity INT NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id)
);

-- Create the reviews table
CREATE TABLE reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    book_id INT,
    customer_id INT,
    rating INT NOT NULL,
    comment TEXT,
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
Step 2: Python Script

Create a Python script (app.py) to interact with the MySQL database and implement basic functionality. This script will include functions for viewing books, registering customers, placing orders, filtering books by genre, and sorting books.

python


import mysql.connector
from getpass import getpass

# Connect to MySQL
connection = mysql.connector.connect(
    host="localhost",
    user="your_username",
    password=getpass("Enter MySQL password: "),
    database="bookstore"
)

cursor = connection.cursor()

def fetch_all_books():
    query = "SELECT * FROM books"
    cursor.execute(query)
    books = cursor.fetchall()
    return books

def fetch_books_by_genre(genre):
    query = "SELECT * FROM books WHERE genre = %s"
    cursor.execute(query, (genre,))
    books = cursor.fetchall()
    return books

def sort_books(order_by):
    query = f"SELECT * FROM books ORDER BY {order_by}"
    cursor.execute(query)
    books = cursor.fetchall()
    return books

def register_customer(username, password):
    query = "INSERT INTO customers (username, password) VALUES (%s, %s)"
    values = (username, password)
    cursor.execute(query, values)
    connection.commit()

def place_order(customer_id, book_id, quantity):
    # Check if the book is available in sufficient quantity
    check_quantity_query = "SELECT quantity FROM books WHERE book_id = %s"
    cursor.execute(check_quantity_query, (book_id,))
    available_quantity = cursor.fetchone()[0]
    if available_quantity < quantity:
        return "Error: Insufficient quantity available."

    # Place the order
    order_query = "INSERT INTO orders (customer_id) VALUES (%s)"
    cursor.execute(order_query, (customer_id,))
    order_id = cursor.lastrowid

    order_item_query = "INSERT INTO order_items (order_id, book_id, quantity) VALUES (%s, %s, %s)"
    cursor.execute(order_item_query, (order_id, book_id, quantity))

    # Update the book quantity
    update_quantity_query = "UPDATE books SET quantity = quantity - %s WHERE book_id = %s"
    cursor.execute(update_quantity_query, (quantity, book_id))

    connection.commit()
    return f"Order placed successfully. Order ID: {order_id}"

def main():
    print("Welcome to the Online Bookstore!")

    # Example: Fetch all books
    books = fetch_all_books()
    for book in books:
        print(f"{book[0]}. {book[1]} by {book[2]} - ${book[4]}")

    # Example: Fetch books by genre
    genre = input("Enter a genre to filter books (press Enter to skip): ")
    if genre:
        filtered_books = fetch_books_by_genre(genre)
        print(f"\nBooks in the {genre} genre:")
        for book in filtered_books:
            print(f"{book[0]}. {book[1]} by {book[2]} - ${book[4]}")
    
    # Example: Sort books
    order_by = input("\nEnter sort option (title/author/price - press Enter to skip): ")
    if order_by:
        sorted_books = sort_books(order_by)
        print("\nSorted Books:")
        for book in sorted_books:
            print(f"{book[0]}. {book[1]} by {book[2]} - ${book[4]}")

    # Example: Register a customer
    username = input("\nEnter your username: ")
    password = getpass("Enter your password: ")
    register_customer(username, password)
    print("Customer registered successfully.")

    # Example: Place an order
    customer_id = 1  # You can fetch the customer_id after registration
    book_id = int(input("\nEnter the book ID you want to order: "))
    quantity = int(input("Enter the quantity: "))
    result = place_order(customer_id, book_id, quantity)
    print(result)

if __name__ == "__main__":
    main()

# Close the connection
cursor.close()
connection.close()
