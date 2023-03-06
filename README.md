# Starve-Free-Readers-Writers-Problem

The readers-writers problem is a famous synchronisation problem in the field of operating systems.
 The problem has objects of two kinds readers and writers working on the same shared data.
 The initial observations on analysing the problem statement are as follows :
 1. reading and writing cannot occur at the same time.
 2. Multiple writers cannot write at the same time.
 3. Multiple readers can read at the same time.
```
      The solutions presented in class :
      1 . prioritize readers
      2 . prioritize writers
```

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

```
    Queue
    We declare a new data type Node in the form of a struct
    for pid(porcess id) : a datatype provided by C++ (uint32_t -> unsigned integere of 32 bits) is used.
    std_err_value is an integer used to denote that the operation used on the queue was invalid and can be defined
    by the user as follows.

    #define std_err_value  999999999
    
    Struct Node
    { 
      unit32_t pid         
      Node *next         //pointer pointing to next node in queue or NULL in the case of END
      Node(uint32_t x)         //parameterized constructor 
      { 
         pid = x
         next = NULL
      }
}
      from here on is standard implementation of queue 

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
        { 
            uint32_t val = head->data
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
     { 
        sem++;
        if(sem < 1)
        {   unint32_t picked_up = Q.pop()
            // waking up a blocked process
            wakeup(picked_up)
        }
    }
    void wait(uint32_t pid)
    {   
        sem--
        if(sem < 0)
        {   Q.push(pid)
            // system-call that can block a process through its pid 
            block(pid)
        }
    }
}

```

 
 The solution pseudocode in three parts :
 

 
 Part 1: These are global variables
```
    Semaphore* in, out, write 
    in->sem = 1
    out->sem = 1 
    write->sem = 0 

    int in_readers = 0 // readers that have started reading
    int out_readers = 0 // #readers that have completed reading 
    boolean wait_write = 0 // true if a writer is waiting 

    semaphores in , out , write :
    in: manages the queue of all incoming processes, helps to use in_readers.
    out: helps using out_readers
    write: contains at most one process at any time, a writer which is waiting for readers to finish .
         
    in_readers and out_readers variables are maintained to not allow writers to access data unless all readers
    have finished their reading.
    The wait_write is maintained to not allow more readers to start reading while a writer is waiting which came
    before them.

 ```

Part 2 : Reader process
```

       // it is assumed  that this process will be written in the form of a method to which the pid will be a param
        in->wait(pid) // Call wait 
        in_readers++ // increment in_readers as the process starts reading
        in->signal()         
        // Critical section : perform the read operation
        out->wait(pid) // wait on the out semaphore 
        out_readers++             // increment out_readers after reading is complete
        if(in_readers == out_readers && wait_write) {   // if writer is waiting and the last reader has finished
            write->signal()                   
            }                           // then signal write semaphore
        out->signal() 

         Here all variables reffered with "" re semapohores 
        1. A Reader process on arrival waits on "in", then once woken up it changes in_readers, releases 
        "in" and enters the critical section, i.e. starts reading.
        2. On completion of its job, it waits on the "out" and when woken up it updates out_readers. 
        3. At this point, it checks if in_readers and out_readers are equal && wait_write is 1. If both the 
        conditions are satisfied, it means entire set of readers have finished their job and now the waiting writer 
        may enter,henec it does "write"->signal().
        After which "out" is released.

```
Part 3 : Writer Process
```

       // it is assumed  that this process will be written in the form of a method to which the pid will be a param
        in->wait(pid) //waiting on in
        out->wait(pid) //waiting on out
        if(in_readers == out_readers) {   
        // if no processes are currently reading enter the critical section
            out->signal()  //signalling the out semaphore
        } 
        else {                  // wait for readers to finish
            wait_write = 1       // writer has started waiting for the readers inside the critical section to finish
            out->signal()         //out semaphore released
            writer->wait()        // wait on writer semaphore
            wait_write = 0         // when no more writers waiting
            }
        //Perform the Wrting operation , Critical section
        in->signal()
        
        1. Writer process waits on "in" and then "out" semaphores.
        2. Then it checks if any reader is still reading if there are none then it releases "out" and starts writing.
        3. If readers are still reading the writer releases "out" but starts waiting on "writer",also the waiting
           to 1. Once write->signal() is triggered, waiting is set to 0 and goes on to execute its critical section.
```

**References**
1. https://arxiv.org/abs/1309.4507
2. https://www.geeksforgeeks.org/queue-linked-list-implementation/amp/
