Download Link: https://assignmentchef.com/product/solved-operating-systems-laboratory-cs39002-assignment-5
<br>
<strong>Operating Systems Laboratory (CS39002)</strong>

<strong>Assignment 5:  </strong>Simulation of Virtual Memory using Demand Paging

<strong><u>Problem Overview:</u></strong>

It is required to simulate a demand-paged virtual memory management system. In a typical operating system, processes arrive in the system and they are scheduled for execution by the scheduler. Once a process is allocated CPU, it starts execution whereby the CPU generates a sequence of virtual addresses. The corresponding sequence of virtual page numbers is referred to as the <strong><em>page reference string</em></strong>. Once a virtual address (page number) is generated, it is given to the Memory Management Unit (MMU) to determine the corresponding physical address (frame number) in memory. In order to determine the frame number, it consults the <strong><em>translation lookaside buffer (TLB),</em></strong><em> in the event of a miss, it consults the <strong>page table</strong></em>. If the page is present in the main memory the current process can continue its execution; otherwise, a <strong><em>page fault</em></strong> occurs and the requested page must be brought from disk to main memory (i.e. an I/O operation). During this time, the scheduler takes the CPU away from current process, puts it to waiting state, and schedules another process. Meanwhile the page-fault handling interrupt service routine <strong>(ISR)</strong> associated with the MMU performs the following activities to bring the requested page into main memory. First, it checks if there is any free frame in main memory. If so, it loads the page into this frame. Otherwise it identifies a <strong><em>victim page</em></strong> using Least Recently Used (LRU) page-replacement algorithm, removes the victim page from memory, and brings in the new page. Once the page-fault is handled, the process comes out of the waiting state and gets added to the ready queue so that the scheduler can select it again.

The whole program has to be structured into the following four modules:

<ol>

 <li><strong>Master</strong> is responsible for creating other modules as well initialization of the data structures.</li>

 <li><strong>Scheduler</strong> is responsible for scheduling various processes using FCFS algorithm.</li>

 <li><strong>MMU</strong> translates the page number to frame number and handles page-fault, and also manages the page table.</li>

 <li><strong>Process</strong> generates the page numbers from page reference string.</li>

</ol>

After all the <strong>Process</strong>es have finished execution, the <strong>Scheduler</strong> will notify <strong>Master</strong>. Master will terminate Scheduler, MMU and then will terminate itself.

The inputs to the program will be the following:

<ol>

 <li>i) Total number of processes (k) ii) Virtual address space – maximum number of pages required per process (m) iii) Physical address space – total number of frames in maim memory (f) iv) Size of the TLB (s)</li>

 <li><strong>The Master module:</strong></li>

</ol>

The Master module performs the following tasks.

<ol>

 <li>Creates and initializes different data structures as mentioned below:

  <ol>

   <li><strong>Page Table</strong> – There will be one page table for each process, where the size of each page table is same as the virtual address space. This is to be implemented using shared memory (SM1).</li>

   <li><strong>Free Frame List</strong> – This will contain a list of all free frames in main memory, and will be used by MMU. This isto be implemented using shared memory (SM2).</li>

  </ol></li>

</ol>

<ul>

 <li><strong>Ready Queue</strong> – This queue is used by the scheduler, and is to beimplemented using message queue (MQ1).</li>

</ul>

<ol>

 <li>In order to communicate between different modules it creates two more message queues MQ2 (for communication between Scheduler and MMU), and MQ3 (for communication between Process-es and MMU).</li>

</ol>

<ol>

 <li>Creates the <strong>Scheduler</strong> module as a separate Linux process for scheduling the Process-es from ready queue (MQ1). It passes the parameters (MQ1, MQ2) via command-line arguments during process creation.</li>

 <li>Creates <strong>MMU</strong> module as a separate Linux process for translating page numbers to frame numbers and to handle page faults. It passes the parameters (MQ2, MQ3, SM1, SM2) via command line arguments during process creation.</li>

 <li>Creates k number of <strong>Process</strong>-es as separate Linux processes at fixed interval of time (250 ms). <strong>Master</strong> generates the page reference string (R<sub>i</sub>) for every process (P<sub>i</sub>) and passes the same via command line argument during process creation. It also passes the parameters (MQ1, MQ3) to every process.</li>

