Simple Mental Model

Think of it like this:

Row storage = optimized for writing transactions

Column storage = optimized for reading analytics

Analytics = scan big data → aggregate → filter
Columnar storage was built exactly for that.



5️⃣ The Big Takeaway

Columnar storage exists because:

Analytics workloads read a few columns across many rows.

So storing data by column:

Reduces I/O

Improves compression

Speeds up queries

Lowers cost





Columnar layout makes this possible because: Data is contiguous in memory How actually it stores lesser memory footprint ?

Contiguous Layout ≠ Automatically Smaller

Just storing data contiguously does NOT automatically reduce size.

If you store:

[100, 50, 120]
That alone doesn’t save memory.

The savings come from:

Compression efficiency

Encoding techniques

Reduced memory loading

Better CPU cache behavior



2️⃣ Why Columnar Enables Better Compression

Think about a Row Store

Imagine this table:

id	country	status	amount
1	India	active	100
2	India	active	200
3	India	inactive	300
Row-based storage in memory:

1,India,active,100
2,India,active,200
3,India,inactive,300
Notice how "India" and "active" are scattered.

The compression engine sees mixed types:

int

string

string

int

Patterns are broken.

Harder to compress efficiently.





Columnar Layout

    ↓

Better Encoding

    ↓

Better Compression

    ↓

Less I/O

    ↓

Better Cache Efficiency

    ↓

Vectorized Execution

    ↓

Massive Parallelism



Actually here compression algorithm not alone decides, instead data storage layout also matters for reducing the memory foot print 

Here after better cache efficiency because of encoding and cpu cache, How vectorized execution and massive parllelism only true for columnor instead of row ?

Short answer:

👉 They can do it.
👉 But columnar layout makes it natural, efficient, and scalable.
👉 Row layout makes it expensive and awkward.

Now let’s go deep.

1️⃣ Vectorized Execution — What It Really Means

Vectorized execution means:

Instead of:

for each row:
    if amount > 100:
        count++
We do:

Compare 1024 values at once using SIMD CPU instructions
Modern CPUs have SIMD registers (AVX2, AVX-512) that can process:

8

16

32

even 64 values in one instruction

2️⃣ Why Columnar Layout Makes Vectorization Natural

Imagine a column:

amount:
100, 200, 50, 300, 80, 400
Memory layout:

[100][200][50][300][80][400]
All values:

Same type

Same size

Contiguous

No interruptions

CPU can load:

[100, 200, 50, 300]
into one SIMD register and compare all 4 at once.

Perfect.



Row layout memory:

[1, India, 100]
[2, US, 200]
[3, India, 50]
[4, UK, 300]
In memory:

1, India, 100, 2, US, 200, 3, India, 50, 4, UK, 300
To vectorize amount > 100:

Engine must:

Read row

Skip id

Skip country

Extract amount

Store amount in temp buffer

Repeat

Then vectorize

That extraction step kills efficiency.







SIMD works best when: Memory is aligned Data is contiguous No branching No type heterogeneity How SIMD acheives this 
Excellent. Now we’re at CPU micro-architecture level 👏
Let’s go step by step and remove the magic.

You’re asking:

How does SIMD actually achieve this?
(aligned memory, contiguous data, no branching, homogeneous types)

Important clarification:

👉 SIMD itself does not “achieve” these conditions.
👉 The data layout must enable them.
👉 SIMD instructions then exploit them.

Let’s unpack this properly.

1️⃣ What SIMD Actually Is

SIMD = Single Instruction, Multiple Data

Instead of:

add(a1, b1)
add(a2, b2)
add(a3, b3)
add(a4, b4)
CPU does:

ADD [a1,a2,a3,a4] , [b1,b2,b3,b4]
in one instruction.

Modern CPUs have:

128-bit registers (SSE)

256-bit registers (AVX2)

512-bit registers (AVX-512)

Example with AVX2:

256-bit register
If each value is 32-bit int → 8 integers processed in one instruction.

