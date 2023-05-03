Download Link: https://assignmentchef.com/product/solved-cs333-programming-project-4-kernel-resource-management
<br>
In this project, you will implement three monitors that will be used in our Kernel.  These are the ThreadManager, the ProcessManager, and the FrameManager.  The code you write will be similar to other code from the previous projects in that these three monitors will orchestrate the allocation and freeing of resources.

There is also an additional task—re-implement the Condition and Mutex classes to provide Hoare Semantics—but that code will not be used in the Kernel.

Download New Files

Start by creating a new directory for this project and then download all the files from:

<strong>http://www.cs.pdx.edu/~harry/Blitz/OSProject/p4/</strong>

Even though some of the files have the same names, be sure to get new copies for each project.  In general some files may be modified.

Please keep your old files from previous projects separate and don’t modify them once you submit them.  This is a good rule for all programming projects in all classes.  If there is ever any question about whether code was completed on time, we can always go back and look at the Unix “file modification date” information.

For this project, you should get the following files:

<strong>     makefile </strong>

<strong>     DISK </strong>

<strong>     Runtime.s </strong>

<strong>     Switch.s </strong>

<strong>     System.h </strong>

<strong>     System.c </strong>

<strong>     List.h </strong>

<strong>     List.c </strong>

<strong>     BitMap.h </strong>

<strong>     BitMap.c </strong>

<strong>     Main.h </strong>

<strong>     Main.c </strong>

<strong>     Kernel.h </strong>

<strong>     Kernel.c </strong>

The packages called <strong>Thread</strong> and <strong>Synch</strong> have been merged together into one package, now called

<strong>Kernel</strong>.  This package contains quite a bit of other material as well, which will be used for later projects.  In this and the remaining projects, you will be modifying the <strong>Kernel.c</strong> and <strong>Kernel.h</strong> files.  Don’t modify the code that is not used in this project; just leave it in the package.

The <strong>Kernel.c</strong> file contains the following stuff, in this order:

<strong>Thread scheduler functions </strong>

<strong>Semaphore class </strong>

<strong>Mutex class </strong>

<strong>Condition class </strong>

<strong>Thread class </strong>

<strong>ThreadManager class </strong>

<strong>ProcessControlBlock class </strong>

<strong>ProcessManager class </strong>

<strong>FrameManager class </strong>

<strong>AddrSpace class TimerInterruptHandler other interrupt handlers </strong>

<strong>SyscallTrapHandler </strong>

<strong>Handle functions </strong>




In this project, you can ignore everything after <strong>TimerInterruptHandler</strong>.  The classes called

<strong>ThreadManager</strong>, <strong>ProcessManager</strong>, and <strong>FrameManager</strong> are provided in outline, but the bodies of the methods are unimplemented.  You will add implementations.  Some other methods are marked “unimplemented;” those will be implemented in later projects.

The <strong>BitMap</strong> package contains code you will use; read over it but do not modify it.

The <strong>makefile</strong> has been modified to compile the new code.  As before, it produces an executable called <strong>os</strong>.

You may modify the file <strong>Main.c</strong> while testing, but when you do your final run, please use the <strong>Main.c </strong>file as it was distributed.  In the final version of our kernel, the <strong>Main</strong> package will perform all initialization and will create the first thread.  The current version performs initialization and then calls some testing functions.

Task 1:  Threads and the ThreadManager

In this task, you will modify the <strong>ThreadManager</strong> class and provide implementations for the following methods:

<strong>            Init </strong>

<strong>            GetANewThread </strong>

<strong>            FreeThread </strong>




In our kernel, we will avoid allocating dynamic memory.  In other words, we will not use the heap.  All important resources will be created at startup time and then we will carefully monitor their allocation and deallocation.




An example of an important resource is <strong>Thread</strong> objects.  Since we will not be able to allocate new objects on the heap while the kernel is running, all the <strong>Thread</strong> objects must be created ahead of time.  Obviously, we can’t predict how many threads we will need, so we will allocate a fixed number of <strong>Thread</strong> objects (e.g., 10) and re-use them.




