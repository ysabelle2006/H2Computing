# Reading Files

## TXT Files

```
with open("file.txt","r") as fin:
    line = None
    data = []
    while line != "":
        line = fin.readline().strip()
        if line != "":
            data.append(line)

# FILE CLOSES AUTOMATICALLY
```
<br>

## CSV Files 

```
with open("file.csv","r") as fin:
    line = fin.readline()
    data = []
    while line != "":
        line = fin.readline().strip()
        if line != "":
            a,b = line.split(",")
            data.append((a,b))

# FILE CLOSES AUTOMATICALLY
```
<br>

## JSON Files

```
import json
with open("file.json","r") as fin:
    data = json.load(fin)

# FILE CLOSES AUTOMATICALLY
```

<br>
<br>

# Socket Programming

## Client Side
```
import socket

host = "127.0.0.1"
port = 8000
eol = "\n"

mysocket = socket.socket()
mysocket.connect((host,port))

endCon = False

while endCon == False:

    data = ""
    while eol not in data:
        data += mysocket.recv(1024).decode()

    if "end" in data:
        endCon = True
        msg = "end"
    else:
        print(data)
        msg = input("enter client: ")
        
    msg = (msg + eol).encode()
    mysocket.sendall(msg)

mysocket.close()

```

## Server Side
```
import socket

host = "127.0.0.1"
port = 8000
eol = "\n"

mysocket = socket.socket()
mysocket.bind((host,port))
mysocket.listen()
newsocket, address = mysocket.accept()

endCon = False

while endCon == False:

    msg = input("enter server: ")
    msg = (msg + eol).encode() 
    newsocket.sendall(msg)

    data = ""
    while eol not in data:
        data += newsocket.recv(1024).decode()

    # Code continues...

    if "end" in data:
        endCon = True
        newsocket.sendall(("end"+eol).encode()) 
    else:
        print(data)

newsocket.close()
mysocket.close()
```

<br><br>

# MongoDB

<strong>RUN THE MONGO SERVER EXE BEFORE ANY NOSQL QUESTION</strong>

## Creating a New DB

```
from pymongo import MongoClient

client = MongoClient("localhost",127017) # port number can be found when opening Mongod.exe (server)
db = client["dbname"]
col = db["colname"]
col.insert_many(data) # data is a list of dictionaries where each dictionary key is a data type in the row

# To print out new db records
out = col.find({},{"_id":0})
print(out)
client.close()
```

## Reading information in a new DB
```
from pymongo import MongoClient

client = MongoClient("localhost",127017)
db = client["dbname"]
col = db["colname"]

# Query & Projections (exact match)
out = col.find({"local":10},{"_id":0,"height":1})

# Query & Projections (if exists and if greater than 10)
out = col.find({"local":{"$exists":True, "$gte":10}},{"_id":0,"height":0})

# Query & Projections (Use of OR operator with not equals)
out = col.find({"$or":[{"local":False},{"height":{"$ne":10}}]}, {})

# Query & Projections (Use of in operator with list)
out = col.find({"class":{"$in":["23S60","23S61"]}}, {})



# Queries for Embedded Documents

embeddedDoc = {"name":{"sirname":"Wong","firstname":"Ysabelle"}}

#Query Exact Match
col.find({"name":{"sirname":"Wong","firstname":"Ysabelle"}})

# Query based on 1 item in the EMBEDDED document
col.find({"name.sirname":"Wong"})
col.find({"name.firstname":"Ysabelle"})



# Queries for Documents with Arrays

arrayDoc = {"subjects":["CP","MA","PH"]}

# Query will return all documents where subjects contain "CP" in the list
col.find({"subjects":"CP"})

# Query will return all documents where subjects contain at least one element in ["CP","CH"] (need not be both)
col.find({"subjects":{"$in":["CP","CH"]}})



# Queries for Documents with Documents in Arrays

embeddedArrayDoc = {"vaccination":[{"date":17122006,"manufacturer":"Moderna"}, {"date":01031975, "manufacturer":Pfizer}]}

# Query will return all documents that in the array vaccination manufacturer is in ["Pfizer","AstraZeneca"] at least once

col.find({"vaccination.manufacturer":{"$in":["Pfizer","AstraZeneca"]}})


# Meant for multiple queries in the array
# Query will return all documents where in the array vaccination, the date exists and the manufacturer is Pfizer

col.find({"vaccination":{"$elemMatch":[{"date":{"$exists":True}}, {"manufacturer":"Pfizer"}]}})



# There are times when sorting and limiting the number of records is required

col.find({}).limit(3) # This limits the records outputted to 3

col.find({}).sort([("name",1), ("class",-1)]) # This sorts the records first in ascending order by name then for those that tie, it will sort by class in descending order
```

## Inserting New Documents
```
data = [{"gender":F, "name":"Ysabelle"},{"name":"Hana","age":17}]
inserted = col.insert_many(data)

data = {"name":"Hana","age":17}
inserted = col.insert_one(data)

# If number / IDs of inserted documents needed, then create a variable and store the return value of insert_many or insert_one. The variable inserted contains the id / list of ids of documents that were just inserted

inserted.inserted_id # For inserted_one
inserted.inserted_ids # For inserted_many --> returns list of ids

# Mainly use for counting number of inserted documents or to make further changes after that
```

