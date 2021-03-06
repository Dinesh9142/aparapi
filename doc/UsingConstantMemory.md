#UsingConstantMemory
*How to make use of constant memory in a Kernel Updated Feb 28, 2012 by frost.g...@gmail.com*
##How to make use of new constant memory feature
By default all primitive arrays accessed by an Aparapi Kernel is considered global. If we look at the generated code using `-Dcom.amd.aparapi.enableShowGeneratedOpenCL=true` we will see that primitive arrays (such as `int buf[]`) are mapped to `__global` pointers (such as `__global int *buf`) in OpenCL.

Although this makes Aparapi easy to use (especially to Java developers who are unfamiliar to tiered memory hierarchies), it does limit the ability of the 'power developer' wanting to extract more performance from Aparapi on the GPU.

This [page](http://www.amd.com/us/products/technologies/stream-technology/opencl/pages/opencl-intro.aspx?cmpid=cp_article_2_2010) from AMD's website shows the different types of memory that OpenCL programmers can exploit.

Global memory buffers in Aparapi (primitive Java arrays) are stored in host memory and are copied to Global memory (the RAM of the GPU card).

Local memory is 'closer' to the compute devices and not copied from the host memory, it is just allocated for use on the device. The use of local memory on OpenCL can lead to much more performant code as the cost of fetching from local memory is much lower.

Local memory is shared by all work item's (kernel instances) executing in the same group. This is why the use of local memory was deferred until we had a satisfactory mechanism for specifying a required group size.

We recently also added support for constant memory for data that needs to be written once to the GPU but will not change.

Aparapi only supports constant arrays, not scalers.

##How to define a primitive array as "constant"
We have two ways define a constant buffer. Either we can decorate the variable name with a _$constant$ suffix (yes it is a valid identifier n Java).

    final int[] buffer = new int[1024]; // this is global accessable to all work items.
    final int[] buffer_$constant$ = new int[]{1,2,3,4,5,6,7,8,9} // this is a constant buffer

    Kernel k = new Kernel(){
        public void run(){
             // access buffer
             // access buffer_$constant$
             // ....
        }
    }

Alternatively (if defining inside the derived Kernel class - cannot be used via anonymous inner class pattern above!) we can can use the @Constant annotation.

    final int[] buffer = new int[1024]; // this is global accessable to all work items.

    Kernel k = new Kernel(){
        @Constant int[] constantBuffer = new int[]{1,2,3,4,5,6,7,8,9} // this is a constant buffer
        public void run(){
             // access buffer
             // access constantBuffers
             // ....
        }
    }

##Can I see some code?
I updated the Mandelbrot example so that the pallete of RGB values is represented using constant memory, the source can be found here. Look at line #95. BTW for me this resulted in a 5-7 % performance improvement.

[http://code.google.com/p/aparapi/source/browse/trunk/samples/mandel/src/com/amd/aparapi/sample/mandel/Main.java](tp://code.google.com/p/aparapi/source/browse/trunk/samples/mandel/src/com/amd/aparapi/sample/mandel/Main.java)