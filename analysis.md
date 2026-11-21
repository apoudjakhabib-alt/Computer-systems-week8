Cache Behavior Lab Analysis
Section 1: Performance Measurements
Matrix Size	Row-Major (ms)	Column-Major (ms)	Ratio (Column/Row)
1000 x 1000	4.00	3.00	0.75x
2000 x 2000	21.00	19.00	0.90x
4000 x 4000	33.00	163.00	4.94x
8000 x 8000	175.00	733.00	4.19x
As the matrix size increased, the performance difference became much more noticeable. For smaller matrices the execution times were similar, but for larger matrices row-major traversal was over four times faster than column-major traversal. 


 Optimization Level Comparison
 
Optimization Level	Row-Major (8000x8000)	Column-Major (8000x8000)	Ratio
O0	225 ms	789 ms	3.51x
O2	44 ms	1283 ms	29.16x
O3	36 ms	1114 ms	30.94x
The O3 optimization level does not eliminate the performance gap between row-major and column-major traversal. While compiler optimizations significantly reduce execution times, row-major traversal remains much faster because it accesses memory in a cache-friendly order. The compiler can optimize instructions and loop execution, but it cannot change the fundamental memory access pattern without changing the behavior of the program.


 Assembly Analysis

See Screenshot 2026-11-21 013303.png (Row-Major Assembly)

See Screenshot 2026-11-21 013342.png (Column-Major Assembly)

The generated assembly code for the row-major and column-major functions is very similar. Both functions use nested loops and perform the same summation operation on the matrix elements. Although the assembly instructions are nearly identical, the measured performance is very different.

This indicates that the performance difference does not originate from the machine instructions themselves. Instead, the difference comes from how each function accesses memory. Row-major traversal accesses memory sequentially, which is cache friendly, while column-major traversal accesses memory with large jumps between elements, causing more cache misses and slower execution.


 Cache Behavior Explanation

In C, two-dimensional arrays are stored in row-major order. This means that all elements in a row are stored next to each other in memory, followed by the elements of the next row. When a program traverses a matrix using row-major order, it accesses memory sequentially. Because neighboring elements are stored close together, the CPU can efficiently load multiple values at once into a cache line.

During row-major traversal, when the CPU fetches a cache line from memory, that cache line contains several nearby elements that will be used immediately. As the inner loop moves across a row, most memory accesses are already available in the cache. This reduces the number of expensive memory accesses and improves overall performance. The performance measurements showed that row-major traversal becomes significantly faster on larger matrices because it takes advantage of spatial locality.

Column-major traversal behaves differently. Although it performs the same computation, it accesses elements from different rows before moving to the next column. Because rows are stored contiguously in memory, each access may jump a large distance from the previous one. This causes the CPU to load many cache lines that contain mostly unused data. As a result, more cache misses occur and execution time increases significantly for large matrices.

The compiler cannot completely solve this issue through optimization. Compiler optimizations can improve 