# Credit
I would like everybody to know, that this project is not only mine, but each contributor (especially [@BelKed](https://github.com/BelKed)) has provided a great amount of help. It would not be in its current state without everyone's contributions. Thank you very much!

# `edupage-api`
[![CodeFactor](https://www.codefactor.io/repository/github/ivanhrabcak/edupage-api/badge)](https://www.codefactor.io/repository/github/ivanhrabcak/edupage-api) 

This python library allows easy access to EduPage. This is not a Selenium web scraper. 
It makes requests directly to EduPage's endpoints and parses the HTML document.

If you find any issue with this code, it doesn't work, or you have a suggestion, please, let me know by opening an [issue](https://github.com/ivanhrabcak/edupage-api/issues/new/choose)!

If you, even better, have fixed the issue, added a new feature, or made something work better, please, open a [pull request](https://github.com/ivanhrabcak/edupage-api/compare)!

# Installing
You can install this library with pip:

```
pip install edupage-api
```

# Usage
## Login
You can log in easily, works with any school:

```python
from edupage_api import Edupage, BadCredentialsException, LoginDataParsingException

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")

try:
    edupage.login()
except BadCredentialsException:
    print("Wrong username or password!")
except LoginDataParsingException:
    print("Try again or open an issue!")
```

## Timetable
### Check all available timetable dates
```python
from edupage_api import Edupage

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Get dates for all available timetables
dates = edupage.get_available_timetable_dates()
print(f"Available timetable dates: {dates}") # ['2021-02-03', '2021-02-04']
```

### Get the timetable
```python
from edupage_api import Edupage, EduDate

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Get today's date
today = EduDate.today() # '2021-02-03'
print(f"Today's date: {today}")

timetable = edupage.get_timetable(today) # returns EduTimetable

# Print each lesson (as dict)
print("Today's lessons:")
for lesson in timetable:
    print(lesson)
print()

# Get yesterday's date
yesterday = EduDate.yesterday_this_time() # '2021-02-04'
print(f"Yesterday's date: {yesterday}")

# This will return None, because the timetable from yesterday is not available
timetable_for_yesterday = edupage.get_timetable(yesterday)
print(f"Timetable for yesterday: {timetable_for_yesterday}")
```

### Get the first lesson
```python
from edupage_api import Edupage, EduDate

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Timetable for today
timetable = edupage.get_timetable(EduDate.today())

# Get first lesson
first_lesson = timetable.get_first_lesson()
print(f"First lesson: {first_lesson}")

# The starting and ending time of the first lesson
start_time = first_lesson.length.start
end_time = first_lesson.length.end

print(f"Start time: {start_time}")
print(f"End time: {end_time}")
```

### Get current lesson for a given time
```python
from edupage_api import Edupage, EduDate, EduTime

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Timetable for today
timetable = edupage.get_timetable(EduDate.today())

# Get current time
current_time = EduTime.now()

current_lesson = timetable.get_lesson_at_time(current_time)
print(f"Current lesson: {current_lesson}")
```

### Get next lesson for a given time
```python
from edupage_api import Edupage, EduDate, EduTime

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Timetable for today
timetable = edupage.get_timetable(EduDate.today())

# Get current time
current_time = EduTime.now()

next_lesson = timetable.get_next_lesson_at_time(current_time)
print(f"Next lesson: {next_lesson}")
```

The `EduLesson` class provides some information about the lesson:

#### `EduLesson`: 
- `period`: The order of period in timetable (e.g. 1).
- `name`: The subject of this lesson.
- `teacher`: The teacher that will teach this lesson
- `classroom`: The classroom number where the lesson will be.
- `length`: `EduLength` –> The length (start and end times) of the lesson.
- `online_lesson_link`: A string with link to the online lesson. If this lesson is not online, `online_lesson_link` is `None`.

### Tell EduPage that you are on an online lesson
Useful for automating your presence, because you don't actually have to be on the lesson.

You can tell EduPage that you are on the current lesson like this:

```python
from edupage_api import Edupage, LessonUtil

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

timetable = edupage.get_timetable(EduDate.today())
next_lesson = timetable.get_next_lesson_at_time(EduTime.now())

if LessonUtil.is_online_lesson(next_lesson):
    next_lesson.sign_into_lesson(edupage)
    print("You are now 'present' on the next lesson!")
else:
    print("The next lesson is not an online lesson!")
```

## Get notifications
```python
from edupage_api import Edupage

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

notifications = edupage.get_notifications()

for notification in notifications:
    print(f"{notification.date_added} | {notification.event_type}: {notification.text}")
```

The `EduNotification` class provides some more information about the notifications:

#### `EduNotification`
- `id`: An internal Edupage ID, which can be used to find the event corresponding to this notification. Useless for now.
- `event_type`: Type of notification. Currently, we have 6 types:
    + `NotificationType.MESSAGE`
    + `NotificationType.HOMEWORK`
    + `NotificationType.GRADE`
    + `NotificationType.SUBSTITUTION`
    + `NotificationType.TIMETABLE`
    + `NotificationType.EVENT`
- `author`: Author of notification.
- `recipient`: Recipient of notification. It can be, for example, whole school, some class, current student, ...
- `text`: Text of notification.
- `date_added`: `EduExactDateTime` –> When was this notification published.
- `attachments`: List of `EduAttachment` objects.
- `subject`: The subject which this notification is from.
- `due_date`: `EduDate` –> **Just for homeworks!** When the notification (homework) is due.
- `grade`: `EduGrade` –> **Just for grades!** Grade of notification.
- `start`: `EduDate` –> **Just for events!** Start date of event.
- `end`: `EduDate` –> **Just for events!** End date of event.
- `event_type_name`: **Just for events!** Type of event. For example: Holiday, Distance education, Short test, Exam, ...

## Get news from the webpage
Thanks to how EduPage's message system works, you can get recent news from the webpage like this:

```python
from edupage_api import Edupage

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

# Note: if you are not logged in or there was an error, get_news returns None
news = edupage.get_news() # returns a list of EduNews

for message in news:
    print(str(message))
```

## Get a list of students
This is an EduPage-curated list of students. When students enter the school, they get assigned a number. If anybody changes school, leaves or anything happens with any student, the numbers don't change. It just skips the number.

```python
from edupage_api import Edupage, EduStudent

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

students = edupage.get_students() # This list is usually sorted alphabetically

# Sort the list by student's numbers
students.sort(key = EduStudent.__sort__)

for student in students:
    print(f"{student.number_in_class}: {student.fullname}")
```

## Get a list of teachers
```python
from edupage_api import Edupage, EduTeacher

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

teachers = edupage.get_teachers() # This list is usually sorted alphabetically

# Sort the list by teacher's numbers
teachers.sort(key = EduTeacher.__sort__)

for teacher in teachers:
    print(f"{teacher.id}: {teacher.fullname}")
```

## Get homework
```python
from edupage_api import Edupage

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")  
edupage.login()

homework = edupage.get_homework()

for hw in homework:
    print(f"{hw.due_date} | {hw.subject}: {hw.title}")
```

Homework, other than its title and description, provides some more information:

#### `EduHomework`
- `due_date`: `EduDate` –> When the homework is due
- `subject`: The subject which this homework is from
- `groups`: If this subject is divided into groups, the target should be here. **Needs testing**
- `title`: The title of the homework message. This is usually what you in a notification in the Edupage app.
- `description`: A detailed description of the homework. (It's usually blank)
- `event_id`: A internal Edupage ID, which can be used to find the event corresponding to this homework. Useless for now.
- `datetime_added`: `EduDateTime` –> A date and time when this homework was assigned.


## Sending messages
You can send a message to one or multiple people when you have an object that extends `EduPerson`:

```python
from edupage_api import Edupage, EduStudent

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

for student in students:
    if student.fullname == "John Smith":
        # Ignore the attachments parameter, for some reason attachments do not work
        edupage.send_message(student, "Hello John!")
```

## Upload a file to Edupage's cloud
The file will be hosted forever (and for free) on Edupage's servers. The file is tied to your user account, but anybody with a link can view it.

Anyway, Edupage limits file size to 50 MB and the file can have only some extensions. All supported file extensions could be found on this [Edupage help site](https://help.edupage.org/?p=u1/u113/u132/u362/u467).

```python
from edupage_api import Edupage
from edupage_api.cloud import EduCloud

# You will need to add bigger timeout for bigger files
edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password", timeout=30)
edupage.login()

f = open("image.png", "rb")

uploaded_file = EduCloud.upload_file(edupage, f)
link = uploaded_file.get_url(edupage)

print(f"Link to your file: {link}")
```

# Lunch
You can get menu for a day like this:
```python
from edupage_api import Edupage
from edupage_api.date import EduDate

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

today = EduDate.today()
lunch_for_today = edupage.get_lunch(today)

# Now you have access to these fields:
# .date --> date of the lunch
# .served_from (EduDate) --> time when the lunch will start to be served
# .served_to (EduDate) --> time when the lunch will no longer be served
# .amount_of_foods  --> how many choices there are
# .chooseable_menus (list) --> which menus can be choosed
# .can_be_changed_until (EduDateTime) --> last date when the lunch can be changed
# .title (str) --> title of the whole lunch (usually contains information about all choices)
# .menus (list) --> list of all choices for this lunch

# menu is of type EduMenu
for menu in lunch_for_today.menus:
    print(f"Number: {menu.number}") # This field can also be None!!
    print(f"Name: {menu.name}")
    print(f"Weight: {menu.weight}")
    print(f"Allergens: {menu.allergens}")
    
    # can be None if noone rated this menu or this menu is not rateable
    # Example: Soups in my school cannot be rated
    if menu.rating: 
        print("Rating:")
        print(f"        Quantity: {menu.rating.quantity_average}")
        print(f"        Quality:  {menu.rating.quality_average}")
    else:
        print("No rating!")
    
    print("-" * 30)
```

# Rate a lunch
Here's an example of rating the first rateable menu that was served today:
```python
from edupage_api import Edupage
from edupage_api.date import EduDate

edupage = Edupage("Subdomain (Name) of your school", "Username or E-Mail", "Password")
edupage.login()

today = EduDate.today()
lunch_for_today = edupage.get_lunch(today)

first_food = None

# find the first food that is has a number -> is rateable
for menu in lunch_for_today.menus:
    if menu.number:
        first_food = menu
        break

if first_food == None:
    print("No rateable food!")

# def rate(edupage, quantity_rating, quality_rating)
was_rating_successful = first_food.rating.rate(edupage, 5, 5)

# if the rating was unsucessful, it can be because 
# the food wasn't served to you, or it's not in the
# edupage database that the food was served to you
print(was_rating_successful)
```

# Upcoming features
- [x] Lunches
- [x] Grades
- [x] Reading your own notifications
- [x] Connecting to the online lessons (with your presence being acknowledged by Edupage)
- [x] Uploading (and hosting) files on the Edupage cloud (if possible)
- [x] Writing messages to other students/teachers
- [x] Make this library available through PyPi

Feel free to suggest any other features! Just open an [issue with the *Feature Request* tag](https://github.com/ivanhrabcak/edupage-api/issues/new?labels=feature+request&template=feature_request.md&title=%5BFeature+request%5D+).