When a user process starts up, the kernel will need to obtain a new <strong>Thread</strong> object for it.  When a process dies, the <strong>Thread</strong> object must be returned to a pool of free <strong>Thread</strong> objects, so it can be recycled for another process.




<strong>Kernel.h</strong> contains the line:




<u>const</u> MAX_NUMBER_OF_PROCESSES = 10




Since each process in our OS will have at most one thread, we will also use this number to determine how many <strong>Thread</strong> objects to place into the free pool initially.




To manage the <strong>Thread</strong> objects, we will use the <strong>ThreadManager</strong> class.  There will be only one instance of this class, called <strong>threadManager</strong>, and it is created and initialized at startup time in <strong>Main.c</strong>.




Whenever you need a new <strong>Thread</strong> object, you can invoke <strong>threadManger.GetANewThread</strong>.  This method should suspend and wait if there are currently none available.  Whenever a thread terminates, the scheduler will invoke <strong>FreeThread</strong>.  In fact, the <strong>Run</strong> function has been modified in this project to invoke <strong>FreeThread</strong> when a thread terminates—thereby adding it to the free list—instead of setting its <strong>status</strong> to UNUSED.




Here is the definition of <strong>ThreadManager</strong> as initially distributed:




<u>class</u> ThreadManager     <u>superclass</u> Object     <u>fields</u>

threadTable: <u>array</u> [MAX_NUMBER_OF_PROCESSES] <u>of</u> Thread       freeList: List [Thread]     <u>methods</u>       Init ()

Print ()

GetANewThread () <u>returns</u> <u>ptr</u> <u>to</u> Thread       FreeThread (th: <u>ptr</u> <u>to</u> Thread)   <u>endClass</u>




When you write the <strong>Init</strong> method, you’ll need to initialize the array of <strong>Thread</strong>s and you’ll need to initialize each <strong>Thread</strong> in the array and set its status to <strong>UNUSED</strong>.  (Each Thread will have one of the following as its status: <strong>READY</strong>, <strong>RUNNING</strong>, <strong>BLOCKED</strong>, <strong>JUST_CREATED</strong>, and <strong>UNUSED</strong>.)

Threads should have the status <strong>UNUSED</strong> iff they are on the <strong>freeList</strong>.  You’ll also need to initialize the <strong>freeList</strong> and place all <strong>Thread</strong>s in the <strong>threadTable</strong> array on the <strong>freeList</strong> during initialization.




You will need to turn the <strong>ThreadManager</strong> into a “monitor.”  To do this, you might consider adding a <strong>Mutex</strong> lock (perhaps called <strong>threadManagerLock</strong>) and a condition variable (perhaps called <strong>aThreadBecameFree</strong>) to the <strong>ThreadManager</strong> class.  The <strong>Init</strong> method will also need to initialize <strong>threadManagerLock </strong>and <strong>aThreadBecameFree</strong>.




The <strong>GetANewThread</strong> and <strong>FreeThread</strong> methods are both “entry methods,” so they must obtain the monitor lock in the first statement and release it in the last statement.




<strong>GetANewThread</strong> will remove and return a <strong>Thread</strong> from the <strong>freeList</strong>.  If the <strong>freeList</strong> is empty, this method will need to wait on the condition of a thread becoming available.  The <strong>FreeThread</strong> method will add a <strong>Thread</strong> back to the <strong>freeList</strong> and signal anyone waiting on the condition.




The <strong>GetANewThread</strong> method should also change the <strong>Thread</strong> status to <strong>JUST_CREATED</strong> and the <strong>FreeThread</strong> method should change it back to <strong>UNUSED</strong>.




We have provided code for the <strong>Print</strong> method to print out the entire table of <strong>Thread</strong>s.




