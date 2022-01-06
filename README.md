# Memory Management 

<b>Note: When explaining the algorithm I will only include code I added to the skeleton code.</b>
## FIFO (First In First Out) Replacement Scheme

No extra data structures were needed for my implementation. Instead of using a queue to keep track of the next entry to replace, I added a rank attribute to the page table entries that allowed me to keep track of that information. This rank attribute was set to a global rank variable that incremented as entries were swapped in to the table. The smaller the rank value the earlier the entry was added to the table.

### Algorithm

<b>If there is a frame hit:</b>
<br/>
<i>If it was a memory write, set the entry’s dirty bit to true</i>
```c
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
```

<b>If an empty frame is found:</b>
<br/>
<i>After page is added to the able, set it's rank to the current global rank variable and increment the global rank variable. If it was a memory write, set the dirty bit to true.</i>
```c
page_table[frame].rank = rank++;
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
```

<b>Else if no empty frame is found:</b>
<br/>
<i>Iterate through the page table and find index of the entry with the smallest rank (the first entry added)</i>
```c
int target_index = 0;
for ( i = 0; i < size_of_memory; i++) {
     if (page_table[i].rank < page_table[target_index].rank) {
          target_index = i;
     }
}
```
<i>Replace the entry at the chosen frame with the new page, set it's rank to the global rank varable and increment the global rank variable</i>
```c
page_table[target_index].page_num = page;
page_table[target_index].rank = rank++;
```
<i>If the page has been written to, increment the number of swap outs. If the entry added is a memory write, set it's dirty bit to true, if not set it to false. Incremented the number of sawp ins.</i>
```c
if(page_table[target_index].dirty == 1){
     swap_outs++;
}
if(memwrite == 1){
page_table[target_index].dirty = 1;    
}
else {
     page_table[target_index].dirty = 0; 
}
swap_ins++;
```

## LRU (Least Recently Used) Replacement Scheme

No extra data structures were needed for my implementation. Instead of using a queue to keep track of the least recently used entry, I added a rank attribute to the page table entries that allowed me to keep track of that information. This rank attribute was set to a global rank variable that incremented as entries were swapped in to the table or if a frame hit occurs. The smaller the rank value the longer it has been since that value was used.

### Algorithm

<b>If there is a frame hit:</b>
<br/>
<i>If it was a memory write, set the entry’s dirty bit to true. Update that entry's rank as it has just been used and should therfore have the largest rank value.</i>
```c
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
page_table[frame].rank = rank++;
```

<b>If an empty frame is found:</b>
<br/>
<i>After page is added to the able, set it's rank to the current global rank variable and increment the global rank variable. If it was a memory write, set the dirty bit to true.</i>
```c
page_table[frame].rank = rank++;
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
```

<b>Else if no empty frame is found:</b>
<br/>
<i>Iterate through the page table and find index of the entry with the smallest rank (the least recently used)</i>
```c
int target_index = 0;
for ( i = 0; i < size_of_memory; i++) {
     if (page_table[i].rank < page_table[target_index].rank) {
          target_index = i;
     }
}
```
<i>Replace the entry at the chosen frame with the new page, set it's rank to the global rank varable and increment the global rank variable</i>
```c
page_table[target_index].page_num = page;
page_table[target_index].rank = rank++;
```
<i>If the page has been written to, increment the number of swap outs. If the entry added is a memory write, set it's dirty bit to true, if not set it to false. Incremented the number of sawp ins.</i>
```c
if(page_table[target_index].dirty == 1){
     swap_outs++;
}
if(memwrite == 1){
page_table[target_index].dirty = 1;    
}
else {
     page_table[target_index].dirty = 0; 
}
swap_ins++;
```
<b>Note: The only difference between this algorithm and fifo is the addition of `page_table[frame].rank = rank++;` if a frame hit occurs.</b>

## Second Chance Replacement Scheme

No extra data structures were needed for my implementation. Instead of using a queue to keep track of the first entry that was added,  I added global replacement index variable that keeps track of which entry to try and replace next. I added a reference attibute to the page table entries to keep track of if an entry has been referenced. If the entry at the replacement index has it's reference bit set to true, I will check the next item in table until I find one with it's reference bit set to false. This allows the entries that have been referenced a second chance in the table.

### Algorithm

<b>If there is a frame hit:</b>
<br/>
<i>If it was a memory write, set the entry’s dirty bit to true. Set the reference bit to true.</i>
```c
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
page_table[frame].ref = 1;
```