## Updating Documents
```
# Only changes for the first document found
# For set, if the category did not exist at first
# If upsert is True, if the document is not found and nothing was updated, it will create a new document with the value of set

modified = col.update_one({"local":False},{"$set":{"name":"Tan"}},upsert=True) 

# To remove one field
modified = col.update_many({"local":False},{"$unset":"hello"}) 

# To remove multiple fields
modified = col.update_many({"local":False},{"$unset":["hello","name"]}) 

# To find number of modified documents:
count = modified.modified_count 

# To add current date/timestamp to a field
col.update_many({},{"$currentDate":{"fieldName":{"$type":"date"}}}) 

col.update_many({},{"$currentDate":{"fieldName":{"$type":"timestamp"}}}) 

# To increase the value of a field by a certain amount
numToIncrease = 1
col.update_many({},{"$inc":{"fieldName":numToIncrease}})

# no commit statement needed
```

## Deleting Documents
```
# Able to delete either the first record or all records that met the query
deleted = col.delete_one({"status":"reject"})
deleted = col.delete_many({"status":"reject"})

# Able to get number of records deleted
print(deleted.deleted_count)
```

# Sqlite3

## Setting up Database
```
import sqlite3

# Creating database in .sql (ie not in Python - do not run this in Python)
sql_statement = "CREATE DATABASE database"

# In Python
con = sqlite3.connect("database.db") # this creates the database if does not exist

# useful when testing to prevent table already exists error
con.execute("DROP TABLE IF EXISTS Student")

# may use """ """ or use \ to break line when writing SQL statements
con.execute("""CREATE TABLE Student(StudentID INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, Name TEXT(200) NOT NULL,
TeacherID INTEGER NOT NULL, FOREIGN KEY(TeacherID) REFERENCES Teacher(TeacherID))""")

# use ? to substitute in the values to prevent SQL injection
# if there is only 1 variable to sub in, still need to put , after the variable because tuple syntax
con.execute("""INSERT INTO Student(Name, TeacherID) VALUES (?,?)""",(name,teacherID))

con.commit() #needed when modifying table
con.close()
```

## Inserting Documents (requires row ID)
```
import sqlite3

con = sqlite3.connect("database.db")
cur = con.cursor()
cur.execute("""INSERT INTO Student(Name, TeacherID) VALUES (?,?)""",(name,teacherID))
newest_insert = cur.lastrowid # also works for last updated row -> only for autoincrement columns

cur.close()
con.commit()
con.close()
```

## Selecting Documents

### Without Cursor
```
import sqlite3

con = sqlite3.connect("database.db")
rows = con.execute("SELECT StudentID, Name FROM Student WHERE TeacherID = ?",(teacherID,))

data = {}
for row in rows: # even if there is only one value, also need to have for loop
    data["studentid"] = row[0] # index of row follows the order of column name in SELECT statement
    data["name"] = row[1]

#ALTERNATIVELY:
data = rows.fetchall() # will return a nested list (inner list is for all the columns and outer list is the records)

con.close()
# no commit needed as no changes made
```

### With Cursor (and row factory)
```
import sqlite3

con = sqlite3.connect("database.db")
con.row_factory = sqlite3.Row
cur = con.cursor()
rows = cur.execute("SELECT StudentID, Name FROM Student WHERE TeacherID = ?",(teacherID,))

data = rows.fetchall() # will return a list of dictionaries (the dictionary key is the column name)

cur.close()
con.close()
# no commit needed as no changes made

```

# Searching & Sorting Algorithms

## Searching

### Linear Search

Time Complexity: O(1) [minimum] & O(n) [maximum]
<br>

#### Algorithm:

1. Iterate through list to check element against target

2. Once target is found, either continue finding next instance of target or end loop and return index

3. If not found, it will return -1 / empty list after completing the entire iteration
<br>
<br>

```
unsortedList = [31, 99, 146, 167, 88, 12, 145, 65, 7, 159, 106, 27, 134, 113, 42, 130, 149, 171, 92, 7, 147, 142, 123, 31, 120, 53, 122, 46, 135, 192, 50, 120, 194, 151, 130, 58, 123, 190, 82, 144, 149, 191, 149, 163, 4, 118, 174, 106, 16, 64, 100, 121, 69, 43, 154, 129, 191, 172, 193, 142, 160, 173, 19, 154, 182, 200, 27, 45, 56, 186, 76, 125, 78, 9, 39, 63, 162, 23, 101, 94, 100, 137, 28, 69, 85, 46, 176, 2, 83, 187, 5, 129, 47, 143, 57, 145, 79, 197, 154, 183]

target = 7

def linearSearchSingle(target,unsortedList):
    for i in range(len(unsortedList)):
        if unsortedList[i] == target:
            return i 

    return -1

def linearSearchMultiple(target,unsortedList):
    out = []
    for i in range(len(unsortedList)):
        if unsortedList[i] == target:
            out.append(i)

    return out
```
<br>

### Binary Search

Time Complexity: O(1) [minimum] & O(log n) [maximum]

