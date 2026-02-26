Last name of Student 1: Houlihan
First name of Student 1: Kai
Email of Student 1: khoulihan@ucsb.edu
Last name of Student 2: Vannier
First name of Student 2: Olivier
Email of Student 2: ovannier@ucsb.edu

See the description of this assignment  for detailed reporting requirements 


Part B

Q2.a List parallel code that uses at most two barrier calls inside the while loop

void work_block(long my_rank)
{
	double local_error = __DBL_MAX__;
	int block_size = ceil((double)matrix_dim/thread_count);
	int k = 0;
	int start = my_rank * block_size;
	int end = ((start + block_size) > matrix_dim) ? matrix_dim : (start + block_size);
	while (k < no_iterations) {
		int i = start;
		while (i < end) {
			mv_compute(i);
			i++;
		}
		//pthread_barrier_wait(&mybarrier);
		double temp_error = 0;
		double glob_error = 0;
		for (int p = 0; p < matrix_dim; p++) {
			temp_error = fabs(vector_y[p] - vector_x[p]);	
			if (temp_error > glob_error) {
				glob_error = temp_error;
			}
		} 
		//pthread_barrier_wait(&mybarrier);
		local_error = glob_error;
		k++;
		if (local_error < ERROR_THRESHOLD) {
			break;
		}
		pthread_barrier_wait(&mybarrier);
		for (int p = start; p < end; p++) {	
			vector_x[p] = vector_y[p];
		}
		pthread_barrier_wait(&mybarrier);
	}
}

void work_blockcyclic(long my_rank) { 	
	double local_error = __DBL_MAX__;
	int k = 0, i = 0, start = 0;

	while (k < no_iterations) {
		start = my_rank * cyclic_blocksize;
		while (start < matrix_dim) {
			i = start;
			while (i < start + cyclic_blocksize && i < matrix_dim) {
				mv_compute(i);
				i++;
			}
			start += (thread_count * cyclic_blocksize);
		}
		double temp_error = 0;
		double glob_error = 0;
		for (int p = 0; p < matrix_dim; p++) {
			temp_error = fabs(vector_y[p] - vector_x[p]);	
			if (temp_error > glob_error) {
				glob_error = temp_error;
			}
		} 
		local_error = glob_error;
		k++;
		if (local_error < ERROR_THRESHOLD) {
			break;
		}
		start = my_rank * cyclic_blocksize;
		pthread_barrier_wait(&mybarrier);
		while (start < matrix_dim) {
			i = start;
			while (i < start + cyclic_blocksize && i < matrix_dim) {
				vector_x[i] = vector_y[i];
				i++;
			}
			start += (thread_count * cyclic_blocksize);
		}
		pthread_barrier_wait(&mybarrier);
	}
}

Q2.b Report parallel time, speedup, and efficiency for  the upper triangular test matrix case when n=4096 and t=1024. 
Use 2 threads and 4  threads (1 thread per core) under blocking mapping, and block cyclic mapping with block size 1 and block size 16.    
Write a short explanation on why one mapping method is significantly faster than or similar to another.

Stats:

This section was done with CSIL. The uptime stats are: up 14:45, 11 users, load average: 0.15, 0.35, 0.48 

1 Thread (Control):
Test 9: n=4K t=1K blockmapping: Latency = 25.756490 sec with 1 threads. Matrix dimension 4096 
Test 10: n=4K t=1K block cylic (r=1): Latency = 23.571512 sec with 1 threads. Matrix dimension 4096 
Test 11: n=4K t=1K block cyclic (r=16): Latency = 25.508964 sec with 1 threads. Matrix dimension 4096 
Test 12: n=4K t=1K upper block mapping: Latency = 0.106858 sec with 1 threads. Matrix dimension 4096 
Test 13: n=4K t=1K upper block cylic (r=1): Latency = 0.105930 sec with 1 threads. Matrix dimension 4096 
Test 14: n=4K t=1K upper block cyclic(r=16): Latency = 0.107552 sec with 1 threads. Matrix dimension 4096 
Summary: Failed 0 out of 6 tests

2 Threads:
Test 9: n=4K t=1K blockmapping: Latency = 13.922183 sec with 2 threads. Matrix dimension 4096 
Test 10: n=4K t=1K block cylic (r=1): Latency = 13.844523 sec with 2 threads. Matrix dimension 4096 
Test 11: n=4K t=1K block cyclic (r=16): Latency = 13.338632 sec with 2 threads. Matrix dimension 4096 
Test 12: n=4K t=1K upper block mapping: Latency = 0.080051 sec with 2 threads. Matrix dimension 4096 
Test 13: n=4K t=1K upper block cylic (r=1): Latency = 0.067184 sec with 2 threads. Matrix dimension 4096 
Test 14: n=4K t=1K upper block cyclic(r=16): Latency = 0.059207 sec with 2 threads. Matrix dimension 4096 
Summary: Failed 0 out of 6 tests
    - Speedup (Tests 9, 10, 11, 12, 13, 14): 1.85, 1.71, 1.91, 1.33, 1.58, 1.82
    - Efficiency (Tests 9, 10, 11, 12, 13, 14): .92, .85, .96, .67, .79, .91

4 Threads:
Test 9: n=4K t=1K blockmapping: Latency = 6.519421 sec with 4 threads. Matrix dimension 4096 
Test 10: n=4K t=1K block cylic (r=1): Latency = 6.699975 sec with 4 threads. Matrix dimension 4096 
Test 11: n=4K t=1K block cyclic (r=16): Latency = 6.702075 sec with 4 threads. Matrix dimension 4096 
Test 12: n=4K t=1K upper block mapping: Latency = 0.043481 sec with 4 threads. Matrix dimension 4096 
Test 13: n=4K t=1K upper block cylic (r=1): Latency = 0.032513 sec with 4 threads. Matrix dimension 4096 
Test 14: n=4K t=1K upper block cyclic(r=16): Latency = 0.030507 sec with 4 threads. Matrix dimension 4096 
Summary: Failed 0 out of 6 tests
    - Speedup (Tests 9, 10, 11, 12, 13, 14): 3.95, 3.51, 3.81, 2.45, 3.26, 3.53
    - Efficiency (Tests 9, 10, 11, 12, 13, 14): .99, .88, .95, .61, .82, .88

