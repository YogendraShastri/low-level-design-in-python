# low-level-design-in-python
For beginners lets see the easy LLD for Library System. it can be more complex but my objective here is to give easy LLD design so that beginner level person can easily understand. 

### Requirements :
**Functional Requirements**
* Admin should be able to add new books to the library.
* Admin should be able to remove existing books.
* Students should be able to:
   - View all books in the library.
   - Borrow a book if available.
   - Return a previously borrowed book.
* A book cannot be borrowed if it is already issued.
* A student can only return a book they have borrowed.
* Book availability should be updated in real-time.

**Non-Functional Requirements**
* system must be simple and suitable for beginners.
* Code must follow OOP principles: encapsulation, separation of concerns.
* Console-based UI (no GUI or web required).
* Code must be readable and maintainable for interns or new developers.

### Diagram :
```python

           +------------------------+
           |        Admin           |
           +------------------------+
           | - name                 |
           | - library: Library     |
           +------------------------+
           | +add_new_book(...)     |
           | +delete_book(...)      |
           +-----------+------------+
                       |
                       | uses
                       v
           +-------------------------------+
           |       Library                 |
           +-------------------------------+
           | - books: Dict[book_id]        |
           | - book_assigned               |
           +-------------------------------+
           | +add_book(book)               |
           | +remove_book(book)            |
           | +borrow_book(std_id,book_id)  |
           | +return_book(std_id,book_id)  |
           | +view_all_books()             |
           +-----------+-------------------+
                       ^  ^  ^  
      uses             |  |  | 
                       |  |  |
         +-------------+  +----------------------+
         |                                   uses|
+------------------------+          +------------------------+
|       Student          |          |        Book            |
+------------------------+          +------------------------+
| - student_id           |          | - id                   |
| - student_name         |          | - name                 |
| - student_class        |          | - author               |
| - library: Library     |          | - price                |
+------------------------+          | - is_available         |
| +borrow_books(book_id) |          +------------------------+
| +return_books(book_id) |
| +view_books()          |
+------------------------+


```

Code : 

```python
# Book Entity
class Book:
    def __init__(self, id, name, author, price):
        self.id = id
        self.name = name
        self.author = author
        self.price = price
        self.is_available = True

    def get_book_details(self):
        print(f"""
        Book ID     : {self.id}
        Book Name   : {self.name}
        Author      : {self.author}
        Price       : ₹{self.price}
        Status      : {'Available' if self.is_available else 'Not Available'}
        """)


# Library Class
class Library:
    def __init__(self):
        self.books = {}           # book_id -> Book
        self.book_assigned = {}   # book_id -> student_id

    # Admin operations
    def add_book(self, book: Book):
        if book.id not in self.books:
            self.books[book.id] = book
            print(f"Book '{book.name}' added to the library.")
        else:
            print("Book already exists in the library.")

    def remove_book(self, book: Book):
        if book.id in self.books:
            self.books.pop(book.id)
            print(f"Book '{book.name}' removed from the library.")
        else:
            print("Book not found in the library.")

    # Student operations
    def borrow_book(self, student_id, book_id):
        if book_id in self.books:
            book = self.books[book_id]
            if book.is_available:
                self.book_assigned[book_id] = student_id
                book.is_available = False
                print(f"Book '{book.name}' borrowed by student ID {student_id}.")
            else:
                assigned_to = self.book_assigned.get(book_id, "Unknown")
                print(f"Book already borrowed by student ID {assigned_to}.")
        else:
            print("Book does not exist.")

    def return_book(self, student_id, book_id):
        if book_id in self.book_assigned and self.book_assigned[book_id] == student_id:
            book = self.books[book_id]
            book.is_available = True
            self.book_assigned.pop(book_id)
            print(f"Book '{book.name}' returned by student ID {student_id}.")
        else:
            print("This student did not borrow this book or book was never borrowed.")

    def view_all_books(self):
        print("===== Library Books =====")
        if not self.books:
            print("No books available in the library.")
        for book in self.books.values():
            book.get_book_details()


# Admin Role
class Admin:
    def __init__(self, name, library: Library):
        self.name = name
        self.library = library

    def add_new_book(self, id, name, author, price):
        book = Book(id, name, author, price)
        self.library.add_book(book)

    def delete_book(self, book_id):
        if book_id in self.library.books:
            book = self.library.books[book_id]
            self.library.remove_book(book)
        else:
            print("Cannot delete. Book not found.")


# Student Role
class Student:
    def __init__(self, std_id, std_name, std_class, library: Library):
        self.student_id = std_id
        self.student_name = std_name
        self.student_class = std_class
        self.library = library

    def borrow_books(self, book_id):
        self.library.borrow_book(self.student_id, book_id)

    def return_books(self, book_id):
        self.library.return_book(self.student_id, book_id)

    def view_books(self):
        self.library.view_all_books()

# ------------------- Usage Example -------------------
if __name__ == "__main__":
    # Create Library
    central_library = Library()

    # Admin adds books
    admin = Admin("Mr. Sharma", central_library)
    admin.add_new_book(1, "Python 101", "Guido van Rossum", 500)
    admin.add_new_book(2, "Clean Code", "Robert C. Martin", 600)

    # Student borrows book
    student1 = Student(101, "Aman", "10th Grade", central_library)
    student1.view_books()
    student1.borrow_books(1)
    student1.borrow_books(2)
    student1.borrow_books(2)  # Try again

    # Return book
    student1.return_books(2)

    # Final state of books
    student1.view_books()

```
Output : 

```python
Book 'Python 101' added to the library.
Book 'Clean Code' added to the library.
===== Library Books =====

        Book ID     : 1
        Book Name   : Python 101
        Author      : Guido van Rossum
        Price       : ₹500
        Status      : Available
        

        Book ID     : 2
        Book Name   : Clean Code
        Author      : Robert C. Martin
        Price       : ₹600
        Status      : Available
        
Book 'Python 101' borrowed by student ID 101.
Book 'Clean Code' borrowed by student ID 101.
Book already borrowed by student ID 101.
Book 'Clean Code' returned by student ID 101.
===== Library Books =====

        Book ID     : 1
        Book Name   : Python 101
        Author      : Guido van Rossum
        Price       : ₹500
        Status      : Not Available
        

        Book ID     : 2
        Book Name   : Clean Code
        Author      : Robert C. Martin
        Price       : ₹600
        Status      : Available
        

Process finished with exit code 0

```