Note that the <strong>Print</strong> method disables interrupts.  The <strong>Print</strong> method is used only while debugging and will not be called in a running OS so this is okay.  Within the <strong>Print</strong> method, we want to get a clean picture of the system state—a “snapshot”—(without worrying about what other threads may be doing) so disabling interrupts seems acceptable.  However, the other methods—<strong>Init</strong>, <strong>GetAThread</strong> and <strong>FreeThread</strong>—must NOT disable interrupts, beyond what is done within the implementations of <strong>Mutex</strong>, <strong>Condition</strong>, etc.




In <strong>Main.c</strong> we have provided a test routine called <strong>RunThreadManagerTests</strong>, which creates 20 threads to simultaneously invoke <strong>GetAThread</strong> and <strong>FreeThread</strong>.  Let’s call these the “testing threads” as opposed to the “resource threads,” which are the objects that the <strong>ThreadManager</strong> will allocate and monitor.  There are 20 testing threads and only 10 resource thread objects.




Every thread that terminates will be added back to the <strong>freeList</strong> (by <strong>Run</strong>, which calls <strong>FreeThread</strong>).

Since the testing threads were never obtained by a call to <strong>GetANewThread</strong>, it would be wrong to add them back to the <strong>freeList</strong>.  Therefore, each testing thread does not actually terminate.  Instead it freezes up by waiting on a semaphore that is never signaled.  By the way, the testing threads are allocated on the heap, in violation of the principle that the kernel must never allocate anything on the heap, but this is okay, since this is only debugging code, which will not become a part of the kernel.




In the kernel, we may have threads that are not part of the <strong>threadTable</strong> pool (such as the <strong>IdleThread</strong>), but these threads must never terminate, so there is no possibility that they will be put onto the <strong>freeList</strong>.  Thus, the only things on the <strong>freeList</strong> should be <strong>Threads</strong> from <strong>threadTable</strong>.




You will also notice that the <strong>Thread</strong> class has been changed slightly to add the following fields:




<u>class</u> Thread

…     <u>fields</u>

…

isUserThread: <u>bool</u>

userRegs: <u>array</u> [15] <u>of</u> <u>int</u>    — Space for r1..r15       myProcess: <u>ptr</u> <u>to</u> ProcessControlBlock     <u>methods</u>

…

<u>endClass</u>




These fields will be used in a later project.  The <strong>Thread</strong> methods are unchanged.










<h1>Task 2:  Processes and the ProcessManager</h1>




In our kernel, each user-level process will contain only one thread.  For each process, there will be a single <strong>ProcessContolBlock</strong> object containing the per-process information, such as information about open files and the process’s address space.  Each <strong>ProcessControlBlock</strong> object will point to a <strong>Thread</strong> object and each <strong>Thread</strong> object will point back to the <strong>ProcessControlBlock</strong>.




There may be other threads, called “kernel threads,” which are not associated with any user-level process.  There will only be a small, fixed number of kernel threads and these will be created at kernel start-up time.




For now, we will only have a modest number of <strong>ProcessControlBlocks</strong>, which will make our testing job a little easier, but in a real OS this constant would be larger.




<u>const</u> MAX_NUMBER_OF_PROCESSES = 10




All processes will be preallocated in an array called <strong>processTable</strong>, which will be managed by the <strong>ProcessManager</strong> object, much like the <strong>Thread</strong> objects are managed by the <strong>ThreadManager</strong> object.




Each process will be represented with an object of this class:




<u>class</u> ProcessControlBlock     <u>superclass</u> Listable     <u>fields</u>       pid: <u>int</u>       parentsPid: <u>int</u>

status: <u>int</u>                   — ACTIVE, ZOMBIE, or FREE       myThread: <u>ptr</u> <u>to</u> Thread                    exitStatus: <u>int</u>       addrSpace: AddrSpace

fileDescriptor: <u>array</u> [MAX_FILES_PER_PROCESS] <u>of</u> <u>ptr</u> <u>to</u> OpenFile     <u>methods</u>       Init ()