</ol>

The <strong>Master</strong> module is to be implemented in Master.c or Master.cpp file. It will read 3 inputs (k, m, f) as mentioned above. For every process, it selects a random number between (1,m) and assigns it as the required number of pages of that process and allocates frames proportionately.

The page reference string will be generated as follows:

<ul>

 <li>For a process P<sub>i</sub>, if total number of pages required is m<sub>i</sub> as randomly selected by <strong>Master</strong>, then

  <ul>

   <li>Length of reference string for P<sub>i</sub> is &gt;=2* m<sub>i</sub> and &lt;=10*m<sub>i</sub></li>

   <li>Select a random number (<em>x</em>) from this interval and that is the length of the reference string</li>

  </ul></li>

 <li><strong>Master</strong> first creates and initializes all data structures, then creates <strong>Scheduler</strong> and next creates <strong>MMU</strong>. Then it creates all the ‘k’ <strong>Process</strong>-es.</li>

 <li>After performing all process creation tasks, <strong>Master</strong> will wait() till scheduler notifies completion of all the process execution. <strong>Master</strong> will terminate <strong>Scheduler</strong> and <strong>MMU</strong> first, and then terminates itself.</li>

</ul>

<ol start="2">

 <li><strong>The Scheduler module</strong></li>

</ol>

Scheduler is responsible for scheduling all the ‘k’ processes. It continuously scans the ready queue and selects processes in FCFS order for scheduling. Initially when the ready queue is empty, it waits on the ready queue (MQ1). Once a process gets added, it starts scheduling.

Scheduler selects the first process from ready queue, removes it from the queue and sends it a signal for starting execution. Then the scheduler blocks itself till gets notification message from MMU. It can receive two types of message from MMU:

<ul>

 <li>Type I message “<strong><em>PAGE FAULT HANDLED</em></strong>” – After successful page-fault handling</li>

</ul>

After getting this signal, it enqueues the current process and schedules the first process from Ready queue.

<ul>

 <li>Type II message “<strong><em>TERMINATED</em></strong>” – After successful termination of process After getting this signal, it schedules the first process from Ready queue.</li>

</ul>

The <strong>Scheduler</strong> module is to be implemented in sched.c or sched.cpp file. <strong>Master</strong> executes the program via exec() with the proper arguments as explained in the implementation of <strong>Master</strong>. Once all the <strong>Process</strong>-es are executed, <strong>Scheduler</strong> informs the <strong>Master</strong> to terminate all the modules.

<ol start="3">

 <li><strong>The Process modules:</strong></li>

</ol>

Execution of process means generation of page numbers from reference string. Process sends the page number to MMU using message queue (MQ3) and receives the frame number from MMU.

<ul>

 <li>If MMU sends a valid frame number o It parses the next page number in the reference string and goes in loop</li>

 <li>Else, in case of page fault o It gets -1 as frame number from MMU and saves the current element of the reference string for continuing its execution when it is scheduled next and goes into wait(). MMU invokes page fault handling routine to handle the page fault. It may be noted that the current process is out of Ready queue and Scheduler enqueues it to the Ready queue once page fault is resolved.</li>

 <li>Else, in case of invalid page reference o It gets -2 as frame number from MMU and terminates itself. The MMU informs the scheduler to schedule the next process.</li>

</ul>

When the Process completes the scanning of the reference string, it sends -9 (marker to denote end of page reference string) to the MMU and MMU will notify the Scheduler (see MMU).

The <strong>Process</strong> module is to be implemented in process.c or process.cpp. Processes are created by <strong>Master </strong>with proper argument (as mentioned in Master section). The processes will put them in the Ready queue and pause themselves. Whenever the Scheduler schedules a process, only then it will come out of this pause state and will start execution. The process reads the reference string one by one and sends them to the MMU and receives the corresponding frame number (when available), -1 (when page faults occurs). When the process completes the scanning of the reference string, it sends -9 (to denote end of page reference string) to the MMU, and MMU will notify the Scheduler. Scheduler will terminate the process and removes from Ready queue.