<b>If an empty frame is found:</b>
<br/>
<i>After page is added to the able, set it's reference bit to true. If it was a memory write, set the dirty bit to true</i>
```c
page_table[frame].ref = 1;
if(memwrite == 1){
     page_table[frame].dirty = 1;
}
```

<b>Else if no empty frame is found:</b>
<br/>
<i>Iterate through the page table starting at the replacement index, if the reference bit is false, chose that index. Otherwise
set the ref bit to false and continue (giving the entry a second chance). If you have gone through the whole table and not found an entry with a reference bit set to false, iterate through the table again.</i>
```c
int i = replacement_index;
while(i < size_of_memory) {
     if (page_table[i].ref == 0) {
          replacement_index = i;
          break;
     }
     if(page_table[i].ref == 1) {
          page_table[i].ref = 0;
     }
     i++;
     if(i == size_of_memory){
     i = 0;
     }
}
```
<i>Replace the entry at the chosen frame with the new page, set it's reference bit to true</i>
```c
page_table[replacement_index].page_num = page;
page_table[replacement_index].ref = 1;
```
<i>If the page has been written to, increment the number of swap outs. If the entry added is a memory write, set it's dirty bit to true, if not set it to false. Incremented the number of sawp ins.</i>
```c
if(page_table[replacement_index].dirty == 1){
     swap_outs++;
}
if(memwrite == 1){
page_table[replacement_index].dirty = 1;    
}
else {
     page_table[replacement_index].dirty = 0; 
}
swap_ins++;
```
<i>Set the repalcement index to the next item in the table and make sure it does not exceed the size of the table</i>
```c
replacement_index++;
replacement_index = replacement_index % size_of_memory;
```

## Implementation Strategies and Advantages

### FIFO 

The traditional implementation for this replacement scheme is to use queue to keep track of the order in which pages are added to the page table. Instead of this, I used a rank attibute and global variable as describes above. 

<b>Here are the advantages of using this implementation instead of a queue:</b>

<b>1) Smaller Space Complexity</b> <br/>
My implementation does not require any extra data structures. If I were to use a queue, I would need extra storage. The size of the queue would be the same size as the page table (or the value of the numframes parameter input in the command line). As the size of the page table increases, using the extra storage could become more undesirable. I was able to avoid this by adding the rank attribute to the existing page table entries.

<b>2) Faster Run Time</b><br/>
My implementation only requires one iteration through the page table to find the frame of the entry with the smallest rank. If I were to use an array implementation of a queue, I would need to first need to remove the front element, shift over all the elements in the queue and add the removed element to the back of the queue. I would then need to iterate through the page table to find the frame of the page to be replaced.

### LRU 

My strategy for the LRU replacement scheme, was to impelement a simple algorithm and build of my previous fifo algorithm implementation. I realized that fifo and lru are quite similar. The main difference is that fifo only cares about when a page is swapped in to a table and lru also cares if frame hit occurs on a page. This allowed me to reuse my code from fifo and simply add one line of code to update the rank of an entry if a frame hit occurs.

<b>Here are the advantages of using this implementation instead of a queue:</b>

<b>1) Smaller Space Complexity</b> <br/>
Same explination as the fifo replacement scheme.

<b>2) Faster Run Time</b><br/>
Same explination as the fifo replacement scheme.

### Second Chance 

The traditional implementation for this replacement scheme is to use queue to keep track of the order in which pages were added to the table like in the fifo repalcement scheme. Instead of this, I added a global replacement index that kept track of which frame to attempt to replace in the page table. The index is always set to the entry in the table after the entry that was replaced, this is similar to adding that entry to the back of a queue since it will only reach that entry once it has gone through the rest of the table and has looped back to that entry.  

<b>Here are the advantages of using this implementation instead of a queue:</b>

<b>1) Smaller Space Complexity</b> <br/>
Same explination as the fifo replacement scheme.

<b>2) Faster Run Time</b><br/>
My implementation only requires one iteration through the page table to find the first entry in the table with the reference bit set to false, starting at the repalcement index. The only time when the loop runs more than the size of the page table is when the first page fault occurs with no free frame. In this case all the page table entries reference bits are set to true. Therefore, it will iterate through all the elements, setting their reference bits to false and ending up back at the begining of the table where that entry’s reference bit is now false. So in this situation the loop will run `size_of_memory + 1` times. If I were to use an array implementation of a queue, I would need to first need iterate through the queue, checking each elements reference bit until I found an entry with a reference bit set to false. I would then need to iterate through the page table to find the frame of the page to be replaced.
