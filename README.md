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

## Elevator System Design
### Requirements
* The system should model a building with multiple floors and a configurable number of elevators.
* Passengers can request an elevator using hall buttons (Up/Down) on each floor and select a destination floor using inside buttons once inside the elevator.
* When a hall button is pressed, the system assigns the nearest idle elevator to the requested floor.
* The elevator moves to the requested floor, opens and closes its door, and updates the display to show the current floor and direction.
* Once inside the elevator, passengers can select their destination floor using the inside panel, which queues the floor in the system.
* The system supports basic door operations (open/close) and includes a stub for emergency calling from inside the elevator.
* The architecture supports multiple elevators, making it extendable for buildings with more complex lift requirements.

### UML Diagram:
```python
+-------------------+
| <<enum>>          |
| ElevatorState     |
+-------------------+
| - MOVING_UP       |
| - MOVING_DOWN     |
| - IDLE            |
+-------------------+

+-------------------+
| <<enum>>          |
| ElevatorStatus    |
+-------------------+
| - UP              |
| - DOWN            |
+-------------------+

+-------------------+
| <<enum>>          |
| DoorState         |
+-------------------+
| - OPEN            |
| - CLOSE           |
+-------------------+

+-------------------+
| Door              |
+-------------------+
| - door_state      |
+-------------------+
| + open_door()     |
| + close_door()    |
+-------------------+

+-------------------+
| ElevatorDisplay   |
+-------------------+
| - current_floor   |
| - elevator_direction |
+-------------------+
| + update_status() |
+-------------------+

+-------------------+
| CallEmergency     |
+-------------------+
| + call_emerg()    |
+-------------------+

+------------------------+
| InsideButton           |
+------------------------+
| - lift                 |
| - call_emergency_service |
+------------------------+
| + press()              |
| + press_door_open()    |
| + press_door_close()   |
| + call_emergency()     |
+------------------------+

+-------------------+
| HallButton        |
+-------------------+
| - elevator_system |
+-------------------+
| + press_up()      |
| + press_down()    |
+-------------------+

+---------------------------+
| Lift                      |
+---------------------------+
| - lift_id                 |
| - current_floor           |
| - lift_state              |
| - lift_direction          |
| - door: Door              |
| - display: ElevatorDisplay|
| - elevator_system         |
| - inside_button: InsideButton |
+---------------------------+

+----------------------------+
| Building                   |
+----------------------------+
| - build_id                 |
| - build_address            |
| - build_floors             |
| - build_lifts: List[Lift]  |
| - elevator_system          |
+----------------------------+
| + initialize_lifts()       |
| + get_lifts()              |
+----------------------------+

+-----------------------------+
| ElevatorSystem              |
+-----------------------------+
| - building                  |
| - destination_list: List[int] |
+-----------------------------+
| + add_destination()         |
| + move_lift()               |
| + get_lift()                |
+-----------------------------+

Relationships:
-------------
Building "One" -To-> "many" Lift
Lift "One" -To-> "One" InsideButton
Lift "One" -To-> "One" ElevatorDisplay
Lift "One" -To-> "One" Door
InsideButton "One" -To-> "One" CallEmergency
InsideButton "One" -To-> "One" Lift
HallButton "One" -To-> "One" ElevatorSystem
ElevatorSystem "One" -To-> "many" Lift

```

