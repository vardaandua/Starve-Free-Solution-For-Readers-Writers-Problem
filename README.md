# Starve-Free-Readers-Writers-Problem

The readers-writers problem is a famous synchronisation problem in the field of operating systems.
 The problem has objects of two kinds readers and writers working on the same shared data.
 The initial observations on analysing the problem statement are as follows :
 1. reading and writing cannot occur at the same time.
 2. Multiple writers cannot write at the same time.
 3. Multiple readers can read at the same time.

The solutions presented in class :
1 . prioritize readers
2 . prioritize writers

But these solutions are not starve free.
Hence a proposed solution for the above problem which is also starve free is described as follows :
1. Use semaphores that work like a queue data structure (FIFO : first in first out)
2. Hence if a process gets first in queue it will be the first one to get out , that is get executed or will get 
to enter the critical section by releasing the semaphore.
3. In this solution it is ensured that if a writer is waiting no newly entered reader gets to read hence no 
starvation for writers.
4. The Queueing nature guarantees no starvation for readers.

Hence the above presented idea is a starve free solution to the Readers Writer Problem.

From here on I present a pseudocode for the queue and semaphore required above :

 Queue
 We declare a new data type Node in the form of a struct
 for pid(porcess id) : a datatype provided by C++ (uint32_t -> unsigned integere of 32 bits) is used.
 std_err_value is an integer used to denote that the operation used on the queue was invalid and can be defined
 by the user as follows.
 
 '''
#define std_err_value  999999999
Struct Node
{   unit32_t pid         
    Node *next         //pointer pointing to next node in queue or NULL in the case of END
    Node(uint32_t x)         //parameterized constructor 
    {   pid = x
        next = NULL
    }
}
    // from here on is standard implementation of queue 
struct Queue
{
    Node *head = NULL , *tail = NULL      
    // head is the pointer of next executing process while tail is the point of entry
    void push(uint32_t pid)
    {
        Node *last = new Node(pid)
        if(head == NULL)                    //if current process is the first one 
        {
            head = NULL
            tail = NULL
        }    
        else                                // if the process is kth one and k>1
        {
            tail->next = last
            tail = last
        }
    }
    uint32_t pop()
    {
        if(head euqals NULL)              // if the size of queue = 0
            return std_err_value                // returns an error value 
        else
        {  uint32_t val = head->data
            head = head->next
            if(head euqals NULL)            //if current node was the last
            tail = NULL
            return val
        }
    }
}
struct Semaphore
{
    int sem = 1  , Queue Q
    void signal()
    {   sem++;
        if(sem < 1)
        {   unint32_t picked_up = Q.pop()
            // waking up a blocked process
            wakeup(picked_up)
        }
    }
    void wait(uint32_t pid)
    {   sem--
        if(sem < 0)
        {   Q.push(pid)
            // system-call that can block a process through its pid 
            block(pid)
        }
    }
}
'''
