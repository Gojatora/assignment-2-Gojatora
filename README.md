**Reflection:**

This is the final code that was written from the provided chunk of code:

![image alt]([image_url](https://github.com/Gojatora/assignment-2-Gojatora/blob/eb9aea529431827f2912bfb41c2521ccdaad8cda/Code%20Screenshot.png))

Output:

![image alt](https://github.com/Gojatora/assignment-2-Gojatora/blob/eb9aea529431827f2912bfb41c2521ccdaad8cda/Output%20Screenshot.png)

Implementing this distributed program provided a significant learning curve regarding how parallel software actually executes. Initially, I was confused when the program produced no meaningful output after simply running the code cell. I realized that the standard Jupyter "Run" command only executes a single process (Rank 0), leaving the Master with zero workers to communicate with.

The breakthrough came when I learned that I could access the system terminal directly within the notebook by using the exclamation mark (!). By running !mpiexec -n 11 python mpi_demo.py, I was able to trigger the MPI orchestrator to spawn the necessary 11 instances of the script. This experience highlighted the fundamental difference between sequential and distributed computing: parallel programs require a specific environment to manage multiple processes. Once properly launched, the power of the Master-Worker model became clear: ten workers performed unique calculations simultaneously, showing how distributed systems can scale horizontally to handle massive workloads.


**Guide Question**

**1. Why is message passing required in distributed systems?**
In a distributed system, processes run on independent computers (nodes) that do not share the same physical memory (RAM). Because Process A cannot directly reach into the memory of Process B on another machine, they must use Message Passing to exchange information.

Communication across Networks: It allows processes to coordinate over a network (like the Internet or a local cluster).
Synchronization: It ensures that one process doesn't move forward until it has received necessary data from another (using "blocking" calls like recv).
Scalability: You can add 10, 100, or 1,000 more workers to a system simply by connecting them to the message-passing network.

**2. What happens if one process fails?**

In a basic MPI program like the one we wrote, the system is highly sensitive to failure.
When a process fails in a message-passing environment, the entire system often faces immediate disruption because standard MPI setups are highly sensitive to errors. A common consequence is deadlock, or "hanging," which occurs if a worker process crashes before it can send its expected data; because the Master process is programmed to wait at a receiving command until the message arrives, it will stall indefinitely waiting for communication that can never happen. 

Furthermore, many MPI runtimes handle unexpected exits through system termination, where the manager automatically aborts the entire job across all remaining processes to prevent the spread of corrupted data or the waste of computational resources. To mitigate these risks, professional distributed systems incorporate fault tolerance strategies, such as "check-pointing"—the practice of periodically saving the state of a computation so it can be resumed—or utilizing specialized libraries designed to detect process failure and automatically restart the failed components.

**3. How does this model differ from shared-memory programming?**

The core difference between these two models lies in how processes access and exchange data across the system. In shared-memory programming, multiple threads or processes coexist on a single machine and have access to the same global memory space, much like several people writing on a single shared whiteboard. Because they all "see" the same variables, communication is implicit; if one thread updates a piece of data, others see that change immediately.

In contrast, the message-passing model used in distributed systems operates under the "shared-nothing" principle, where each process lives in its own isolated "silo" of memory. Since Process A cannot directly read the memory of Process B (often because they are on physically different computers), they must explicitly package data into a message and "mail" it across a network. While shared-memory is generally faster and easier to program for a single multi-core computer, message passing is the only way to scale applications across thousands of separate servers in a cloud or supercomputing cluster.

