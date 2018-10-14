---
num: "Lecture 3"
desc: "Dynamic Memory Management"
ready: true
date: 2018-10-11 11:00:00.00-7:00
---

# Dynamic Memory Management

* So far, we have been working exclusively with static allocation and stack memory.
* ... but there are situations where knowing the exact size BEFORE runtime is limiting.
* It’s not feasible to store dynamically growing / shrinking objects on the stack!
* For dynamically allocating memory during the execution of a program, there’s the free store, or more commonly known as the heap.

# Heap
* The heap allows users to manually decide when, how much, and what will be stored in it.
* On the flip side, you must also decide when memory will be deallocated (deleted) in order to reuse it.
* <b>Q:</b> What happens if you forget to delete stuff from the heap?
	* Memory leaks! Memory space is finite and eventually your program will crash if unused memory is not being deleted.
* <b>Q:</b> But we can store strings of different sizes on the stack... what’s up with that?
	* On the stack, strings are a fixed size containing fields such as the length of the string, and a memory address to the sequence of characters on the heap.
	* Note that a memory address and integer size ARE of fixed length (as well as other information the string class may have).
	* Behind the scenes, as strings may grow and shrink on the heap, the address that points to the sequence of chars on the heap can change.
* The programmer manages heap memory space manually.

## "new" operator

* The primary use for pointers are to access objects that have been dynamically allocated on the heap.
* The result of "new" is a pointer to the object that was allocated on the heap.

## Example
```
int* p = new int(10);	// allocate an int on the heap and point p to it
*p = 2;			// sets the int value to 2 (without address change)
int& q = *p;		// q refers to the value that p points to
q = 4;			// change int value to 4

cout << *p << endl;	// prints 4

int* r = &q;		// make r point to the address that q refers to
*r = 6;			// change int value that r points to to 6

cout << *p << endl;	// prints 6
cout << q << endl;	// prints 6
cout << *r << endl;	// prints 6
```

![new example](newExample.png)

## "delete" operator

* Anytime you use the “new” operator, consider if you need to eventually delete the memory it consumes.
	* Otherwise, your program will have memory leaks and eventually crash!

## Example of memory leaks

```
void g(int x) {
	int* p = new int(x);	// memory leak!
	delete p;		// remove this and see application size grow!
	return;
}

int main() {
for (int i = 0; i < 100000000; i++)
		g(i);
}
```

* since int* p in g() is a local variable, this gets removed from the stack, BUT THE VALUE ON THE HEAP REMAINS.
	* Once the variable is removed when the function exits, there is no way to remove this from the heap.
		* This is a memory leak!
* "delete" is the opposite of new
	* "new" creates an object on the heap and returns a pointer to it.
	* "delete" takes a pointer and removes the data from the heap.
		* "delete p;" doesn’t delete the variable p, it deletes what p points to on the heap!
		* "delete" only works for values that exist on the heap.

# Dangling Pointers
* After a pointer’s memory is deleted, the pointer itself is still available to use.
* BUT, you should never use a dangling pointer (the behavior of it is undefined).
	* And why would you want to use a pointer to something that shouldn’t be there?

## Dangling Pointers Example

```
int g(int x) {
	double* p = new double(x);
	delete p;	// delete value from the heap
	return *p; 	// p is a dangling pointer since heap contents were removed!
			// Results in undefined behavior. Shouldn’t do this!
}
```

# Dynamically allocating arrays
* We have been allocating arrays on the stack, but we can also do this on the heap!
	* Actually, we SHOULD do this on the heap if our array size is not known during compile time.
	* Example of declaring and deleting an array on the heap. 

```
int* arr = new int[10];	// create an array of 10 integers on the heap
delete [] arr;		// removes the array from the heap. Note the [] syntax for deleting arrays from the heap.
```

* Some compilers may have extended functionality to support dynamically allocating arrays using the syntax for statically allocating arrays.
	* For example, g++/clang++ allows this, but VisualStudio currently does not:

```
int x;
cout << "Enter a positive number: ";
cin >> x;
// int intArray[x]; // ERROR IN VisualStudio, NOT g++ / clang++
int* intArray = new int[x]; // LEGAL IN BOTH
for (int i = 0; i < x; i++) {
	intArray[i] = i;
	cout << intArray[i] << endl;
}

// Note: to delete an array on the heap, use delete [] intArray;
delete [] intArray;
```

# Linked Lists

* There are many ways to organize data (data structures)
* We've seen examples of stacks and arrays
	* But there are others like trees, tries, hash tables, …
* And there are many ways to IMPLEMENT these data structures
	* Some may use arrays (like arrayLists or vectors)
	* Some may use object pointers to link structures together.
		* Linked lists form a collection similar (but also very different) than an array.

* We can represent any type of data using structs.
* For example, an ITEM (Node) in a linked list can be a struct that contains some data (like a string, int, double, char, or another struct).
* The NEXT element in a linked list is a pointer to a linked list node representing the next item in the collection.
* Since we don't know how many elements a linkedList may contain, managing this structure on the heap is essential!

## Linked List Structs