Note: Binary search only works for a sorted list
<br>

#### Algorithm:

1. Set low to index of first element to search (start: 0)

2. Set high to index of last element to search (start: len(list)-1)

3. Find midpoint of list mid = (low + high) // 2

4. If midpoint > target then call function again where low = mid + 1 and high remains at high

5. If midpoint < target then call function again where high = mid - 1

6. Repeat until either midpoint = pivot or low > high

    - If finding multiple, once find one instance, do while loop forward and backward to find boundaries then return 

7. If low > high, it means target not found so it will return -1
<br>
<br>

```
def binarySearchRecursiveSingle(sortedList,target,low,high):
    if low <= high:
        mid = (low + high) // 2
        if sortedList[mid] == target:
            return mid
        elif sortedList[mid] < target:
            
            return binarySearchRecursiveSingle(sortedList,target,mid+1,high)
        else:
            return binarySearchRecursiveSingle(sortedList,target,low,mid-1)
    else:
        return -1

def binarySearchRecursiveMultiple(sortedList,target,out,low,high):
    if low <= high:
        mid = (low + high) // 2
        if sortedList[mid] == target and mid not in out:
            upper = mid
            lower = mid
            while upper + 1 < len(sortedList) and sortedList[upper+1] == target:
                upper += 1
            while lower - 1 >= 0 and sortedList[lower - 1] == target:
                lower -= 1
            for i in range(lower,upper+1):
                out.append(i) 
                
            return out
            
        elif sortedList[mid] < target:
            return binarySearchRecursiveMultiple(sortedList,target,out,mid+1,high)
        else:
            return binarySearchRecursiveMultiple(sortedList,target,out,low,mid-1)

    return out

def binarySearchMultiple(sortedList,target):
    low = 0
    high = len(sortedList) - 1
    out = []
    while low <= high:
        mid = (low + high) // 2
        if sortedList[mid] == target:
            upper = mid
            lower = mid
            while upper + 1 < len(sortedList) and sortedList[upper+1] == target:
                upper += 1
            while lower - 1 >= 0 and sortedList[lower - 1] == target:
                lower -= 1
            for i in range(lower,upper+1):
                out.append(i)
            return out
            
        elif sortedList[mid] > target:
            high = mid - 1
        else:
            low = mid + 1

    return out

def binarySearchSingle(sortedList,target):
    low = 0
    high = len(sortedList) - 1
    while low <= high:
        mid = (low + high) // 2
        if sortedList[mid] == target:
            return mid
        elif sortedList[mid] > target:
            high = mid - 1
        else:
            low = mid + 1
    return -1
```
<br>

## Sorting
### Bubble Sort

Time Complexity: O(n) [minimum] & O(n^2) [maximum]
Space Complexity: O(1)


#### Algorithm

1. While there is still sorting in that round of the list (max is len(var)-1), iterate through the list (up to len(var)-1-round)

2. For each element, compare it with the element at the next index. If cur > next then swap their positions and continue

3. If an entire iteration is done without any swapping, the entire programme can end

<br>

```
def bubbleSort(unsortedList):
    round = 0
    end = False

    while round < len(unsortedList) - 1 and end == False:
        end = True
        for i in range(len(unsortedList)-1-round):
            if unsortedList[i] > unsortedList[i+1]:
                temp = unsortedList[i]
                unsortedList[i] = unsortedList[i+1]
                unsortedList[i+1] = temp
                end = False

        round += 1
```
<br>

### Insertion Sort

Time Complexity: O(n) [minimum] & O(n^2) 
Space Complexity: O(1)

#### Algorithm

1. Start from second index (because one number is already sorted)

2. Iterate from second index to end (cur - i)

3. Iterate from start to i (check - j)

4. If cur < check, shift all numbers from j to i by 1 index to the right and put cur at index j

<br>

```
def insertionSort(unsortedList):
    for i in range(1,len(unsortedList)):
        j = 0
        while j < i:
            if unsortedList[i] < unsortedList[j]:
                shift(unsortedList,j,i)
            j += 1


def shift(arr,start,end):
    newstart = arr[end]
    for i in range(end,start,-1):
        arr[i] = arr[i-1]
    arr[start] = newstart
```

<br>

### Quick Sort

Time Complexity: O(n log n) [minimum] & O(n^2) [maximum]

#### Algorithm

1. If len(list) is 0 or 1 just return the list

2. Else, it means there is something to sort and sorting continues

3. Find the mid of the list where mid = (start+end) // 2. The value at this point is the pivot

4. Swap the first and mid value

5. Set the left pointer as the second index and right pointer as the last index

6. Start from the second index and iterate until the left pointer is greater than the right pointer index or the value of left pointer is greater than pivot

7. Start from the end index and iterature until the value at the right pointer is less than or equals to the pivot

8. If the left pointer is still less than the right pointer, swap the values of the left and right pointer and continue from Step 6

9. Once the lp > rp, swap the start and rp values 

10. Call the function again for the list before and after the rp index

<br>