<ol start="4">

 <li><strong>The Memory Management Unit (MMU) Process:</strong></li>

</ol>

<strong>Master</strong> creates <strong>MMU</strong> and then it pauses. <strong>MMU</strong> wakes up after receiving the page number via message queue (MQ3) from <strong>Process</strong>. It receives the page number from the process and checks the <strong>TLB</strong>, failing which, it checks the page table for the corresponding process. There can be following two cases:

<ol>

 <li>If the page is already in page table, <strong>MMU</strong> sends back the corresponding frame number to the process.</li>

 <li>Else in case of page fault

  <ul>

   <li>Sends -1 to the process as frame number</li>

   <li>Invokes the page fault handler to handle the page fault

    <ul>

     <li>If free frame available – update the page table and also updates the corresponding free-frame list.</li>

     <li>If no free frame available – do local page replacement. Select victim page using LRU, replace it and brings in a new frame and update the page table.</li>

    </ul></li>

  </ul></li>

</ol>

 Intimate the Scheduler by sending Type I message to enqueue the current process and schedule the next process.

The data transfer of the page contents between the main memory and the disk may be assumed to happen in the background via DMA, without blocking the execution of the running process. However, the execution of the ISR will occupy the CPU, for a short time, which may be ignored for simplicity.

If MMU receives the page number -9 via message queue then it infers that the process has completed its execution and it updates the free-frame list and releases all allocated frames. After this, MMU sends Type II message to the Scheduler for scheduling the next process.

The MMU maintains a global timestamp (count), which is incremented by 1 after every page reference.

<strong>MMU</strong> is implemented in MMU.c or MMU.cpp. MMU will be executed via the <strong>Master</strong> process with four command line arguments: page table (SM1), free frame list (SM2), MQ2 and MQ3 as mentioned in <strong>Master</strong> module. It will implement page fault handling as a function inside it and call it whenever a page fault occurs.

<strong>Data Structures Required:</strong>

<ul>

 <li>Page Table</li>

</ul>

Implemented as shared memory

For each process there is a page table

Size of each page table is same as the size of virtual space

Each entry in page table contains &lt; frame_number, valid/invalid bit &gt;

Initially, all frame numbers are equal to -1

<ul>

 <li>Free Frame List</li>

</ul>

Implemented as shared memory

Created by Master and maintained by MMU

<ul>

 <li>Process to Page Number Mapping</li>

</ul>

Can be implemented using shared memory

It contains the number of pages required by every process

<strong>Required Outputs:</strong>

The following outputs are to be generated.

<ol>

 <li>c runs in xterm and produce the page fault information:

  <ol>

   <li>Page fault sequence (pi,xi) – where pi indicates the process number and xi indicate the page number for which a page fault is generated. Also prints total number of pagefault for every process.</li>

   <li>Global ordering (ti,pi,xi) – where ti indicates the global timestamp (which is incremented by 1 after every page reference) maintained by MMU, pi is the process number and xi is the page number.</li>

  </ol></li>

 <li>All these outputs are written into an output file “result.txt”.</li>

</ol>

<strong>Evaluation Guidelines: </strong>

While entering marks, the partwise break up should also be entered according to the marking guidelines given below. There is a separate component for individual assessment, based on how the student answers questions.