Print ()       PrintShort ()   <u>endClass</u>




Each process will have a process ID (the field named <strong>pid</strong>).  Each process ID will be a unique number, from 1 on up.




Processes will be related to other processes in a hierarchical parent-child tree.  Each process will know who its parent process is.  The field called <strong>parentsPid</strong> is a integer identifying the parent.  One parent may have zero, one, or many child processes.  To find the children of process X, we will have to search all processes for processes whose <strong>parentsPid</strong> matches X’s <strong>pid</strong>.




The <strong>ProcessControlBlock</strong> objects will be more like C structs than full-blown C++/Java objects: the fields will be accessed from outside the class but the class will not contain many methods of its own.  Other than initializing the object and a couple of print methods, there will be no other methods for <strong>ProcessControlBlock</strong>.  We are providing the implementations for the <strong>Init</strong>, <strong>Print</strong> and <strong>PrintShort</strong> methods.




Since we will have only a fixed, small number of <strong>ProcesControlBlocks</strong>, these are resources which must be allocated.  This is the purpose of the monitor class called <strong>ProcessManager</strong>.




At start-up time, all <strong>ProcessControlBlocks</strong> are initially FREE.  As user-level processes are created, these objects will be allocated and when the user-level process dies, the corresponding <strong>ProcessControlBlock</strong> will become FREE once again.




In Unix and in our kernel, death is a two stage process.  First, an ACTIVE process will execute some system call (e.g., <strong>Exit()</strong>) when it wants to terminate.  Although the thread will be terminated, the <strong>ProcessControlBlock</strong> cannot be immediately freed, so the process will then become a ZOMBIE.  At some later time, when we are done with the <strong>ProcessControlBlock</strong> it can be FREEd.  Once it is FREE, it is added to the <strong>freeList</strong> and can be reused when a new process is begun.




The <strong>exitStatus</strong> is only valid after a process has terminated (e.g., a call to <strong>Exit()</strong>).  So a ZOMBIE process has a terminated thread and a valid <strong>exitStatus</strong>.  The ZOMBIE state is necessary just to keep the exit status around.  The reason we cannot free the <strong>ProcessControlBlock</strong> is because we need somewhere to store this integer.




For this project, we will ignore the <strong>exitStatus</strong>.  It need not be initialized, since the default initialization

(to zero) is fine.  Also, we will ignore the ZOMBIE state.  Every process will be either ACTIVE or FREE.




Each user-level process will have a virtual address space and this is described by the field <strong>addrSpace</strong>.  The code we have supplied for <strong>ProcessControlBlock.Init</strong> will initialize the <strong>addrSpace</strong>.  Although the <strong>addrSpace</strong> will not be used in this project, it will be discussed later in this document.




The <strong>myThread</strong> field will point to the process’s <strong>Thread</strong>, but we will not set it in this project.




The <strong>fileDescriptors</strong> field describes the files that this process has open.  It will not be used in this project.




Here is the definition of the <strong>ProcessManager</strong> object.




<u>class</u> ProcessManager     <u>superclass</u> Object     <u>fields</u>

processTable: <u>array</u> [MAX_NUM_OF_PROCESSES] <u>of</u> ProcessControlBlock       processManagerLock: Mutex       aProcessBecameFree: Condition       freeList: List [ProcessControlBlock]       aProcessDied: Condition

<u>methods</u>       Init ()

Print ()

PrintShort ()

GetANewProcess () <u>returns</u> <u>ptr</u> <u>to</u> ProcessControlBlock

FreeProcess (p: <u>ptr</u> <u>to</u> ProcessControlBlock)

TurnIntoZombie (p: <u>ptr</u> <u>to</u> ProcessControlBlock)

WaitForZombie (proc: <u>ptr</u> <u>to</u> ProcessControlBlock) <u>returns</u> <u>int</u>   <u>endClass</u>




