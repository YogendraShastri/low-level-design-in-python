# low-level-design-in-python
For beginners lets see the easy LLD for Library System. it can be more complex but my objective here is to give easy LLD design so that beginner level person can easily understand. 

## Library Management System
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

## Online Course Enrollment System

### Requirements : 
**Functional Requirements**:
Admin:
* Should be able to add new courses to the catalog.
* Should be able to remove existing courses from the catalog.
* Should be able to view all available courses.

Student:
* Should be able to view the catalog of available courses.
* Should be able to enroll in a course using the course ID.
* Should be able to unenroll from a course.
* Should be able to view all courses they are currently enrolled in.

**Non-Functional Requirements**:
* Code should be modular and follow basic object-oriented principles (encapsulation, separation of concerns).
* Method names should be meaningful and self-explanatory.
* Outputs should be clear for both admin and student actions.
* No need for persistence or a database (everything runs in memory).

### Diagram
```python

"""

                            Class: Admin
                            ----------------------------------------
                            + admin_id: int
                            + admin_name: str
                            + add_courses(catalog: Catalog, course: Course): None
                            + remove_courses(catalog: Catalog, course_id: int): None
                            + list_courses(catalog: Catalog): None
                            ----------------------------------------
                                                  |
                                                  |
                                                  |
                            Class: Catalog
                            ----------------------------------------
                            - courses: Dict[int, Course]
                            + add_courses(course: Course): None
                            + remove_courses(course_id: int): None
                            + get_courses_byID(course_id: int): Course
                            + list_courses(): None
                            ----------------------------------------
                                        /                 \ 
                                       /                   \ 
                                      /                     \ 
            Class: Student                                              Class: Course
----------------------------------------                 ----------------------------------------
+ student_id: int                                                   + course_id: int
+ student_name: str                                                 + course_name: str
- enrolled_courses: Dict[int, Course]                               + course_fee: float
+ enroll_to_course(course_id: int, catalog: Catalog)                
+ un_enroll_to_course(course_id: int): None                         + __str__(): str
+ list_enrolled_courses(): None
+ list_all_courses(catalog: Catalog): None                ----------------------------------------
----------------------------------------
"""
```

**Code :**

```python
"""
Online Course Enrollment System
Actors:
    - Admin
    - Student

Classes:
    - Admin (admin_details, add_courses, remove_courses, list_courses)
    - Student (student_details, enroll_to_course, un_enroll_to_course, list_enrolled_courses, list_all_courses)
    - Catalog (add_courses, remove_courses, get_courses_byID, list_courses)
    - Courses (courses_details, __str__)
"""

class Courses:
    def __init__(self, course_id, course_name, course_price):
        self.course_id = course_id
        self.course_name = course_name
        self.course_price = course_price

    def __str__(self):
        return f"{self.course_id} : ({self.course_name}, {self.course_price} Rs.)"

class Catalog:
    def __init__(self):
        self.courses = {}

    def add_courses(self, course):
        if course.course_id not in self.courses:
            self.courses[course.course_id] = course
            print(f"Course {course.course_name} is added")
        else:
            print(f"Course Already Exists")

    def remove_course(self, course):
        if course.course_id in self.courses:
            self.courses.pop(course.course_id)
            print(f"Course {course.course_name} Removed")
        else:
            print("Course Do not Exists")

    def list_courses(self):
        print("========== List Of Courses =========")
        for course in self.courses.values():
            print(course)
        print("====================================")

    def get_course_by_id(self, course_id):
        if course_id in self.courses:
            return self.courses[course_id]
        else:
            print(f"{course_id} Not Found")

class Admin:
    def __init__(self, admin_id, catalog):
        self.admin_id = admin_id
        self.catalog = catalog

    def add_courses_to_catalog(self, course):
        self.catalog.add_courses(course)

    def remove_courses_from_catalog(self, course):
        self.catalog.remove_course(course)

    def list_courses_from_catalog(self):
        self.catalog.list_courses()


class Student:
    def __init__(self, student_id, student_name):
        self.student_id = student_id
        self.student_name = student_name
        self.enrolled_courses = {}
        self.catalog = Catalog

    def list_all_courses(self, catalog):
        print(f"Student {self.student_name} Logged in : ")
        catalog.list_courses()

    def enroll_to_course(self, course_id):
        course = catalog.get_course_by_id(course_id)
        if course_id not in self.enrolled_courses:
            self.enrolled_courses[course_id] = course
            print(f"Enrolled to Course : {course.course_name}")
        else:
            print("Course Not Found")

    def un_enroll_to_course(self, course_id):
        if course_id not in self.enrolled_courses:
            print("Not Enrolled to the Course")
        else:
            self.enrolled_courses.pop(course_id)
            print(f"Course {course_id} Un-enrolled")

    def list_enrolled_courses(self):
        print("++++++++++ Enrolled Courses ++++++++")
        for course in self.enrolled_courses.values():
            print(course)
        print("++++++++++++++++++++++++++++++++++++")

if __name__ == "__main__":
    catalog = Catalog()
    admin = Admin(1, catalog)
    course1 = Courses(101, "Python Tutorial", 499)
    course2 = Courses(102, "Pandas", 299)
    course3 = Courses(118, "System Design", 799)

    admin.add_courses_to_catalog(course1)
    admin.add_courses_to_catalog(course2)
    admin.add_courses_to_catalog(course3)

    admin.list_courses_from_catalog()

    admin.remove_courses_from_catalog(course3)
    admin.list_courses_from_catalog()
    course4 = Courses(103, "API Course", 599)
    admin.add_courses_to_catalog(course4)
    admin.add_courses_to_catalog(course3)

    student = Student(111011, "Ram")
    student.list_all_courses(catalog)
    student.enroll_to_course(103)
    student.enroll_to_course(118)
    student.list_enrolled_courses()
    student.un_enroll_to_course(103)
    student.list_enrolled_courses()
```

**Output :**

```python
Course Python Tutorial is added
Course Pandas is added
Course System Design is added
========== List Of Courses =========
101 : (Python Tutorial, 499 Rs.)
102 : (Pandas, 299 Rs.)
118 : (System Design, 799 Rs.)
====================================
Course System Design Removed
========== List Of Courses =========
101 : (Python Tutorial, 499 Rs.)
102 : (Pandas, 299 Rs.)
====================================
Course API Course is added
Course System Design is added
Student Ram Logged in : 
========== List Of Courses =========
101 : (Python Tutorial, 499 Rs.)
102 : (Pandas, 299 Rs.)
103 : (API Course, 599 Rs.)
118 : (System Design, 799 Rs.)
====================================
Enrolled to Course : API Course
Enrolled to Course : System Design
++++++++++ Enrolled Courses ++++++++
103 : (API Course, 599 Rs.)
118 : (System Design, 799 Rs.)
++++++++++++++++++++++++++++++++++++
Course 103 Un-enrolled
++++++++++ Enrolled Courses ++++++++
118 : (System Design, 799 Rs.)
++++++++++++++++++++++++++++++++++++

Process finished with exit code 0
```
