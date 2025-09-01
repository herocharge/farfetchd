---
title: "Pointer Wars: Linked list"
date: 2025-09-01
draft: false
ShowToc: true
author: Herocharge
---

#### This is a writeup for pointerwars 2025 level 3
Thanks to Chris Feilbach for organizing the event [view contest here](https://chrisfeilbach.com/2025/06/21/sign-up-for-another-round-of-pointer-wars-linked-list-edition/).

Pwars Github: https://github.com/pointerwars2025/

My Code: https://github.com/herocharge/pointerwars-2025/

The goal is to improve a BFS implemented using queues which in turn were implemented using linked lists.

We start with a basic implementation of linked_list

```c
struct linked_list {
    struct node * head;
    size_t size;
};

struct node {
    struct node * next;
    unsigned int data;
};

```

A lot of operations such as `insert` and `remove` can be called with improper arguments and storing the `size` makes validating the index faster.

We implement a queue on top of this linked_list

```c
struct queue {
    struct* linked_list ll;
};

```

A simple wrapper works well for now.

#### Optimization 1: Adding the tail pointer
We know that queues are FIFO i.e., new nodes are added at the end and popped from the front. Our linked list implementation so far forces us to iterate through the entire linked list to get to the end. So every time a node is inserted into the queue, we iterate throught the whole thing. 

To avoid this we can store an additional pointer in the linked_list struct to keep track of the tail node and when needed can be accessed in constant time.

```c
struct linked_list {
    struct node * head;
    struct node * tail;
    size_t size;
};
```

#### Optimization 2: Getting rid of double dereference
Since the queue is wrapper around a linked list pointer, we end up dereferencing two pointers everytime we need to access something from the linked list.

For example:
```c
queue->ll->size
```

since both `queue` and `queue->ll` could be any pointer, there will be two `load` operations happening and typically those ops are costlier than arithmetic ops.
To avoid this, instead of having a pointer to a `linked_list` in the `queue` struct, we directly embed it.

```c
struct queue {
    struct linked_list ll;
};
```
This allows us to get values in the linked list with a single dereference 

Example:
```c
(&(queue->ll)).size
```


#### Optimization 3: Free list
In the BFS, nodes are constantly pushed and popped, malloc'ing and freeing these nodes everytime push and pop happens can be inefficient. Instead, we can keep aside a node which is meant to be freed and reuse it when a new node needs to be malloc'ed.
We essentially create a smaller linked list inside the linked list to keep track of nodes which are supposed to be freed but are saved for reuse.

We will add nodes only to the head and remove only from the head of this `free list` so it's more akin to a stack.

```c
struct linked_list {
    struct node * head;
    struct node * tail;
    struct node * free_stack;
    size_t size;
};
```

If we need a new node, we first check if the free_stack has a node that we can use. If not, we malloc a new node.

This way, the total number of mallocs will be limited to the maximum width of a BFS search which will be less than the number of nodes visited in the BFS.

#### Optimization 4: Custom malloc and free
Every malloc call has some overhead. Malloc'ing a single block of million bytes is faster than making a million malloc calls for 1 byte each.

A simple benchmark on my PC has the following results:
```
Single malloc time: 0.000387 seconds
Multiple mallocs time: 0.003097 seconds // Almost (8x slower, number doesn't mean much)
```

Since we know we might need a lot of nodes for our BFS, allocating a lot of them at once should be faster them allocating each node individually. 

There were 2 approaches that I could've taken:
1. A separate malloc and free wrapper.
    In this approach, we have a global linked list which keeps track of allocated blocks of nodes which are block of nodes which are free'd only if all the nodes in a block are to be deleted.

    However, I don't want to use a global variable, so I used the next approach (which is probably not the best).

2. Bake the block allocation approach into the create_linked_list function
    To achieve this, we need to have an extra bit with each node indicating if it's the head of a block or not. 

## Current Linked List Implementation

```c
struct linked_list {
    struct node * head;
    struct node * tail;
    struct node * free_stack;
    size_t size;
};

struct node {
    struct node * next;
    unsigned int data;
    bool is_block_head;
};
```

Say this is the current state of the linkedlist 

```
head->  +-----+------+
        | 10  |  o-->| 
        +-----+------+
            +-----+------+
            | 20  |  o-->| 
            +-----+------+
                    +-----+------+
                    | 30  |  o-->| 
                    +-----+------+ <- tail
                            NULL

free_list-> NULL

```


If we remove the second element, it becomes

```
head->  +-----+------+
        | 10  |  o-->| 
        +-----+------+
                    +-----+------+
                    | 30  |  o-->| 
                    +-----+------+ <- tail
                            NULL

free_list-> +-----+------+
            | 20  |  o-->| 
            +-----+------+
                    NULL

```

If we remove second element again, it becomes,
```
head->  +-----+------+
        | 10  |  o-->| 
        +-----+------+ <- tail
                            NULL

free_list-> +-----+------+
            | 30  |  o-->| 
            +-----+------+
                    +-----+------+
                    | 20  |  o-->| 
                    +-----+------+
                            NULL

```

If we want to insert say 40 at the end, we take the nodes from the free list and use it.

```
head->  +-----+------+
        | 10  |  o-->| 
        +-----+------+
                +-----+------+
                | 40  |  o-->| 
                +-----+------+ <- tail
                        NULL

free_list-> +-----+------+
            | 20  |  o-->| 
            +-----+------+
                    NULL

```


Now let's take a look at the BFS code (an abridged version) to understand why we need Optimization 4.
```c
while(!found_path) {
    // Push data onto the queue.
    struct row * row = rows[next_node];
    bool not_done = queue_pop(queue, &next_node);
    // ignoring some visited array logic

    unsigned int* adjacent_nodes = row->adjacent_nodes;
    for(size_t node = 0; node < row->size; node++) {
        unsigned int data = adjacent_nodes[node];
        //....
        if (j == data) {
                found_path = true;
        }
        queue_push(queue, adjacent_nodes[node]);

    }

	// Pop the next row off the queue.
	//
	bool full = queue_pop(queue, &next_node);
}
```

Here we see that for every node we push a lot of nodes. By nature of the BFS, the number of pushes will always be more than number of pops. 
I tried plotting the number of pushes vs pops for test case 1 and the following is what I got.

![Push Plot graph](../../images/push_pop.png)

We see that the number of pushes are significantly more than the number of pops. (Ignore the labels in the graph, im too tired to change them)

This means that the free list is often empty when trying to allocate a new node. 

Idea! If we are ever in a position with an empty free list and need to insert a new node, we preallocate a lot of nodes into the free list which can be used by later pushes.

What's the most efficient way of inserting a lot of nodes at once into the free_list??

This is where Optimization 4 comes in. We allocate a large array of nodes with a single malloc instead of malloc'ing individual nodes. Once the block is allocated we go through and link all the nodes that are next to each other.

```c

struct node * __linked_list_allocate_block(struct linked_list* ll, size_t size){
    assert(size >= 1);
    struct node* head = malloc_fptr(sizeof(struct node) * size);
    if(head == NULL)
        return NULL;
    
    head->is_block_head = true;

    struct node* curr = head;
    for(size_t i = 0; i < size - 1; i++){
        curr->next = curr + 1;
        curr = curr->next;
        curr->is_block_head = false;
    }
    curr->next = NULL;
    ll->free_stack = head->next;
    return head;
}

struct node* __linked_list_get_new_node(struct linked_list* ll){
    if(ll->free_stack != NULL){
        struct node* tmp = ll->free_stack;
        ll->free_stack = ll->free_stack->next;
        return tmp;
    }
    else {
        return __linked_list_allocate_block(ll, extra_size);
    }
}
```


However this causes a problem when deleting the linked list.
Right now, when deleting, I just loop through the free_list and call free on each node. But it wouldn't work anymore. This is because we can only call free on pointers returned by malloc.

Say I allocated a block of size 10 nodes. There are two kinds of nodes in this block. The first node, which has the same pointer as the pointer of the block returned by malloc and the remaining nodes. 
```
+-----+-----+-----+-----+-----+
|  1  |  2  |  3  |  4  |  5  | ......
+-----+-----+-----+-----+-----+
```
I can call `free` only on the first node and when the first node is free'd then the rest of the nodes in this block are free'd as well.

If I continue deleting the linked list as usual, I can either run into a double free on the nodes or a free on an invaid address.

To avoid this, I added an extra field in the node structure

```
struct node {
    struct node * next;
    unsigned int data;
    bool is_block_head;
};
```

`is_block_head` will indicate whether the node is to be deleted or not. 

I delete the linked list in two passes, one where I collect all the block heads and the other where I actually delete them. If I try to delete in a single pass, I might free nodes which I might need to walk the linked list later.

There is one last thing to discuss in Optimization 4. How many nodes should one preallocate?
I played around with a bunch of number and the following is the result.
![Performance plots](../../images/plot.png)

The numbers indicate the different preallocation block sizes.
The plot also indicates some other minor optimizations such as loop unrolling and removing NULL checks. The optimal number depends on the machine you are running this on. 