<table width="448">

 <tbody>

  <tr>

   <td colspan="2" width="39"><strong>Sl</strong></td>

   <td width="363"><strong>Items for the Master</strong></td>

   <td width="46"><strong>Marks</strong></td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1a)</strong></td>

   <td width="363">Spawning the processes of the other modules</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1b)</strong></td>

   <td width="363">Creating the page table</td>

   <td width="46">10</td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1c)</strong></td>

   <td width="363">Creating the free frame list</td>

   <td width="46">10</td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1d)</strong></td>

   <td width="363">Creating the ready queue</td>

   <td width="46">10</td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1e)</strong></td>

   <td width="363">Creating the message queues</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td colspan="2" width="39"><strong>(1f)</strong></td>

   <td width="363">Other operational responsibilities of the Master</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td colspan="2" width="39"> </td>

   <td width="363"><strong>Total</strong></td>

   <td width="46">45</td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"><strong>Sl</strong></td>

   <td width="363"><strong>Items for the Scheduler</strong></td>

   <td width="46"><strong>Marks</strong></td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"><strong>(2a)</strong></td>

   <td width="363">Monitoring and maintaining the ready queue</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"><strong>(2b)</strong></td>

   <td width="363">Selecting processes for scheduling</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"><strong>(2c)</strong></td>

   <td width="363">Scheduling processes on receiving the Type I message</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"><strong>(2d)</strong></td>

   <td width="363">Scheduling processes on receiving the Type II message</td>

   <td width="46">5</td>

  </tr>

  <tr>

   <td width="3"> </td>

   <td width="37"> </td>

   <td width="363"><strong>Total</strong></td>

   <td width="46">20</td>

  </tr>

  <tr>

   <td width="3"></td>

   <td width="37"></td>

   <td width="363"></td>

   <td width="46"></td>

  </tr>

 </tbody>

</table>




<table width="442">

 <tbody>

  <tr>

   <td width="1"> </td>

   <td colspan="2" width="34"><strong>Sl</strong></td>

   <td colspan="2" width="363"><strong>Items for the Process</strong></td>

   <td colspan="2" width="46"><strong>Marks</strong></td>

  </tr>

  <tr>

   <td width="1"> </td>

   <td colspan="2" width="34"><strong>(3a)</strong></td>

   <td colspan="2" width="363">Handling valid page numbers from the MMU</td>

   <td colspan="2" width="46">5</td>

  </tr>

  <tr>

   <td width="1"> </td>

   <td colspan="2" width="34"><strong>(3b)</strong></td>

   <td colspan="2" width="363">Handling page fault from the MMU</td>

   <td colspan="2" width="46">5</td>

  </tr>

  <tr>

   <td width="1"> </td>

   <td colspan="2" width="34"><strong>(3c)</strong></td>

   <td colspan="2" width="363">Handling invalid page reference from the MMU</td>

   <td colspan="2" width="46">5</td>

  </tr>

  <tr>

   <td width="1"> </td>

   <td colspan="2" width="34"> </td>

   <td colspan="2" width="363"><strong>Total</strong></td>

   <td colspan="2" width="46">15</td>

  </tr>

  <tr>

   <td colspan="2" width="34"><strong>Sl</strong></td>

   <td colspan="2" width="363"><strong>Items for the MMU</strong></td>

   <td colspan="2" width="46"><strong>Marks</strong></td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td colspan="2" width="34"><strong>(4a)</strong></td>

   <td colspan="2" width="363">Maintaining and simulating the TLB</td>

   <td colspan="2" width="46">5</td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td colspan="2" width="34"><strong>(4b)</strong></td>

   <td colspan="2" width="363">Maintaining and working on the page table</td>

   <td colspan="2" width="46">5</td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td colspan="2" width="34"><strong>(4c)</strong></td>

   <td colspan="2" width="363">Handling page faults</td>

   <td colspan="2" width="46">10</td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td colspan="2" width="34"><strong>(4d)</strong></td>

   <td colspan="2" width="363">Maintaining the global timestamp</td>

   <td colspan="2" width="46">5</td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td colspan="2" width="34"> </td>

   <td colspan="2" width="363"><strong>Total</strong></td>

   <td colspan="2" width="46">25</td>

   <td width="1"> </td>

  </tr>

  <tr>

   <td width="1"></td>

   <td width="33"></td>

   <td width="1"></td>

   <td width="361"></td>

   <td width="1"></td>

   <td width="45"></td>

   <td width="1"></td>

  </tr>

 </tbody>

</table>