```
def quickSort(unsortedList,low,high):

    if low < high:
        mid = (low+high) // 2
        pivot = unsortedList[mid]
        unsortedList[mid] = unsortedList[low]
        unsortedList[low] = pivot

        lp = low + 1
        rp = high

        while lp <= rp:

            while lp <= rp and unsortedList[lp] <= pivot:
                lp += 1

            while unsortedList[rp] > pivot:
                rp -= 1

            if lp < rp:
                temp = unsortedList[lp]
                unsortedList[lp] = unsortedList[rp]
                unsortedList[rp] = temp

        unsortedList[low] = unsortedList[rp]
        unsortedList[rp] = pivot
        quickSort(unsortedList,low,rp-1)
        quickSort(unsortedList,rp+1,high)

def quickSortNIP(unsortedList):
    if len(unsortedList) == 0 or len(unsortedList) == 1:
        return unsortedList
    
    mid = (len(unsortedList)-1) // 2
    pivot = unsortedList[mid]
    unsortedList[mid] = unsortedList[0]
    unsortedList[0] = pivot

    left = []
    right = []

    lp = 1
    rp = len(unsortedList) - 1

    while lp <= rp:

        while lp <= rp and unsortedList[lp] <= pivot:
            left.append(unsortedList[lp])
            lp += 1

        while unsortedList[rp] > pivot:
            right.append(unsortedList[rp])
            rp -= 1

        if lp < rp:
            temp = unsortedList[lp]
            unsortedList[lp] = unsortedList[rp]
            unsortedList[rp] = temp

    unsortedList[0] = unsortedList[rp]
    unsortedList[rp] = pivot
    return quickSortNIP(unsortedList[:rp]) + [pivot] + quickSortNIP(unsortedList[rp+1:])
```
<br>

### Merge Sort

Time Complexity: O(n log n) [minimum] & O(n log n) [maximum]

#### Algorithm

1. Split the list into two lists until each list is a single element

2. For each of the 2 lists merge them using the algorithm where both start from left and whichever element is bigger gets appended first until all are appended

3. Merged list is then returned to the list before it and the merging continues

<br>

```
def split(unsortedList):
    if len(unsortedList) == 1:
        return unsortedList
    
    mid = len(unsortedList) // 2
    return merge(split(unsortedList[:mid]) ,split(unsortedList[mid:]))

def merge(left,right):
    lp = 0
    rp = 0

    endLeft = False
    endRight = False

    combined = []

    while endLeft == False and endRight == False:
        if left[lp] >= right[rp]:
            combined.append(right[rp])
            rp += 1
        else:
            combined.append(left[lp])
            lp += 1

        if lp >= len(left):
            endLeft = True
        elif rp >= len(right):
            endRight = True

    if endLeft == True:
        combined += right[rp:]
    else:
        combined += left[lp:]

    return combined
```

<br>

# Data Structures

## Linked List

Compared to an array, a linked list uses <strong>dynamic memory allocation</strong>. This means that the number of elements that can be stored in a linked list is only limited by the computer's memory and can be dynamically increased or decreased. However, linked lists <strong>do not support random access</strong> unlike arrays so traversal is done from the root node to the last node to find an element.

### Creation

1. Set original ptr = None
2. For loop to count with ptr = Node(data,ptr)
3. Set root to ptr after for loop ends

<br>

```
ptr = None
for i in range(5):
    ptr = Node(5-i,ptr)
root = ptr
```

### Traversal

Algorithm:

1. Set pointer as root
2. Let pointer = pointer.next until pointer is None (or until alternative condition is met)

### Insertion (by position)

Special Cases:

1. When LinkedList is empty or index == 1, insertion is as root (ie self._root = newNode)

2. For other cases, iterate to the pointer before the index, set newNode._next to be ptr._next then set ptr._next to be newNode

For insertion by data, it is possible that it comes out (e.g. insert words in alphabetical order), for these types, it has the same cases just that now it is less simple to tell if the node is supposed to be at which position, so prior checks are necessary

### Deletion (by data and index)

Special Cases:

1. Deleting in an empty list
2. Deleting the root of a list
3. Deleting other items in a list

Note: for (1), it can be lumped together with error indexes such as indexes less than or equals to 0 and indexes greater than length of list

Deletion by data becomes easier by combining deletion by index and search function whereby you first search for the index (if it does not exist returns -1) then pass it thorugh deletebyPos

If instead of delete, they need pop then just return the deleted value which can be found easily and returned

### Replacement (by data and index)

For replacement by index:

1. Check if index is valid (within count and more than 0)

2. Traverse list until index is reached and change the data

<br>

For replacement by data: Similar to deletion by data use search function with replacement by index to replace with data

### Searching (by data)

1. Traverse through linked list until either target is found or end of linked list (ie when ptr = None) while keeping count of the number of nodes traversed

2. If target is found, return the count

3. If not, it means element not found in linked list and raise error (eg returning -1)

If searching by index is needed, simply traverse until the count is the same as the index and return the data at that pointer

<br>