There will be only one <strong>ProcessManager</strong> and this instance (initialized at start-up time) will be called <strong>processManager</strong>.




processManager = <u>new</u> ProcessManager       processManager.Init ()




The <strong>Print()</strong> and <strong>PrintShort()</strong> methods for <strong>ProcessControlBlocks</strong> are provided for you.  You are to implement the methods <strong>Init</strong>, <strong>GetANewProcess</strong>, and <strong>FreeProcess</strong>.  The methods <strong>TurnIntoZombie</strong> and <strong>WaitForZombie</strong> will be implemented in a later project and can be ignored for now.




The <strong>freeList</strong> is a list of all <strong>ProcessControlBlocks</strong> that are FREE.  The status of a <strong>ProcessControlBlock</strong> should be FREE if and only if it is on the <strong>freeList</strong>.




We assume that several threads may more-or-less simultaneously request a new <strong>ProcessControlBlock</strong> by calling <strong>GetANewProcess</strong>.  The <strong>ProcessManager</strong> should be a “monitor,” in order to protect the <strong>freeList</strong> from concurrent access.  The Mutex called <strong>processManagerLock</strong> is for that purpose.  When a <strong>ProcessControlBlock</strong> is added to the <strong>freeList</strong>, the condition <strong>aProcessBecameFree</strong> can be <strong>Signal</strong>ed to wake up any thread waiting for a <strong>ProcessControlBlock</strong>.




Initializing the <strong>ProcessControlManager</strong> should initialize




the <strong>processTable</strong> array

all the <strong>ProcessControlBlocks</strong> in that array

the <strong>processManagerLock</strong>

the <strong>aProcessBecameFree</strong> and the <strong>aProcessDied</strong> condition variables              the <strong>freeList</strong>




All <strong>ProcessControlBlocks</strong> should be initialized and placed on the <strong>freeList</strong>.




The condition called <strong>aProcessDied</strong> is signaled when a process goes from ACTIVE to ZOMBIE.  It will not be used in this project, but should be initialized nonetheless.




The <strong>GetANewProcess</strong> method is similar to the <strong>GetANewThread</strong> method, except that it must also assign a process ID.  In other words, it must set the <strong>pid</strong>.  The <strong>ProcessManager</strong> will need to manage a single integer for this purpose.  (Perhaps you might call it <strong>nextPid</strong>).  Every time a <strong>ProcessControlBlock</strong> is allocated (i.e., everytime <strong>GetANewProcess</strong> is called), this integer must be incremented and used to set the process’s <strong>pid</strong>.  <strong>GetANewProcess</strong> should also set the process’s status to ACTIVE.




The <strong>FreeProcess</strong> method must change the process’s status to FREE and add it to the free list.




Both <strong>GetANewProcess</strong> and <strong>FreeProcess</strong> are monitor entry methods.










<h1>Task 3: The Frame Manager</h1>




The lower portion of the physical memory of the BLITZ computer, starting at location zero, will contain the kernel code.  It is not clear exactly how big this will be, but we will allocate 1 MByte for the kernel code.  After that will come a portion of memory (called the “frame region”) which will be allocated for various purposes.  For example, the disk controller may need a little memory for buffers and each of the user-level processes will need memory for “virtual pages.”




The area of memory called the frame region will be viewed as a sequence of “frames”.  Each frame will be the same size and we will have a fixed number of frames.  For concreteness, here are some constants from <strong>Kernel.h</strong>.




PAGE_SIZE = 8192                               — in hex: 0x00002000

PHYSICAL_ADDRESS_OF_FIRST_PAGE_FRAME = 1048576 — in hex: 0x00100000   NUMBER_OF_PHYSICAL_PAGE_FRAMES = 512           — in hex: 0x00000200




This results in a frame region of 4 MB, so our kernel would fit into a 5 MByte memory.




