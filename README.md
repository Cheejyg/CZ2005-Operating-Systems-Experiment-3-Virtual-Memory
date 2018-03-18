# Experiment 3

## Virtual Memory

### 1. OBJECTIVES

After completing this lab, you will be able to: 

* Understand why TLB (Translation Look-aside Buffer) can provide fast address translation.
* Know how address translation is done by using IPT (Inverted Page Table).
* Understand how page replacement is essential to virtual memory and how to implement a least recently used replacement algorithm.

### 2. LABORATORY

**Software Lab 1B** (N4-01a-02)

### 3. EQUIPMENT

Pentium IV PC with Nachos 3.4.

### 4. MODE OF WORKING

You should be working alone. No group effort.

### 5. ASSESSMENT

The assessment of this lab will be based on your written report. Your assignment will be assessed according to the following criteria: 
* implementation of the missing routines;
* analysis of the test program output;
* explanation of how the program works; and
* clarify of explanation

### 6. SUBMISSION PROCEDURE

You must submit your lab assignment to Software Lab 1B before the due date. Late submission will be penalized and submission more than one week later will not be accepted. The deadline for submission is **ONE week after your last lab session**. Hence each lab group has a different deadline. You can check out your deadline from the lab submission due dates available in NTULearn.

Your hard-copy submission must include a cover page, clearly showing the code and name of the course (i.e., CE2005/CZ2005 Operating Systems), your name, and lab group number. Please slot in your assignment report according to your group number in the wooden shelf next to the lab main door.

### 7. PROBLEM STATEMENT

In this lab, you are required to complete a virtual memory implementation, including how to get a physical frame for a virtual page from the IPT if it exists there, how to put a physical frame/virtual page entry into TLB, and how to implement a least recently used page replacement algorithm.

A software-managed TLB is implemented in Nachos. There is one TLB per machine. There is also an IPT which maps physical frames to virtual pages. Basically, the translation process first examines the TLB to see if there is a match. If so, the matching entry in the TLB will be used for address translation. If there is a miss, the IPT will be looked up. If a matching entry is found in the IPT, the entry will be used to update the TLB. A miss in the IPT means that the page will have to be loaded from disk, and a page in and page out will be performed.

To decide which page to page out, a page replacement policy is used, for example, a least recently used algorithm which will be explained late on. During each lookup process, you need to perform some checking in order to make sure that you are looking up the correct entry and that the entry is valid.

In order to check whether you are referencing the correct entries from the TLB, you have to check the valid bit. The TLB will get updated when an exception is raised and the required page entry isn't in it. In this case, a new entry needs to be inserted into the TLB. The new entry will be inserted into an invalid entry in the TLB or replace an existing entry if it is full. Since the TLB is small, the replacement policy for the TLB is simply FIFO. When there is a context switch between processes, e.g. the main process executing a child process, the entries in the TLB will be cleared by setting all entries to invalid.

The IPT is simply implemented using an array, represented by the _memoryTable_ (a mapping of what pages are in memory and their properties). There is one entry for each of the physical frame, and each entry contains the corresponding process id, virtual page number, and the last used field that records the tick value when the page was last accessed. The least recently used algorithms works by iterating through the _memoryTable_, from the beginning, to look for the entry that has been least recently used. If there is an entry that is not valid (i.e., its process is dead), the algorithm will return the index of this invalid entry. Otherwise, the algorithm will return the index of the least recently used entry (that is, the entry with the smallest last used field).

### 8. EXERCISES

1. Change working directory to vm by typing `cd ~/nachos-3.4/vm`.

2. Modify the following methods in file _tlb.cc_. You may need to reference the following files: _tlb.h_, _ipt.h_, _ipt.cc_, _machine/translate.h_, _machine/translate.cc_, _machine/machine.h_, and _machine/disk.h_.
	
	* a. _int VpnToPhyPage(int vpn)_ - Gets a physical frame _phyPage_ for a virtual page _vpn_, if exists in the IPT.
	
	* b. _void InsertToTLB(int vpn, int phyPage)_ - Insert a vpn/phyPage entry into the TLB.
	
	* c. _int lruAlgorithm(void)_ - Return the freed physical frame according to the least recently used algorithm.

3. Compile Nachos by typing `make`. If you see "_ln -sf arch/intel-i386-linux/bin/nachos nachos_" at the end of the compiling output, your compilation is successful. If you encounter any anomalies, type make clean to remove all object and executable files and then type `make` again for a clean compilation.

4. Execute the test program to test whether the virtual memory is working properly by typing `./nachos -x ../test/vmtest.noff -d`. If the output of the test program is too long to be shown on the console, the output can be piped to a file by typing `./nachos –x ../test/vmtest.noff –d > output.txt`.

5. Fill the table (examples are shown below) whenever there is a TLB miss (i.e., the matching entry is not found in the TLB, but it could be found in the IPT) or an IPT miss (i.e., the requested page is not in the memory at all; it must be a page fault and must trigger page replacement). Fill as many rows as necessary until Nachos exits.

	The first three columns are the tick that the page fault exception occurred (tick), the virtual page number (vpn) and the corresponding process id (pid). Each process has its own set of virtual page numbers. The next four columns represent each entry in the IPT. Each IPT entry has four values, the process id (pid), virtual page number (vpn), last accessed tick (last used) and valid flag. The next three columns represent each entry in the TLB. Each TLB entry shows the virtual page number (vpn), physical frame number (phy), and the valid flag of the entry. The last column records the dirty page that is paged out, if any. You should record down the entries of the IPT and TLB **before** replacement, and **highlight** the entry that is selected to be updated.

	The first four rows of the table are shown below.
	
tick | vpn | pid | IPT\[0\] | IPT\[1\] | IPT\[2\] | IPT\[3\] | TLB\[0\] | TLB\[1\] | TLB\[2\] | Page Out 
-----|-----|-----|----------|----------|----------|----------|----------|----------|----------|----------
| | | | pid, vpn, last used, valid | | | | vpn, phy, valid| | | |
10 | 0 | 0 | ___0,0,0,0___ | 0,0,0,0 | 0,0,0,0 | 0,0,0,0 | ___0,0,0___ | 0,0,0| 0,0,0 |
13 | 9 | 0 | 0,0,12,1 | ___0,0,0,0___ | 0,0,0,0 | 0,0,0,0 | 0,0,1 | ___0,0,0___ | 0,0,0 | 
15 | 26 | 0 | 0,0,12,1 | 0,9,15,1 | ___0,0,0,0___ | 0,0,0,0 | 0,0,1 | 9,1,1 | ___0,0,0___ | 
20 | 1 | 0 | 0,0,12,1 | 0,9,19,1 | 0,26,17,1 | ___0,0,0,0___ | ___0,0,1___ | 9,1,1 | 26,2,1 | 

6. Write down the page size, the number of physical frames, and the TLB size defined in Nachos as well as the number of pages used by the test program and the number of TLB miss, page faults and page out occurred during the execution of the test program.

### 9. Report

It is recommended that your report should include analysis of the test program output. The analysis should clearly explain which TLB entry or physical frame should be used whenever there is a TLB miss or page fault. DO NOT attach the debugging printout from the program.
* Limit your report to 10 pages;
* Include your implementation of code fragments of _int VpnToPhyPage(int vpn)_, _void InsertToTLB(int vpn, int phyPage)_,and _int urlAlgorithm(void)_ in the report; and
* Leave a copy of your code in your account