```
class Node:

    def __init__(self,data=None,Next=None):
        self._data = data
        self._next = Next

    def __str__(self):
        if self._next != None:
            nextData = self._next._data
        else:
            nextData = "None"
        return f"Data: {self._data} | Next: {nextData}"

class LinkedList:
    def __init__(self,root=None):
        self._root = root

    def count(self):
        ptr = self._root
        count = 0
        while ptr != None:
            count += 1
            ptr = ptr._next

        return count

    def __str__(self):
        ptr = self._root
        if ptr == None:
            return "None"
        else:
            out = ""
            while ptr != None:
                out += str(ptr) + " "
                ptr = ptr._next

            return out

    def insert(self,data,index):
        new = Node(data)
        if index == 1 or self._root == None:
            new._next = self._root
            self._root = new
        else:
            if index >= self.count() + 1:
                index = self.count() + 1

            cur = 1
            ptr = self._root

            while  cur < index - 1:
                ptr = ptr._next
                cur += 1

            new._next = ptr._next
            ptr._next = new

    def search(self,data):
        ptr = self._root
        count = 1
        end = False
        while end == False:
            if ptr._data == data:
                return count
            
            if ptr._next == None:
                end = True
            else:
                count += 1
                ptr = ptr._next

        return -1
    
    def deletebyPos(self,index):
        if index > self.count() or index <= 0:
            print("Delete failed, element not found in list")
        else:
            if index == 1:
                self._root = self._root._next
            else:
                cur = 1
                ptr = self._root
                while cur != index-1:
                    ptr = ptr._next
                    cur += 1

                    ptr._next = ptr._next._next

    def deletebyData(self,data):
        self.deletebyPos(self.search(data))

    def replace(self,index,data):
        if index > self.count() or index <= 0:
            print("Replace failed, element not found in list")

        else:
            cur = 1
            ptr = self._root
            while cur < index:
                cur += 1
                ptr = ptr._next

            ptr._data = data

    def replacebyData(self,target,data):
        self.replace(self.search(target),data)
```

## Queue

Queue follows a First In First Out (FIFO) system. The first element to be enqueued will also be the first element to be dequeued and vice versa. (It can be implemented in terms of an array or linked list)

Special Cases:

- Enqueue: Adding to a full list (only for array)
- Dequeue: Removing from an empty list

<br>

For Array:
- After enqueue, change end to end + 1

- After dequeue, change start to start + 1

- Both end and start work on circular system (ie if there is space at the front the indexes are expected to go back to the front)

- To check if full or empty, maintain a count

<br>

```
# Using linked list

class Node:

    def __init__(self,data=None,Next=None):
        self._data = data
        self._next = Next

    def __str__(self):
        if self._next != None:
            nextData = self._next._data
        else:
            nextData = "None"
        return f"Data: {self._data} | Next: {nextData}"

class Queue():
    def __init__(self,head=None):
        self._head = head

    def __str__(self):
        ptr = self._head
        if ptr == None:
            return "None"
        
        out = ""
        while ptr != None:
            out += str(ptr) + " "
            ptr = ptr._next

        return out

    def count(self):
        count = 0
        ptr = self._head
        while ptr != None:
            ptr = ptr._next
            count += 1

        return count
    
    def enqueue(self,data):
        new = Node(data)
        if self._head == None:
            self._head = new
        else:
            ptr = self._head
            while ptr._next != None:
                ptr = ptr._next

            ptr._next = new

    def dequeue(self):
        if self._head == None:
            return None
        
        out = self._head._data
        self._head = self._head._next
        return out



# Using array

class Queue():
    def __init__(self,size=10,start=0,end=-1):
        self._queue = [None] * size
        self._start = start
        self._end = end
        self._size = size
        self._cursize = 0
    
    def __str__(self):
        return str(self._queue)
    
    def enqueue(self,data):
        if self._cursize == self._size:
            print("Queue is full")
            return None
        else:
            self._end = (self._end + 1) % self._size
            self._queue[self._end] = data

    def dequeue(self):
        if self._cursize == 0:
            print("Queue is empty.")
            return None
        else:
            out = self._queue[self._start]
            self._queue[self._start] = None
            self._start = (self._start + 1) % self._size
            return out
```

## Stack

Stack follows a First In Last Out (FILO) system. The first element to be pushed will also be the last element to be popped and vice versa. (It can be implemented in terms of an array or linked list)

Special Cases:
- Pop: Popping from an empty stack
- Push: Pushing to a full stack (only for array)

<br>

```
#Using linked list

class Stack():
    def __init__(self,base=None):
        self._base = base

    def count(self):
        count = 0
        ptr = self._base
        while ptr != None:
            ptr = ptr._next
            count += 1

        return count
    
    def __str__(self):
        if self._base == None:
            return "None"
        
        ptr = self._base
        out = ""
        while ptr != None:
            out += str(ptr) + " "
            ptr = ptr._next

        return out

    def push(self,data):
        new = Node(data)
        if self._base == None:
            self._base = new
        else:
            ptr = self._base
            while ptr._next != None:
                ptr = ptr._next

            ptr._next = new

    def pop(self):
        if self._base == None:
            print("Stack is empty.")
            return None
        if self._base._next == None:
            out = self._base._data
            self._base = None
            return out
        else:
            ptr = self._base
            while ptr._next._next != None:
                ptr = ptr._next
            out = ptr._next._data
            ptr._next = None
            return out



# Using array

class Stack():
    def __init__(self,size=10):
        self._size = size
        self._top = -1
        self._stack = [None] * self._size

    def __str__(self):
        return str(self._stack)

    def push(self,data):
        if self._top == self._size - 1:
            print("Stack is full")
        
        else:
            self._top += 1
            self._stack[self._top] = data

    def pop(self):
        if self._top == -1:
            print("Stack is empty")
            return None

        else:
            out = self._stack[self._top]
            self._stack[self._top] = None
            self._top -= 1
            return out
```
<br>