The frame size and the page size are the same, namely 8K.  In later projects, each frame will hold a page of memory.  For now, we can think of each frame as a resource that must be managed.  We will not really do anything with the frames.  This is similar to the dice in the gaming parlor and the forks for the philosophers… we were concerned with allocating them to threads, but didn’t really use them in any way.




Each frame is a resource, like the dice of the game parlor, or the philosophers’ forks.  From time to time, a thread will request some frames; the <strong>frameManager</strong> will either be able to satisfy the request, or the requesting thread will have to wait until the request can be satisfied.




For the purposes of testing our code, we will work with a smaller frame region of only a few frames.  This will cause more contention for resources and stress our concurrency control a little more.  (For later projects, we can restore this constant to the larger value.)




NUMBER_OF_PHYSICAL_PAGE_FRAMES = 27           — For testing only




Here is the definition of the <strong>FrameManager</strong> class:




<u>class</u> FrameManager     <u>superclass</u> Object     <u>fields</u>

framesInUse: BitMap       numberFreeFrames: <u>int</u>       frameManagerLock: Mutex       newFramesAvailable: Condition

<u>methods</u>       Init ()

Print ()

GetAFrame () <u>returns</u> <u>int</u>          — returns addr of frame

GetNewFrames (aPageTable: <u>ptr</u> <u>to</u> AddrSpace, numFramesNeeded: <u>int</u>)

ReturnAllFrames (aPageTable: <u>ptr</u> <u>to</u> AddrSpace)   <u>endClass</u>




There will be exactly one <strong>frameManager</strong> object, created at kernel start-up time.




frameManager = <u>new</u> FrameManager       frameManager.Init ()




With frames (unlike the <strong>ProcessControlBlocks</strong>) there is no object to represent each resource.  So to keep track of which frames are free, we will use the <strong>BitMap</strong> package.  Take a look at it.  Basically, the <strong>BitMap</strong> class gives us a way to deal with long strings of bits.  We can do things like (1) set a bit, (2) clear a bit, and (3) test a bit.  We will use a long bit string to tell which frames are in use and which are free; this is the <strong>framesInUse</strong> field.  For each frame, there is a bit.  If the bit is 1 (i.e., is “set”) then the frame is in use; if the bit is 0 (i.e., is “clear”) then the frame is free.




The <strong>frameManager</strong> should be organized as a “Monitor class.”  The <strong>frameManagerLock</strong> is used to make sure only one method at a time is executing in the <strong>FrameManager</strong> code.




We have provided the code for the <strong>Init</strong>, <strong>Print</strong>, and <strong>GetAFrame</strong> methods; you’ll need to implement <strong>GetNewFrames</strong>, , and <strong>ReturnAllFrames</strong>.




The method <strong>GetANewFrame</strong> allocates one frame (waiting until at least one is available) and returns the address of the frame.  (Since there is never a need to return frames one at a time, there is no “ReturnOneFrame” method.)




When the frames are gotten, the <strong>GetNewFrames</strong> method needs to make a note of which frames have been allocated.  It does this by storing the address of each frame it allocates (the address of the first byte in each frame) into an <strong>AddrSpace </strong>object.




An <strong>AddrSpace</strong> object is used to represent a virtual address space and to tell where in physical memory the virtual pages are actually located.  For example, for a virtual address space with 10 pages, the <strong>AddrSpace</strong> object will contain an ordered list of 10 physical memory addresses.  These are the addresses of the 10 “frames” holding the 10 pages in the virtual address space.  However, the <strong>AddrSpace</strong> object contains more information.  For each page, it also contains information about whether the page has been modified, whether the page is read-only or writable, etc.  The information in an <strong>AddrSpace</strong> object is stored in exactly the format required by the CPU’s memory management hardware.  In later projects, this will allow us to use the <strong>AddrSpace</strong> object as the current page table for a running user-level process.  At that time, when we switch to a user-level process, we’ll have to tell the CPU which <strong>AddrSpace</strong> object to use for its page table.  In addition to looking over the code in <strong>AddrSpace</strong>, you may want to review the BLITZ architecture manual’s discussion of page tables.




