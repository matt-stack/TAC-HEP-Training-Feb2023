## **1. Investigating Copy-Compute Overlap**

For your first task, you are given a code that performs a silly computation element-wise on a vector. You can initially compile, run and profile the code if you wish. 

compile it using the following:

```
nvcc -o overlap overlap.cu
```

In this case, the output will show the elapsed time of the non-overlapped version of the code. This code copies the entire vector to the device, then launches the processing kernel, then copies the entire vector back to the host.

You can also run this code with Nsight Systems if you wish:

```
nsys profile -o <destination_dir>/overlap.qdrep ./overlap
```

Note that you will have to copy this file over to your local machine and install Nsight Systems for visualization. You can download Nsight Systems here:
https://developer.nvidia.com/nsight-systems

This visual output should show you the sequence of operations (*cudaMemcpy* Host to Device, kernel call, and *cudaMemcpy* Device To Host). Note that there is an initial "warm-up" run of the kernel; disregard this. You should be able to witness that the start and duration of each operating is indicating that there is no overlap.

Your objective is to create a fully overlapped version of the code. Use your knowledge of streams to create a version of the code that will issue the work in chunks, and for each chunk perform the copy to device, kernel launch, and copy to host in a single stream, then modifying the stream for the next chunk. The work has been started for you in the section of code after the #ifdef statement. Look for the FIXME tokens there, and replace each FIXME with appropriate code to complete this task.

When you have something ready to test, compile with this additional switch:

```
nvcc -o overlap overlap.cu -DUSE_STREAMS
```

If you run the code, there will be a verification check performed, to make sure you have processed the entire vector correctly, in chunks. If you pass the verification test, the program will display the elapsed time of the streamed version.  You should be able to get to at least 2X faster (i.e. half the duration) of the non-streamed version. If you wish, you can also run this code with the Nsight Systems profiler using the above given command. This will generate a visual output, and you should be able to confirm that there is indeed overlap of operations by zooming in on the portion of execution related to kernel launches. You can see the non-overlapped version run, followed by the overlapped version. Not only should the overlapped version be faster, you should see an interleaving of computation and data transfer operations.

If you need help, refer to *overlap_solution.cu*.

## **2. Cuda Event Timing**

In this exercise, we the same code from the overlap exercise, but we want to replace the current timing system with Cuda Streams. Please look through the code, and tackle the FIXMEs placed throughout. Please refer to the API documentation for review: https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__EVENT.html#group__CUDART__EVENT 

You can compile the code with:

```
nvcc -o event_timing event_timing.cu
```

You can run the code with:

```
./event_timing
```
## **3. Default Stream + Nsys**

In this exercise, we still have the same code from the overlap exercise except for one change, take a look now at default_stream.cu and see if you can spot the small difference between the last exercise. Answer: We have taken the stream argument out of the kernel launch, and are back to launching in the default stream. This is purely for learning purposes, I do not recommend launching kernels to the default stream when you are aiming for concurrency with Streams. Using Nsight Systems, we want to profile the code and see the impact that launching work to the default stream had our overlapping memory operations and compute. 


You can compile the code with:

```
nvcc -o default_stream default_stream.cu
```

And you can profile it with

```
nsys profile -o default_stream_profile ./default_stream
```

If you are on a remote system, you will need to copy the nsys-rep file to your local machine and open it up in the Nsys GUI. Compare this result with a profile of exercise 2.

Also to note, we CANT get back the same operation we had before by simply converting the default to act as a normal stream. You can test this with:
```
nvcc --default-stream per-thread -o default_stream_per_thread default_stream.cu
```

And then take a profile, you should see all the kernels execute immediately without waiting for the data, as they are out of sync with the streams we had before!