### Use Cases:

- Check matching of brackets 

- Postfix / Infix / Prefix expressions
    - to evaluate mathematical expressions (convert from infix to postfix/prefix then evaluate the postfix)

#### Check Matching Brackets
```
def evalExp(exp):

    matching = {"}":"{", "]":"[",")":"(",">":"<"}

    opstack = Stack()
    for char in exp:
        if char in matching.values():
            opstack.push(char)
        elif char in matching:
            if opstack.count() != 0:
                top = opstack.pop()
                if matching[char] != top:
                    return False
            else:
                return False
            
    if opstack.count() == 0:     
        return True
    else:
        return False
```

<br>

#### To convert infix to postfix:

1. Create an empty stack called opstack

2. Iterate through the expression

3. If char is a number then add it to the output list/string

4. If char is a "(", add it to the opstack

5. If char is a ")", pop everything in the stack and add it into the output list until a "(" is popped though do not add "(" to the list

6. If char is + or -, pop all operators until a "(" and ")" is popped [make sure to put the () back] and add it to the list then push char into the stack

7. If char is a * or /, pop operators until a +, -, (, ) is popped [make sure to put +-() back] and add it to the list then push char into the stack

8. Once the iteration is done, pop all remaining operators and add it to the list

<br>

```
def infixToPostfix(infix):
    exp = infix.split(" ")
    opstack = Stack()
    postfix = ""
    for char in exp:
        if char.isdigit() == True:
            postfix += char
        elif char == "(":
            opstack.push(char)
        elif char == ")":
            end = False
            while end == False:
                cur = opstack.pop()
                if cur == "(":
                    end = True
                else:
                    postfix += cur
        else:
            if char == "*" or char == "/":
                end = False
                while end == False:
                    if opstack.count() == 0:
                        end = True
                    else:
                        cur = opstack.pop()
                        if cur == "+" or cur == "-" or cur == "(" or cur == ")":
                            opstack.push(cur)
                            end = True
                        else:
                            postfix += cur
                opstack.push(char)

            else:
                end = False
                while end == False:
                    if opstack.count() == 0:
                        end = True
                    else:
                        cur = opstack.pop()
                        if cur == "(" or cur == ")":
                            opstack.push(cur)
                            end = True
                        else:
                            postfix += cur

                opstack.push(char)

    while opstack.count() != 0:
        postfix += opstack.pop()
    
    return postfix
```

<br>

#### To evaluate a postfix expression:

1. Create an empty number stack

2. Iterate through the infix expression

3. If it is a number, add it to the stack

4. If it is an operator, pop 2 numbers from the stack and apply the operator where final = [secondpop] [operator] [firstpop]

5. Add final to the stack

6. Once iteration is finished, pop a number from the stack which is the evaluated result

<br>

```
def evalPostfix(postfix):
    numstack = Stack()
    for char in postfix:
        if char.isdigit() == True:
            numstack.push(int(char))
        else:
            bottom = numstack.pop()
            top = numstack.pop()

            if char == "+":
                final = top + bottom
            elif char == "-":
                final = top-bottom
            elif char == "*":
                final = top * bottom
            else:
                final = top - bottom

            numstack.push(final)

    eval = numstack.pop()
    return eval
```

<br>

#### To convert from infix to prefix:

1. Reverse the prefix expression

2. Create an empty stack called opstack

3. Iterate through the expression

4. If the char is a number, add it to the output list

5. If the char is a ), push it to the opstack

6. If the char is a (, pop all operators from the opstack until and add it to the output list until the ) is popped

7. If the char is a + or -, pop * and / operators (aka all operators higher then + and -) until a ()+- operator is met and add to output, then add char to the stack

8. If the char is a * or /, add to stack (this is because there are no operators higher than * and /)

9. Once iteration is done, add all remaining operators in stack to the output list

10. Reverse the output list before returning

<br>

```
def infixToPrefix(infix):
    exp = infix.split(" ")[::-1]
    opstack = Stack()
    prefix = ""
    for char in exp:
        if char.isdigit() == True:
            prefix += char
        elif char == ")":
            opstack.push(char)
        elif char == "(":
            end = False
            while end == False:
                cur = opstack.pop()
                if cur == ")":
                    end = True
                else:
                    prefix += cur

        elif char == "+" or char == "-":
            end = False
            while end == False:
                if opstack.count() == 0:
                    end = True
                else:
                    cur = opstack.pop()
                    if cur == "(" or cur == ")" or cur == "+" or cur == "-":
                        opstack.push(cur)
                        end = True
                    else:
                        prefix += cur

            opstack.push(char)

        else:
            opstack.push(char)

    while opstack.count() != 0:
        prefix += opstack.pop()

    prefix = prefix[::-1]
    return prefix
```

