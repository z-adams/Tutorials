# Compute Shaders

Compute shaders are an invaluable tool that allows for the GPU to be used for
arbitrary computation.

## GPU Architecture

Before we begin, it's worth briefly reviewing the basic architecture of a GPU.
If you are already familiar with the concepts of vector processing (e.g.
SSE/AVX), feel free to skip this section.

### MIMD/SIMD

The power of GPUs comes from parallel computation, but the parallelism of a GPU
is somewhat different from that of a CPU.

A typical CPU has several physical cores, which have their own "program
counter." The program counter stores the current position in whatever code is
executing on the CPU. These separate program counters allow each CPU core to
execute a different part of the program, at the same time. This allows for true
multithreading, wherein a program uses multiple cores to do multiple, different
things at the same time. In Flynn's taxonomy, this is called **MIMD**, for
**M**ultiple **I**nstruction, **M**ultiple **D**ata. That is, the CPU can
perform completely separate tasks, on completely separate data, in parallel.

The GPU, in contrast, is a **SIMD** machine (**S**ingle **I**nstruction,
**M**ultiple **D**ata). Although the GPU does have multiple physical cores, its
primary computing power comes from the use of **vector processing.** In a SIMD
architecture, the processor has one program counter, but has vector registers
that can store many pieces of independent data. This arrangement allows for very
cheap and compact parallelism in the hardware, which is what allows GPUs to have
so many "cores."

Let's examine a pseudocode example. Imagine we have two arrays which we want to
add together:

```
int vec1[8] = {0, 1, 2, 3, 4, 5, 6, 7};
int vec2[8] = {7, 6, 5, 4, 3, 2, 1, 0};
```

Each array stores 64 integers, and we want to add `vec1` and `vec2` element-wise
and store the results in `vec3`. On the CPU, this would look something like:

```
int vec3[8];
for (i = 0; i < 8; i++)
{
    vec3[i] = vec1[i] + vec2[i];
}

// and the result we expect is:
vec3 == {7, 7, 7, 7, 7, 7, 7, 7};
```

This works fine, but it takes a lot of instructions as we have to loop over all
the array elements and sum them one-by-one. Let's see how vector processing
improves this issue.

Let's say our vector procesor has 8 "lanes" to store data. Each lane can store
a regular 32-bit variable, like an `int`. This means that each arithmetic
operation, such as `+`, can be executed on 8 values at the same time,
element-wise. Since our arrays are 8 elements wide, we can perform our
element-wise addition between the two arrays with *one instruction*, which is
much faster than looping over all the elements one-by-one.

We can represent this in pseudocode by storing our data in an `int_8x` type,
which will represent the fact that an array of 8 ints can be fit into one
primitive without needing arrays. Our code becomes:

```
int_8x vec1;
int_8x vec2;
int_8x vec3;

vec3 = vec1 + vec2;
```

Which emphasizes the fact that only one `+` operation is needed to add two sets
of 8 integers.

### Branching

Now let's say we have a loop like this:
```
int vec1[8] = {2, 2, 2, 2, 2, 2, 2, 2};
int vec2[8] = {3, 3, 3, 3, 3, 3, 3, 3};
int vec3[8];

for (i = 0; i < 8; i++)
{
    if (i is even)
    {
        vec3[i] = vec1[i];
    }
    else
    {
        vec3[i] = vec2[i];
    }
}

// result:
vec3 == {2, 3, 2, 3, 2, 3, 2, 3};
```

On a CPU, this is not a problem. The CPU looks at each value of `i`, and decides
what to do. On our vector hardware, on the other hand, this poses a serious
problem. Because our data is stored in 8-wide registers, we *must* process them
in groups of 8. In this case, however, the logic demands that different things
happen to the elements within our 8-wide vector register. How can we replicate
the CPU's behavior in this scenario?