The code in method




GetNewFrames (aPageTable: <u>ptr</u> <u>to</u> AddrSpace, numFramesNeeded: <u>int</u>)

<strong> </strong>

needs to do the following:




<ul>

 <li>Acquire the frame manager lock.</li>

 <li>Wait on <strong>newFramesAvailable</strong> until there are enough free frames to satisfy the request. (3) Do a loop for each of the frames</li>

</ul>

for i = 0 to numFramesNeeded-1

<ul>

 <li>determine which frame is free (find and set a bit in the <strong>framesInUse</strong> BitMap)</li>

 <li>figure out the address of the free frame</li>

 <li>execute the following</li>

</ul>

aPageTable.SetFrameAddr (i, frameAddr)

to store the address of the frame which has been allocated

<ul>

 <li>Adjust the number of free frames</li>

 <li>Set <strong>numberOfPages</strong> to the number of frames allocated.</li>

 <li>Unlock the frame manager</li>

</ul>




The code in method




ReturnAllFrames (aPageTable: ptr to AddrSpace)




needs to do more or less the opposite.  It can look at <strong>aPageTable.numberOfPages</strong> to see how many are being returned.  It can then go through the page table and see which frames it possessed.  For each, it can clear the bit.




for i = 0 to numFramesReturned-1      frameAddr = aPageTable.ExtractFrameAddr (i)       bitNumber = …frameAddr…

framesInUse.ClearBit(bitNumber )        endFor




It will also need to adjust the number of free frames and “notify” any waiting threads that more frames have become available.




You’ll need to do a <strong>Broadcast</strong>, because a <strong>Signal</strong> will only wake up one thread.  The thread that gets awakened may not have enough free frames to complete, but other waiting threads may be able to proceed.  A broadcast should be adequate, but perhaps after carefully studying the Game Parlor problem, you will find a more elegant approach which wakes up only a single thread.




Also note that there is a possibility of starvation here.  It is possible that one large process will be waiting for a lot of frames (e.g., 100 frames).  Perhaps there are many small processes which free a few frames here and there, but there are always other small processes that grab those frames.  Since there are never more than a few free frames at a time, the big process will get starved.




This particular scenario for starvation, where processes are competing for frames) is a very real danger in an OS and a “real” OS would need to ensure that starvation could not happen.  However, in our situation, it is acceptable to provide a solution that risks starvation.




Do not modify the code for the <strong>AddrSpace</strong> class.




<strong> </strong>

<strong> </strong>

<h1>Task 4:  Change Condition Variables to Hoare Semantics</h1>




The code we have given you for the <strong>Signal</strong> method in the <strong>Condition</strong> class uses MESA semantics.  Change the implementation so that it uses Hoare semantics.




With MESA semantics, you tend to see code like this in monitors:




NewResourcesHaveBecomeAvail: Condition




<em><u>In method A:</u> </em>

…

numberAvail = numberAvail + 1              NewResourcesHaveBecomeAvail.Signal()

…

<em><u>In method B:</u> </em>

…

while (numberAvail == 0)

NewResourcesHaveBecomeAvail.Wait()

endWhile

…




The code in method B contains a <strong>while</strong> loop because there is a possibility that some other thread has snuck in between the <strong>Signal</strong> and the <strong>Wait</strong>.  When method A increments <strong>numberAvail</strong>, the condition (“resources are now available for some other thread to use”) has been made true.  Method A invokes <strong>Signal</strong> to wake up some thread that is waiting for resources.  The problem is that some other thread may have run between the <strong>Signal</strong> and the reawakening of the <strong>Wait</strong> in method B.  Other threads may have come in and grabbed all the resources.  So when the waiting thread is reawakened, it must always recheck for resources (or more generally, it must check to ensure the condition is still true.)

Unfortunately, the condition may have changed back to false (i.e., no resources are available), so the thread will have to wait some more.

