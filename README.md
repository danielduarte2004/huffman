# Huffman Coding/Compression

Developed my own Java implementation of file compression and decompression using Huffman Coding. I also developed my own test cases and used files of different sizes to test the code.

## Outline 

- [Project Introduction](#project-introduction)
- [Running the Code](#running-the-code)
- [Analysis](#analysis)
- [Appendix: How the Tree in `decompress` was generated](#appendix-how-the-tree-in-decompress-was-generated)

## Project Introduction

There are many techniques used to compress digital data (that is, to represent it using less memory). This assignment covers Huffman Coding, which is used everywhere from zipping a folder to jpeg and mp3 encodings. You can optionally read more about the history of Huffman Coding and this project in the expandable section below.

<details>
<summary>Optional Background of Huffman</summary>
Huffman coding was invented by David Huffman while he was a graduate student at MIT in 1950 when given the option of a term paper or a final exam. For details, see [this 1991 Scientific American Article][Huffman Article]. In an autobiography Huffman had this to say about the epiphany that led to his invention of the coding method that bears his name:

"*-- but a week before the end of the term I seemed to have nothing to show for weeks of effort. I knew I'd better get busy fast, do the standard final, and forget the research problem. I remember, after breakfast that morning, throwing my research notes in the wastebasket. And at that very moment, I had a sense of sudden release, so that I could see a simple pattern in what I had been doing, that I hadn't been able to see at all until then. The result is the thing for which I'm probably best known: the Huffman Coding Procedure. I've had many breakthroughs since then, but never again all at once, like that. It was very exciting.*"

[The Wikipedia reference][Huffman Wikipedia] is extensive as is [this online material][Duke Huffman ] developed as one of the original [Nifty Assignments][Nifty Assignments]. Both jpeg and mp3 encodings use Huffman Coding as part of their compression algorithms. In this assignment you'll implement a complete program to compress and uncompress data using Huffman coding.

We first gave a Huffman coding assignment at Duke in Spring of 1994. Over the years many people have worked on creating the infrastructure for the bit-reading and -writing code (as we changed from C to C++ to Java at Duke) and the GUI that drives the Huffman assignment. It was one of the original so-called "nifty assignments" (see http://nifty.stanford.edu) in 1999. In Fall of 2018 we moved away from the GUI and using a simple main and the command-line. This was done for pragmatic and philosophical reasons.
</details>

**You should read about the Huffman algorithm, and work to understand it. You won't be able to complete the project easily without the overview you'll get from this reading: [Huffman algorithm reading and examples](https://www.cs.duke.edu/csed/poop/huff/info/).** You should be able to answer the questions provided in the expandable section below based on your reading and class/discussion. **Try to do so before beginning to code (but you do not need to turn in your answers).**

<details>
<summary>Pre-reading self-assessment questions</summary>

1. Why are two passes over the input file to be compressed required when creating a compressed version of the input file?
2. What aspects of creating the Huffman tree from counts account for that process being a greedy algorithm?
3. At a high-level, how is the tree used to create 8-bit char/chunk encodings?
4. What is written first, after the magic number, in the compressed file?
5. Why are the bits written at the end of the compressed file representing PSEUDO_EOF required?
6. After reading the magic number and tree, how are the bits representing compressed data read when decompressing, e.g., how many bits are read each time the compressed data is accessed?

</details>

When you've read the description of the algorithm and data structures used you'll be ready to implement both decompression (a.k.a. uncompressing) and compression  using Huffman Coding. You'll be using input and output or I/O classes that read and write 1 to many bits at a time, i.e., a single zero or one to several zeros and ones. This will make debugging your program a challenge.


## Running the Code

**Run `HuffMainDecompress`**. This prompts for a file to decompress, then calls `HuffProcessor.decompress`. You're given a stub version of that method; it initially ***simply copies the first file to another file***, it doesn’t actually decompress it. To make sure you know how to use this program, we recommend you run the program as described.

Choose `mystery.tif.hf` from the data folder to decompress (the `.hf` suffix indicates this has been compressed by a working `HuffProcessor.compress`). When prompted with a name for the file to save, use a `UHF prefix`, i.e., save with the name `UFHmystery.tif.uhf (that suffix is the default).  

Then run `diff` on the command line/terminal (details in the expandable section below). Use diff to compare two files: the original, `mystery.tif.hf` and the uncompressed version: `UHFmystery.tif.uhf`. The `diff` program should say these files are the same. This is because the code you first get from git simply copies the first file to another file, it doesn't actually decompress it. **You will use `diff` to check whether your implementation is working correctly locally, there are no JUnit tests for this project.**

The main takeaways here in running before implementing `HuffProcess.decompress` are to 
- Understand what to run when decompressing.
- Understand how to use `diff` on the command line to compare files. 

<details>
<summary>Expand for details on running diff at the terminal</summary>

There is a mac/unix command `diff` you can run in a terminal/bash shell on Mac/Windows (including the built-in terminal in VS Code). This command-line `diff`  compares two files and indicates if they are the same bit-for-bit or not. First you must navigate to the directory/folder containing the files you would like to compare. You can use the following commands to navigate directories in your terminal:

- `pwd` stands for "print working directory." Entering it will print the path to your current directory (the folder you are in).
- `ls` stands for "list subdirectories." Entering it will print all files and subfolders in your current folder. 
- `cd` stands for "change direcotory. Entering it, followed by a subfolder name, will navigate to that subfolder. For example, `cd Documents`. Entering `cd` with nothing after will navigate to your home directory. Entering `cd ..` will navigate to the enclosing directory (that is, will "back up one level" in the folder hierarchy).

Once you are in the same folder as the files you would like to compare, you can type (terminal/shell): `diff foo.txt bar.txt` to compare `foo.txt` and `bar.txt`.

If the files are the same _nothing is printed_. If the files are different there's an indication of where they are different for text files, and just `different` if the files are binary/compressed/image files. For your purposes in P5 it isn't especially crucial that you understand the output printed by `diff` - generally you will just want to check if a decompressed file is exactly same as the original file before any compression/decompression.

</details>

## Analysis

You'll submit the analysis as a PDF separate from the code in Gradescope.

For the analysis questions, we will let $`N`$ be the number of total characters in a file to encode, and let $`M`$ be the total number of *unique* characters in the file. Note that both refer to the *non-compressed file*. Note that $`M \leq N`$. Define the *compression ratio* of a file to be the number of bits in the original file divided by the number of bits in the compressed file.

Note that running the `HuffMainCompress` and `HuffMainDecompress` programs will print information to the terminal about the number of bits and the runtime of the compress and decompress algorithms.

**Question 1.** Suppose you want to compress two different files: `fileA` and `fileB`. Both have $`N`$ total characters and $`M`$ unique characters. The characters in `fileA` follow a uniform distribution, meaning each of the unique characters appears $`N/M`$ times. In `fileB`, the $`i`$'th unique character appears $`2^i`$ times (and the numbers add up to $`N`$), so some characters are much more common than others. Which file should achieve a higher compression ratio? Explain your answer. 

**Question 2.** What is the asymptotic runtime complexity of `compress` as a function of $`N`$ and/or $`M`$? Explain your answer, referencing the algorithm / implementation. Be sure to account for all three parts of compression in your explanation: (1) determining counts of characters, (2) creating the Huffman coding tree, and (3) writing the encoded file.

**Question 3.** When running `decompress`, each character that is decompressed requires traversing at most $`M`$ nodes in the Huffman coding tree, and there are $`N`$ such characters. This analysis would at first suggest that the asymptotic runtime complexity of `decompress` is $`O(MN)`$. However, you are unlikely to experience this in practice; this estimate is too simple and pessimistic. To see why, answer the following examining two different extreme cases:
- First consider the case where, like `fileA` in question 1, every unique character appears $`N/M`$ times. Then what would the asymptotic runtime complexity of `decompress` be?
- Now consider the case like `fileB` where the $`i`$'th unique character appears $`2^i`$ times (and the numbers add up to $`N`$). Would the runtime complexity be better or worse than for `fileA`? You do not need to derive the asymptotic runtime complexity exactly, just compare to the answer for `fileA`. *Hint*: Recall your answer to question 1 and observe the relationship between the runtime complexity of `decompress` and the *number of bits* in the data being decompressed.

## Appendix: How the Tree in `decompress` was generated

<details>
<summary> Expand for appendix details </summary>

A 9-bit sequence represents a "character"/chunk stored in each leaf. This character/chunk, actually a value between 0-255, will be written to the output stream when uncompressing. One leaf stores PSEUDO_EOF, this won't be printed during uncompression, but will stop the process of reading bits.

<div align="center">
  <img width="291" height="213" src="p5-figures/huffheadtreeNODES.png">
</div>

When you read the first 0, you know it's an internal node (it's a 0), you'll create an internall `HuffNode` node, and recursively read the left and right subtrees. The left subtree call will read the bits 001X1Y01Z1W as the left subtree of the root and the right subtree recursive call will read  01A1B as the right subtree. Note that in the bit-sequence representing the tree, a single bit of 0 and 1 differentiates INTERNAL nodes from LEAF nodes, not the left/right branch taken in uncompressing -- that comes later. The internal node that's the root of the left subtree of the overall root has its own left subtree of 01X1Y and a right subtree of 01Z1W. When you read the single 1-bit, your code will need to read 9-bits to obtain the value stored in the leaf.

</details>
