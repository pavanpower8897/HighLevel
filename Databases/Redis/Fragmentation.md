-> Memory fragmentation is the metric of free memory which is not allocated.

There are two type fragmentations
1. External fragmentation -> Which consider the total partitions/page sizes which are completely free
2. Internal fragmentation -> Which considers the remaining free spaces in partitions which are filled partially

Questions: How redis does the defragmentations ?

Suppose if the page size is 4KB 
Then whenever redis trying to store the string size of 2KB it request for a new page from OS page pool and it will allocated to redis and in that it will use 2KB, Suppose in the next request to store 4KB it checks whether already allocated pages has sufficient free space to store if not it request for new page
Suppose if we have lot of intermented free spaces which redis cant use to store most of the data requests then it keeps on requesting new pages which is not memory optimized also we call it increase in fragmentation and to optimize this it uses defragementation and reshuffle the data to store in consicutive pages mostly. 

If the data size is greater than4kb page size then it requests for multiple consecutive pages vertiually not phsysicall consecutive (Like consecutive page ids) and store that suppose for 10KB (4kb+4kb + 2kb) now 2kb will be free in one page.


<img width="933" alt="Screenshot 2024-05-23 at 7 28 35 PM" src="https://github.com/pavanpower8897/HighLevel/assets/44682188/90c1f9b4-a362-4131-9b89-ee35c62e2648">