And with MESA semantics, starvation becomes a bigger problem.  What if the thread waits, wakes up, and then goes back to sleep, over and over.  Each time a <strong>Signal</strong> occurs, some other thread just happens to get in first and steal the resource.  Then the unlucky thread keeps waking up, testing, and going back into a waiting sleep.  This is a very real possibility if you have three threads and they are scheduled in round-robin fashion.  Thread A runs and signals thread C.  Thread B runs and takes the resource.  Thread C runs, finds the condition is false and goes back to sleep.  Then thread A runs again, and so on.

With Hoare semantics, the thread needing the resources can use code like this:




…

if (numberAvail == 0)

NewResourcesHaveBecomeAvail.Wait()

endIf        …

Once the thread is reawakened, it can be sure that nothing has happened between the <strong>Signal</strong> and it; it can be sure that no other thread has gotten into the monitor.




Starvation is easier to ensure against, too.  If a thread needs a resource, it waits.  We assume the waiting queue is FIFO, so that the waiting thread will eventually be awakened.  And when it wakes, it will proceed forward, without looping.  Therefore, each <strong>Signal</strong> wakes one thread which proceeds and, given enough <strong>Signals</strong> to awaken any thread who went to sleep first, the thread in question will eventually get awakened.

(Of course starvation-related bugs are still possible!  For example, it might be that the code fails to <strong>Signal</strong> the condition enough times, leaving some thread waiting forever.  The contribution of the monitor concept is to make it easier to write bug-free code, not to make it impossible to create bugs!)

All further design choices are up to you.  We are not providing any testing code at all; you’ll have to figure out how to test your code.

If you have time, you may go back to the previous tasks to incorporate your Hoare Semantics code in their solutions, but remember to complete the above tasks <u>before</u> starting on the Hoare Semantics task.

<strong><u>Do not modify…</u></strong>

Do not modify any files except:

<strong>     Kernel.h </strong>

<strong>     Kernel.c </strong>

Do not create global variables (except for testing purposes).  Do not modify the methods we have provided.

Policy Concerning Project Deadlines




The previous projects were designed to get you up to speed and to let you know how much time the programming portion of this class will take.  Some of the tasks were intentionally quite difficult, in order to challenge the ambitious students and to motivate all students to pay close attention to the subtleties of concurrency problems and their solutions.  The previous projects were independent, in the sense your solution code did not need to be 100% correct in order to continue with the BLITZ kernel project.




Things change with this project.  Buggy code in this project is not acceptable and will make it impossible to complete the future projects.




In particular, you must complete tasks 1, 2 and 3 in this project before you can begin the next project.  You must get your code working and pass the test programs before going on, unlike the previous projects in which successful completion was not a barrier to moving on.  However, task 4 (the Hoare Semantics task) is somewhat optional, in the sense that this code will not be needed in future projects.




If you are able to complete tasks 1, 2 and 3 on time, then hand in whatever you were able to achieve on task 4 and move on to the next project.  The Hoare Semantics task is provided as an extra challenge; if you don’t have time for it, then do not allow yourself to fall behind on the next project.




If you are unable to complete tasks 1, 2 and 3, keep working on them until your code works 100% correctly.




Students who have difficulty completing the projects on time will start to fall behind.  Perhaps such a student can worker a little harder to catch up on the next project and can still get the entire kernel finished on time.  Or perhaps the student will remain behind schedule and will be unable to complete one or more of the later projects.  Such a student will not be able to get the completed kernel working by the end of the class.




We recommend that your instructor base grades, from this project onward, on program “style” only.  We recommend they simply not accept any programs that fail to work correctly, as defined by our test suite.  If there are problems with the output, the student must finish / fix the code and re-submit it when it is working correctly, regardless of how long it takes.  The final course grade will then be determined in part by how many of the projects the student was able to complete before the end of the course.




Your instructor may alter this policy.  Of course, your instructor’s policy takes precedence over what is written here.