Report:
Block-cyclic mapping is faster than block mapping as it allows for a better distribution of data across threads as well as
greater parallelism across threads. Both methods take advantage of block mapping's memory/cache blocking where each 
thread can access data in succession in memory. However, block-cyclic mapping differs as adopting a cyclic approach
to block mapping allows for greater load balancing as the round-robin style can distribute the blocks more evenly. 
Block mapping having to operate on large chunks of data isn't great for load balancing especially when the matrix size
cannot be divided evenly by the number of threads available.



-----------------------------------------------------------------
Part C

1. Report what code changes you made for blasmm.c. 

# CHANGES
        const double *B_col = &B[j * LDB];
        double *C_col = &C_dgemv[j * LDC];
        //Your solution to call cblas_dgemv
        cblas_dgemv(CblasColMajor, CblasNoTrans, M, K, 1.0, A, LDA, B_col, INCX, 0.0, C_col, INCY);


2. Conduct a latency and GFLOPS comparison of the above 3 when matrix dimension N varies as 50, 200, 800, and 1600. 
Run the code in one thread and 8 threads on an AMD CPU server of Expanse.
List the latency and GFLOPs of  each method in each setting.  
Explain why when N varies from small to large,  Method 1 with GEMM starts to outperform others. 

# 1 THREAD
Maximum number of threads allowed for MKL: 1
--- Matrix Multiplication Performance Comparison ---
Matrix size: N=50 x N=50

MKL DGEMM     : Time 0.001074 sec. GFLOPS 0.23.  0.07x
MKL DGEMV Loop: Time 0.000037 sec. GFLOPS 6.77.  1.97x
Naive 3 loops : Time 0.000073 sec. GFLOPS 3.43.  1.00x

Mid-point verification looks OK: DGEMM=10.4441, DGEMV=10.4441, Naive=10.4441

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=200 x N=200

MKL DGEMM     : Time 0.000883 sec. GFLOPS 18.12.  3.89x
MKL DGEMV Loop: Time 0.001292 sec. GFLOPS 12.38.  2.66x
Naive 3 loops : Time 0.003431 sec. GFLOPS 4.66.  1.00x

Mid-point verification looks OK: DGEMM=45.1042, DGEMV=45.1042, Naive=45.1042

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=800 x N=800

MKL DGEMM     : Time 0.027127 sec. GFLOPS 37.75.  13.59x
MKL DGEMV Loop: Time 0.085436 sec. GFLOPS 11.99.  4.31x
Naive 3 loops : Time 0.368653 sec. GFLOPS 2.78.  1.00x

Mid-point verification looks OK: DGEMM=201.3727, DGEMV=201.3727, Naive=201.3727

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=1600 x N=1600

MKL DGEMM     : Time 0.191995 sec. GFLOPS 42.67.  36.43x
MKL DGEMV Loop: Time 1.255637 sec. GFLOPS 6.52.  5.57x
Naive 3 loops : Time 6.994009 sec. GFLOPS 1.17.  1.00x

Mid-point verification looks OK: DGEMM=405.6966, DGEMV=405.6966, Naive=405.6966

# 8 THREADS
--- Matrix Multiplication Performance Comparison ---
Matrix size: N=50 x N=50

MKL DGEMM     : Time 0.056254 sec. GFLOPS 0.00.  0.01x
MKL DGEMV Loop: Time 0.000037 sec. GFLOPS 6.77.  12.75x
Naive 3 loops : Time 0.000471 sec. GFLOPS 0.53.  1.00x

Mid-point verification looks OK: DGEMM=10.4441, DGEMV=10.4441, Naive=10.4441

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=200 x N=200

MKL DGEMM     : Time 0.010691 sec. GFLOPS 1.50.  1.26x
MKL DGEMV Loop: Time 0.001294 sec. GFLOPS 12.36.  10.45x
Naive 3 loops : Time 0.013520 sec. GFLOPS 1.18.  1.00x

Mid-point verification looks OK: DGEMM=45.1042, DGEMV=45.1042, Naive=45.1042

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=800 x N=800

MKL DGEMM     : Time 0.042568 sec. GFLOPS 24.06.  10.19x
MKL DGEMV Loop: Time 0.245853 sec. GFLOPS 4.17.  1.76x
Naive 3 loops : Time 0.433698 sec. GFLOPS 2.36.  1.00x

Mid-point verification looks OK: DGEMM=201.3727, DGEMV=201.3727, Naive=201.3727

--- Matrix Multiplication Performance Comparison ---
Matrix size: N=1600 x N=1600

MKL DGEMM     : Time 0.339796 sec. GFLOPS 24.11.  20.66x
MKL DGEMV Loop: Time 1.583653 sec. GFLOPS 5.17.  4.43x
Naive 3 loops : Time 7.020562 sec. GFLOPS 1.17.  1.00x

Mid-point verification looks OK: DGEMM=405.6966, DGEMV=405.6966, Naive=405.6966

# Analysis
When N increases, Method with via GEMM outperforms the others because it maintains cache locality by performing the computations 
using blocking which decreases cache misses. When N increases, we amplify the amount of cache misses because we don't have to reload 
every column back into cache, so GEMM will drastically outperform GEMV or the naive method.