<br>

#### To evaluate a prefix:

1. Reverse the prefix 

2. Create an empty stack called numstack

3. Iterate through the prefix

4. If char is a number, add it to numstack

5. If char is an operator, pop 2 numbers from numstack

6. Calculate final where final = [firstpop] [operator] [secondpop]

7. Push final into numstack

8. Once iteration is complete, pop the result from numstack

<br>

```
def evalPrefix(prefix):
    prefix = prefix[::-1]
    number = Stack()
    for char in prefix:
        if char.isdigit() == True:
            number.push(int(char))
        else:
            top = number.pop()
            bottom = number.pop()
            if char == "+":
                final = top + bottom
            elif char == "-":
                final = top - bottom
            elif char == "*":
                final = top * bottom
            else:
                final = top / bottom

            number.push(final)

    result = number.pop()
    return result
```

## Binary Search Tree

Binary search tree follows a system whereby the first element passed in is the root. Each node (including root) only has left and right pointers with its own data. For subsequent elements, if the next element is bigger than the root, it will go to the right. If smaller, it will go to the left. If the right/left is already taken, it will continue to compare itself until an empty space is found. The adding of an element can be done recursively and non-recursively.

<br>

To display a binary search tree, you can use preorder, inorder or postorder.

Preorder: returns the list of numbers whereby if you add the list of numbers into an empty binary search tree, you will get back the same tree [centre -> left -> right]

1. If pointer is not none, append the current pointer data to output list

2. Then, call the function again but with ptr._left

3. Then, call the function again but with ptr._right

4. Then, return the output list 

- Note: Step 2 to 4 is still in the same if-else condition as Step 1

- The logic behind is that it will first save the root data then travel to the left one and save until it hits the bottom left which is none then it will stop and return to the previous function call where it will continue with right

Inorder: returns a sorted list of numbers in the binary search tree [left -> centre -> right]

1. If pointer is not none, call the function again but with ptr._left

2. Then, append the current pointer data to output list

3. Then, call the function again but with ptr._right

4. Then, return the output list 

- Note: Step 2 to 4 is still in the same if-else condition as Step 1

- The logic behind is that it will first travel to the extreme left node until it hits none for left then saves that data and try right (but will be none also) then go back up to the previous where it will save that data and then try right

Postorder: Returns a list of numbers whereby if you delete the numbers in that order will delete the tree (not really tested in exams as deletion is not very tested) [left -> right -> centre]

1. If pointer is not none, call the function again but with ptr._left

2. Then, call the function again but with ptr._right

3. Then, append the current pointer data to output list

4. Then, return the output list 

- Note: Step 2 to 4 is still in the same if-else condition as Step 1

- The logic behind is that it will first travel to the extreme left node until it hits none for left try right (but will be none also) then saves that data and goes up to try right branch (once done trying) then saves data and goes to previous node

<br>

```
class Node:

    def __init__(self,data=None,left=None,right=None):
        self._left = left
        self._right = right
        self._data = data

    def __str__(self):
        left = "None"
        right = "None"
        data = "None"
        if self._left != None:
            left = self._left._data
        if self._right != None:
            right = self._right._data
        if self._data != None:
            data = self._data
        return f"Data: {data} | Left: {left} | Right: {right}"


class BinarySearchTree():

    def __init__(self,root=None):
        self._root = root

    def add(self,data):
        new = Node(data)

        if self._root == None:
            self._root = new
        else:
            ptr = self._root
            end = False
            while end == False:
                if data > ptr._data:
                    if ptr._right == None:
                        ptr._right = new
                        end = True
                    else:
                        ptr = ptr._right
                else:
                    if ptr._left == None:
                        ptr._left = new
                        end = True
                    else:
                        ptr = ptr._left

    def addRecursive(self,data,ptr):
        if self._root == None:
            self._root = Node(data)
        elif data > ptr._data:
            if ptr._right == None:
                ptr._right = Node(data)
            else:
                self.addRecursive(data,ptr._right)

        else:
            if ptr._left == None:
                ptr._left = Node(data)
            else:
                self.addRecursive(data,ptr._left)

    def count(self,ptr):
        if ptr == None:
            return 0
        else:
            return self.count(ptr._left) + self.count(ptr._right) + 1

    def search(self,ptr,data):
        if ptr == None:
            return False
        elif data < ptr._data:
            return self.search(ptr._left,data)
        elif data > ptr._data:
            return self.search(ptr._right,data)
        else:
            return True
        
    def inorder(self,ptr,out=[]):
        if ptr != None:
            self.inorder(ptr._left,out)
            out.append(ptr._data)
            self.inorder(ptr._right,out)

            return out
        
    def preorder(self,ptr,out=[]):
        if ptr != None:
            out.append(ptr._data)
            self.preorder(ptr._left,out)
            self.preorder(ptr._right,out)

            return out
        
    def postorder(self,ptr,out=[]):
        if ptr != None:
            self.postorder(ptr._left,out)
            self.postorder(ptr._right,out)
            out.append(ptr._data)

            return out
```

# Flask & Web Applications

