Last name of Student 1: Vannier
First name of Student 1: Olivier
Email of Student 1: ovannier@ucsb.edu
Last name of Student 2: Houlihan
First name of Student 2: Kai
Email of Student 2: khoulihan@ucsb.edu


If CSIL is used for performance assessment instead of Expanse, make sure you evaluate when such a machine is lightly 
loaded using “uptime”. Please  indicate your evaluation is done on CSIL and list the uptime index of that CSIL machine.  

Report 
----------------------------------------------------------------------------
1. How is the code parallelized? Show your solution by listing the key computation parallelized with
  OpenMP and related code. 

```cpp

void parallel_itmv_mult(int threadcnt, int mappingtype, int chunksize) {
  /*Your solutuion with OpenMP*/
  int i, k;

#pragma omp parallel num_threads(threadcnt) private(k)
  {

    for (k = 0; k < no_iterations; k++) {
      if (mappingtype == BLOCK_DYNAMIC) {
#pragma omp for schedule(dynamic, chunksize)
        for (i = 0; i < matrix_dim; i++) {
          mv_compute(i);
        }
      } else if (mappingtype == BLOCK_CYCLIC) {
#pragma omp for schedule(static, chunksize)
        for (i = 0; i < matrix_dim; i++) {
          mv_compute(i);
        }
      } else if (mappingtype == BLOCK_MAPPING) {
#pragma omp for schedule(static)
        for (i = 0; i < matrix_dim; i++) {
          mv_compute(i);
        }
      }
#pragma omp for
      for (i = 0; i < matrix_dim; i++) {
        vector_x[i] = vector_y[i];
      }
      // end
    }
  }
}


----------------------------------------------------------------------------
2.  Report the parallel time, speedup, and efficiency with blocking mapping, block cyclic mapping with block size 1 
and block size 16 using  2 cores (2 threads), and 4 cores (4 threads) for parallelizing the code 
in handling a full dense matrix with n=4096 and t=1024. 

Test 9: n=4K t=1K blockmapping: Latency = 7.765898 sec and 4.4244 GFLOPS with 4 threads. Matrix dimension 4096
Test 10: n=4K t=1K block cylic (r=1): Latency = 8.703587 sec and 3.9478 GFLOPS with 4 threads. Matrix dimension 4096
Test 11: n=4K t=1K block cyclic (r=16): Latency = 7.951402 sec and 4.3212 GFLOPS with 4 threads. Matrix dimension 4096
Test 12: n=4K t=1K upper block mapping: Latency = 6.597037 sec and 2.6048 GFLOPS with 4 threads. Matrix dimension 4096
Test 13: n=4K t=1K upper block cylic (r=1): Latency = 4.543804 sec and 3.7819 GFLOPS with 4 threads. Matrix dimension 4096
Test 14: n=4K t=1K upper block cyclic(r=16): Latency = 3.752679 sec and 4.5791 GFLOPS with 4 threads. Matrix dimension 4096
Test 14a: n=4K t=1K upper dynamic(r=16): Latency = 5.109253 sec and 3.3633 GFLOPS with 4 threads. Matrix dimension 4096
Summary: Failed 0 out of 7 tests

21:06:41 up 16:06,  8 users,  load average: 1.79, 0.56, 0.24

----------------------------------------------------------------------------
3.  Report the parallel time, speedup, and efficiency with blocking mapping, block cyclic mapping with block size 1 
and block size 16 using  2 cores (2 threads), and 4 cores (4 threads) for parallelizing the code 
in handling an upper triangular matrix (n=4096 and t=1024).

Write a short explanation on why one mapping method is significantly faster than or similar to another.