### Code :
```python
from enum import Enum

# Enum Implementation
class ElevatorState(Enum):
    MOVING_UP = "MovingUp"
    MOVING_DOWN = "MovingDown"
    IDLE = "Idle"

class ElevatorStatus(Enum):
    UP = "Up"
    DOWN = "Down"

class DoorState(Enum):
    OPEN = "OPEN"
    CLOSE = "CLOSE"

# Door
class Door:
    def __init__(self):
        self.door_state = DoorState.CLOSE

    def open_door(self):
        self.door_state = DoorState.OPEN
        print(f"Lift Door Opened")

    def close_door(self):
        self.door_state = DoorState.CLOSE
        print(f"Lift Door Closed")

# Display
class ElevatorDisplay:
    def __init__(self):
        self.current_floor = 0
        self.elevator_direction = ElevatorStatus.UP

    def update_status(self, floor, direction):
        self.current_floor = floor
        self.elevator_direction = direction
        print(f"Floor : {self.current_floor}, Going: {self.elevator_direction}")

# Call Emergency
class CallEmergency:
    def call_emerg(self):
        print("Emergency Call Initiated")

# Buttons
class InsideButton:
    def __init__(self, lift):
        self.lift = lift
        self.call_emergency_service = CallEmergency()

    def press(self, destination_floor):
        print(f"Passenger Wants to Go : {destination_floor}")
        self.lift.elevator_system.add_destination(destination_floor)

    def press_door_open(self):
        self.lift.door.open_door()

    def press_door_close(self):
        self.lift.door.close_door()

    def call_emergency(self):
        self.call_emergency_service.call_emerg()

class HallButton:
    def __init__(self, elevator_system):
        self.elevator_system = elevator_system

    def press_up(self, current_floor):
        print(f"Up button pressed at floor {current_floor}")
        self.elevator_system.get_lift(current_floor, ElevatorState.MOVING_UP)

    def press_down(self, current_floor):
        print(f"Down button pressed at floor {current_floor}")
        self.elevator_system.get_lift(current_floor, ElevatorState.MOVING_DOWN)

# Elevator
class Lift:
    def __init__(self, lift_id, elevator_system):
        self.lift_id = lift_id
        self.current_floor = 0
        self.lift_state = ElevatorState.IDLE
        self.lift_direction = ElevatorStatus.UP
        self.door = Door()
        self.display = ElevatorDisplay()
        self.elevator_system = elevator_system
        self.inside_button = InsideButton(self)

# Building
class Building:
    def __init__(self, build_id, build_address, build_floors):
        self.build_id = build_id
        self.build_address = build_address
        self.build_floors = build_floors
        self.build_lifts = []
        self.elevator_system = None

    def initialize_lifts(self, num_lifts):
        self.elevator_system = ElevatorSystem(self)
        for i in range(num_lifts):
            lift = Lift(i + 1, self.elevator_system)
            self.build_lifts.append(lift)

    def get_lifts(self):
        return self.build_lifts

# Elevator System
class ElevatorSystem:
    def __init__(self, building):
        self.building = building
        self.destination_list = []

    def add_destination(self, destination_floor):
        if destination_floor not in self.destination_list:
            self.destination_list.append(destination_floor)
            self.destination_list.sort()
            print(f"Destinations: {self.destination_list}")

    def move_lift(self, lift, floor):
        if floor > lift.current_floor:
            lift.lift_state = ElevatorState.MOVING_UP
            lift.lift_direction = ElevatorStatus.UP
        elif floor < lift.current_floor:
            lift.lift_state = ElevatorState.MOVING_DOWN
            lift.lift_direction = ElevatorStatus.DOWN
        else:
            lift.lift_state = ElevatorState.IDLE

        lift.current_floor = floor
        lift.display.update_status(floor, lift.lift_direction)
        lift.door.open_door()
        lift.door.close_door()
        lift.lift_state = ElevatorState.IDLE

    def get_lift(self, current_floor, direction):
        available_lifts = self.building.get_lifts()
        chosen = None
        min_distance = float('inf')
        for lift in available_lifts:
            if lift.lift_state == ElevatorState.IDLE:
                distance = abs(lift.current_floor - current_floor)
                if distance < min_distance:
                    min_distance = distance
                    chosen = lift
        if chosen:
            print(f"Lift {chosen.lift_id} assigned to floor {current_floor}")
            self.move_lift(chosen, current_floor)
            return chosen
        else:
            print("No idle lift available. Please wait.")
            return None

if __name__ == "__main__":
    # Create a building with 5 floors
    building = Building(build_id=1, build_address="MG Road", build_floors=5)
    building.initialize_lifts(num_lifts=2)

    # Get elevator system from building
    elevator_system = building.elevator_system

    # Create hall button (outside lift)
    hall_button = HallButton(elevator_system)

    # Simulate passenger pressing up button at floor 2
    hall_button.press_up(2)

    # Simulate someone inside lift 1 pressing to go to floor 5
    lift_1 = building.get_lifts()[0]
    lift_1.inside_button.press(5)

    # Elevator system moves lift 1 to destination floor
    elevator_system.move_lift(lift_1, 5)
```