## Setting Up Flask
```
from flask import *

# If involves submission/showing of files (local)
from werkzeug.utils import secure_filename
import os

app = Flask(__name__)
# if need to change template directory use app = Flask(__name__,template_folder=new_dir)

# the first argument is the path of the website (use / for home page and change as needed)
# second argument can be omitted if only GET but when POST is needed (for form submission) add to list as needed
@app.route("/",methods=["GET"])
def index():
    return render_template("index.html") # instead of render_template can also be html code in string

app.run()
```

## Flask Applications
```
from flask import *
from werkzeug.utils import secure_filename
import os

app = Flask(__name__)

@app.route("/",methods=["GET"])
def index():
    return render_template("index.html")

# Using GET to pass in responses --> uses a form submission
@app.route("/photos",methods=["GET"]) # shown in url like ?name=hello&age=17
def photos(): #try to ensure function and website page is same name (makes it easier)
    userid = request.args.get("userid")
    filename = request.args.get("filename")

# Using POST to pass in responses --> uses a form submission
@app.route("/add",methods=["GET","POST"])
def add():
    if request.method == "GET": # request.method allows to check which method was used
        return render_template("form.html")
    else:
        name = request.form["name"] # request.form is similar to a dictionary
        age = request.form["age"]
        interest = request.form.getlist("interest") #used for checkbox (ie multi-select) 

        #the second and third arguments are based on variables needed in the HTML using Jinja
        return render_template("results.html",name=name,age=age,interest=interest)

# Arguments from URL --> does not use form submission
@app.route("/videos/<int:userid>/<filename>",methods=["GET"])
def videos(userid,filename):
    return render_template("videos.html",userid=userid,filename=filename)

# Redirecting to another page
@app.route("/media/<type>/<int:userid>/<filename>",methods=["GET"])
def media(type,userid,filename):
    if type == "videos":
        return redirect(url_for("videos",userid=userid,filename=filename))
    else:
        return render_template("error.html")

# Uploading & Downloading of Files (unlikely to come out but just in case)
@app.route("/photos/<filename>")
def get_files(filename):
    return send_from_directory(folder,filename)

@app.route("/photos",methods=["GET","POST"])
def photos():
    files = []
    for key in request.files:
        file = request.files[key] #requests.files is a dictionary with the value being the file object
        filename = secure_filename(file.filename) # prevents malicious filenames
        file.save(path)
        files.append(filename)

app.run()
```

## HTML - Basic

All HTML files are to be in a templates folder (templates folder to be in same folder as python file)

If there are CSS files, all css files and other files (e.g. images) are to be placed in static folder in same folder as python file


```
<! DOCTYPE html>
<html>
    <head>
        <title>My Website</title>
        <style>
            body {
                font-family: sans-serif;
                font-size: 10px;
                background-color:green;
            }

            table,th,td,tr {
                border-collapse: collapse;
                border: 1px solid black;
                margin:auto;
                width: 50%;
            }
            th {
                font-weight: bold;
                text-decoration: underline;
                font-style: italic;
                color: blue;
            }
            th,td {
                text-align:center;
            }
        </style>

        <!-- Alternatively: <link href="index.css" rel="stylesheet"> with css stylesheet in static folder-->
    </head>
    <body>
        <h1>My Website</h1>
        <br>
        <table>
            <tr>
                <th>Header 1</th>
                <th>Header 2</th>
                <th>Header 3</th>
            </tr>
            {% for row in data %}
                <tr>
                    <td>{{row["Head1"]}}</td>
                    <td>{{row["Head2"]}}</td>
                    <td>{{row["Head3"]}}</td>
                </tr>
            <!-- Possible to use {% break %} to break out of loop -->
            {% endfor %}

        </table>

        {% if userid == 1 %}
            <h2>Hi User 1</h2>
        {% endif %}

        <!-- to show images from static folder-->
        <img src={{url_for("static",filename=filename)}} alt="image description" width=50%>

        <!-- to show images from another folder (may be a working folder aka changes and not static) -->
        <img src={{url_for("get_file",filename=filename)}} alt="image description" width=50%>

    </body>
</html>
```

## HTML Form
```
<! DOCTYPE html>
<html>
    <head>
        <title>My Website</title>
        <style>
            body {
                font-family: sans-serif;
            }
        </style>
    </head>
    <body>
        <h1>My Website</h1>
        <form action={{url_for("form")}} method="post" enctype="multipart/form-data">
            <label for="name">Name: </label>
            <input type="text" id="name" name="name" value="hello">

            <br>
            <br>

            <!-- label follows id of input, one label per input, name of all inputs of same group is same-->
            <!-- value is what is returned when called for in program-->

            <input type="radio" name="gender" id="female" value="Female">
            <label for="female">Female </label><br>
            <input type="radio" name="gender" id="male" value="Male">
            <label for="male">Male </label><br>
            <input type="radio" name="gender" id="other" value="Others">
            <label for="other">Others </label><br>

            <!-- Similar style for checkbox (name: which question | value is placeholder/actual value | id is for CSS)-->

            <input type="file" id="myfile" name="myfile">
            <input type="file" id="myfile1" name="myfile1">

            <input type="submit" value="Submit!">
        </form>
    </body>
</html>
```