2️⃣ Why Memory Alignment Matters

CPU fetches memory in chunks (cache lines usually 64 bytes).

SIMD loads expect data aligned to boundaries:

16-byte

32-byte

64-byte

If aligned:

|----------------64 bytes----------------|
[100][200][300][400][500][600][700][800]
CPU can load entire block efficiently.

If unaligned:

...|100][200][300][400][500][600]...
Then CPU must:

Do multiple memory reads

Shift bits

Merge registers

That slows it down.

Columnar storage naturally stores values in flat arrays → easy to align.

Row storage mixes types → alignment breaks frequently.

3️⃣ Why Contiguous Data Matters

SIMD works on vectors like:

[100, 200, 300, 400]
If memory is contiguous:

address X → 100
X+4 → 200
X+8 → 300
X+12 → 400
CPU can do:

vmovdqa ymm0, [X]
One instruction loads 4–8 values.

If data is scattered:

Row layout:

[ id ][ country ptr ][ amount ]
To vectorize amount:

Must skip over id + pointer

Gather values

That requires "gather instructions".

Gather exists, but:

Much slower

Multiple memory accesses

Poor cache locality

Columnar avoids gather completely.

4️⃣ No Type Heterogeneity

SIMD register expects:

int32 OR float32 OR int64
Not:

int, string pointer, int, boolean
Row layout mixes types per row.

Column layout = single type per column.

This is perfect for:

vcmpps  (compare packed floats)
vpaddd  (add packed integers)
Homogeneous data = simple vector instruction.

5️⃣ No Branching (Very Important)

Branching = if/else

CPU branch prediction can fail.

Branch misprediction = 10–20 cycle penalty.

Example scalar code:

for each value:
   if value > 100:
       count++
That’s branching.

Vectorized version:

compare all 8 values at once
generate bitmask
count bits
No branching.
Just arithmetic + bit operations.

Columnar storage enables this because:

Data is in flat arrays

No per-row decoding logic

No jumping around

6️⃣ How SIMD Physically Executes

Let’s simulate:

Query:

WHERE amount > 100
Columnar memory:

[50, 120, 200, 90, 300, 80, 110, 40]
CPU loads:

ymm0 = [50,120,200,90,300,80,110,40]
ymm1 = [100,100,100,100,100,100,100,100]
Instruction:

vpcmpgtd ymm2, ymm0, ymm1
Result mask:

[0,1,1,0,1,0,1,0]
Then:

popcnt(mask)
Count matching rows.

All in a few instructions.

No loop.
No per-row branching.
No decoding.

That’s the power.

7️⃣ Why Row Stores Struggle

Row layout example:

[1, "India", 50]
[2, "US", 120]
[3, "UK", 200]
To evaluate amount > 100:

CPU must:

Load row

Parse row structure

Jump to offset

Extract amount

Repeat

Store amounts in temp array

Then vectorize

Extra work destroys SIMD benefit.

8️⃣ Memory Bandwidth Is The Real Bottleneck

Modern CPUs:

Extremely fast arithmetic

Limited memory bandwidth

SIMD helps because:

Do more work per byte loaded

Columnar helps because:

Load fewer bytes

Use each byte efficiently

Row layout wastes memory bandwidth.

9️⃣ Important Insight

SIMD is not a database feature.

It is a CPU hardware feature.

Columnar databases are designed to:

Align with CPU vector units

Align with cache architecture

Minimize memory stalls

Row databases were designed decades earlier when:

SIMD was primitive

Memory hierarchies were simpler

Workloads were transactional

Columnar is hardware-aware database design.

🔟 Final Mental Model

Columnar storage:

Flat arrays
    ↓
Aligned memory
    ↓
Contiguous loads
    ↓
Homogeneous types
    ↓
No branching
    ↓
Wide SIMD execution
Row storage:

Mixed layout
    ↓
Pointer chasing
    ↓
Cache misses
    ↓
Branching
    ↓
Scalar execution dominates