The only way to do this, without inventing weird instructions, is by masking off
lanes based on the branch condition. In our case, based on the condition `i is
even`, elements 0, 2, 4, and 6  of `vec3` will come from `vec1`, and the rest
will come from `vec2`. We can create a "mask" variable which allows the
processor to disable or enable lanes for writing:

```
int_8x vec1 = {2, 2, 2, 2, 2, 2, 2, 2};
int_8x vec2 = {3, 3, 3, 3, 3, 3, 3, 3};
int_8x vec3 = {0, 0, 0, 0, 0, 0, 0, 0};

int_8x mask = {1, 0, 1, 0, 1, 0, 1, 0};

vec3 = (mask) ? vec1;
// vec3 is now {2, 0, 2, 0, 2, 0, 2, 0}
vec3 = (!mask) ? vec2;
// vec3 is now {2, 3, 2, 3, 2, 3, 2, 3}
```

Where `dst = (mask) ? src` represents setting elements of `dst` to `src` only if
`mask` has a `1` in that lane, and otherwise keeping the original value. On a
scalar processor, this operation would be described by the loop
```
for (i = 0; i < N; i++)
{
    if (mask[i] is true)
    {
        dst = src;
    }
    else
    {
        dst = dst;
    }
}
```
but on our vector architecture, it too happens with a single instruction.

This explains why diverging branches are bad on a GPU. If all 8 lanes of our
vector register take the same branch, it can be expressed as
```
if (condition)
{
    for (i = 0; i < 8; i++)
    {
        // do operation...
    }
}
```
which vectorizes nicely down to
```
if (condition)
{
    // perform one vector operation
}
```
This avoids executing our vector operation twice, since the condition is applied
uniformly to all lanes. There is nothing inherently bad about branching on a
GPU; for instance, in our mask scenario, if the mask was either all `1`s or all
`0`s, one of the operations would affect none of the lanes, and could be skipped
entirely. If that operation is an expensive function, branching to avoid calling
it may indeed improve performance.

### Execution Units

Now that we understand how a vector processor works, we can understand the basic
GPU architecture.

The GPU is full of **execution units** (EUs), which are effectively large SIMD
processors. The internal structure of the EU is usually complicated, but the EU
can effectively be thought of as a single SIMD processor. AMD's GCN architecture
EUs have 64 SIMD lanes, while NVidia's usually have 32. Each EU is similar to a
physical CPU core, in that it can run independently of the others. GPUs
typically contain between 10-50 EUs, although the latest generations are pushing
those numbers much higher. GPU "core" counts in the thousands are actually
counting the total number of SIMD lanes, not independent cores.

## Compute Shaders in OpenGL

Perhaps the most confusing aspect of compute shaders is the concept of "thread
groups." Regardless of language, most resources about GPU programming are
centered around the concept of thread groups, which may be 1, 2, or 3
dimensional collections of threads that perform your compute workload. In
reality, thread group indices are nothing more than a handy tool for organizing
your compute shader workload.

Let's say that I want to process an image that is 256x256 pixels, using a
compute shader. Like a fragment shader, we will write our compute shader such
that each invocation of the shader is responsible for processing a single pixel.
Our image has 65,536 pixels, so we will invoke 65,536 copies of our compute
shader, which will each run on one pixel.

Like in any language, we could pass our compute shader the width of our image
and retrieve pixels from our image with `image(x % width, y / width)`. Thread
groups provide a convenient system to automat this process.

```glsl
#version 430 core

layout(local_size_x = 8, local_size_y = 8, local_size_z = 1) in;
layout(location = 0, rgba8ui) uniform readonly image2D input_image;
layout(location = 0, rgba8ui) uniform writeonly image2D output_image;

vec3 process(vec3 pixel)
{
    // ...
}

void main()
{
    uvec2 xy = gl_GlobalInvocationID.xy;
    vec3 in_pixel = imageLoad(input_image, xy);

    out_pixel = process(in_pixel);
    imageStore(output_image, xy, out_pixel);
}
```