* There are many ways we can implement a linked list, but we will focus on an singly-linked list implementation with a head and tail reference.
* A LinkedList struct will contain two pointers to linked list nodes:
	* one points to the first element of the list (head)
	* the other points to the last element of the list (tail).
* A node in a linked list will contain some data, and a pointer to the next Node in the linked list collection.

```
struct Node {
	int data;
	Node* next;
};

struct LinkedList {
  Node* head;
  Node* tail;
};
```

![general list](generalList.png)

## Example of "walking down" a linked list

* Assume we have a linked list with the following nodes:

```
[10]->[20]->[30]->null
```

* We can walk down the collection and visit every node by using each Node's next pointer as long as the next pointer != NULL.

```
cout << list->head->data << endl; // 10
cout << list->head->next->data << endl; // 20
cout << list->head->next->next->data << endl; // 30
cout << list->head->next->next->next->data << endl; // seg fault!
```

* The performance of certain functionality with data structures largely depends on how the data is organized under-the-hood.
* On a high-level, arrays and linked lists are conceptually the same.
	* However, depending on what kind of operations performed on the data structure, performance may greatly differ. For example,
		* inserting to the front of a singly-linked linked list is much faster than inserting to the front of an array.
		* directly accessing an element in an array is much faster than directly accessing an element in a linked list.

## insertToFront() Linked List

* Order of pointer reassignment matters since we always want to make sure we don't "lose" the address of something we need on the heap.

1. Create new node to insert
2. Assign new node's next pointer to the current list's head.
3. Reassign the list's head to the new node.

![insertToFront](insertToFront.png)

## deleteIndex(int index)

Case 1: Remove first element

1. Reassign list head to list head's next pointer.
2. delete first element.

Case 2: Remove middle element

1. Reassign previous node's next pointer to current node's next pointer.
2. delete current node.

Case 3: Remove last element

1. Set previous node's next pointer to NULL.
2. Reassign list's tail pointer to previous node.
3. delete current node.

![delete index](deleteIndex.png)

# Linked List Implementation

```
// LinkedList.h
#ifndef LINKEDLIST_H
#define LINKEDLIST_H

struct Node {
	int data;
	Node* next;
};

struct LinkedList {
	Node* head;
	Node* tail;
};

void printList(LinkedList* list);
void insertToFront(LinkedList* list, int value);
bool exists(LinkedList* list, int value);
int length(LinkedList* list);
void deleteIndex(LinkedList* list, int index);

#endif
--------------
// LinkedList.cpp
#include <iostream>
#include "linkedList.h"
using namespace std;

void printList(LinkedList* list) {
	for (Node* i = list->head; i != NULL; i = i->next) {
		cout << "[" << i->data << "]->";
	}
	cout << "null" << endl;
}

void insertToFront(LinkedList* list, int value) {
	if (list->head == 0) {		// empty list
		Node* item = new Node;
		item->data = value;
		item->next = NULL;
		list->head = item;
		list->tail = item;
	} else { 			// not empty
		Node* item = new Node;
		item->data = value;
		item->next = list->head;
		list->head = item;
	}
}

bool exists(LinkedList* list, int value) {
	for (Node* i = list->head; i != NULL; i = i->next) {
		if (i->data == value)
			return true;
	}
	return false;
}

int length(LinkedList* list) {
	int counter = 0;
	// let's use a while loop instead of a for...
	Node* temp = list->head;
	while (temp != 0) {
		counter++;
		temp = temp->next;
	}
	return counter;
}

void deleteIndex(LinkedList* list, int index) {
	if (index >= length(list) || index < 0) {
		cerr << "Invalid index: " << index << endl;
		return;
	}

	// check if first item
	if (index == 0 && list->head != 0) {
		Node* temp = list->head;
		list->head = list->head->next;
		delete temp;
		return;
	}

	Node* prev = list->head;
	Node* curr = list->head->next;
	for (int i = 1; i < index; i++) {
		prev = prev->next;
		curr = curr->next;
	}

	// remove the current node
	prev->next = curr->next;
	delete curr;

	// re-assign list->tail if curr was the last node.
	if (index == length(list) - 1)
		list->tail = prev;

	return;
}
----------
// main.cpp
#include <iostream>
#include <string>
#include <fstream>
#include "linkedList.h"

using namespace std;

int main() {
	LinkedList* list = new LinkedList;
	list->head = 0;
	list->tail = 0;

	cout << length(list) << endl;

	insertToFront(list, 10);
	cout << length(list) << endl;
	insertToFront(list, 20);
	insertToFront(list, 30);
	printList(list);			// 30->20->10

	cout << exists(list, 15) << endl; 	// 0
	cout << exists(list, 30) << endl; 	// 1
	cout << exists(list, -1) << endl; 	// 0

	cout << length(list) << endl;		// 3

	deleteIndex(list, -1); 			// err
	deleteIndex(list, 5);			// err
	deleteIndex(list, 3);			// err
	deleteIndex(list, 2);			// OK!
	printList(list);			// 30->20->null

	return 0;
}
----------
$ g++ -o main main.cpp LinkedList.cpp
```
