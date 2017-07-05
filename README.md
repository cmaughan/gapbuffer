## Gap Buffer
If you arrived here by accident, and want to know what a gap buffer is:
https://en.wikipedia.org/wiki/Gap_buffer

This is my attempt at an STL-friendly gap buffer.  It helped me understand STL containers a little better.
This type of data structure is usually used to manipulate text efficiently, when edits occur in the same locality.
Since this implementation is a templated container, you could store anything in it - not just chars.
Most of the standard STL operations are included.  

From the outside this buffer looks like a std::vector.  The difference is that under the covers, a 'Gap' is maintained which makes insertion and removal of elements in the same
region inexpensive.  Iterators behave as you expect them to, but they are effectively skipping over the gap when you increment/decrement them - 
thus the container can be used like a contiguous vector, even though it is not.

An internal find_first_of, find_first_not_of is provided for a little extra speed - since it can make use of knowledge of the gap area
to split the search into 2 efficient loops.

Unit tests are provided to check on the basic behavior.  This project just builds a test executable which runs the tests.  Just include
gap_buffer.h in your project to use it.

## Example
```
GapBuffer<char> myBuffer(0, 4); // Empty buffer, with 4 character gap
myBuffer.push_back('A');
myBuffer.push_back('B');

assert(myBuffer[1] == B);

// Buffer now contains 'AB', and has a 2 character invisible Gap:
// We can use the string method to visualize the gap
assert(myBuffer.string(true) == "AB|2|");

// Remove the first character
myBuffer.erase(myBuffer.begin(), myBuffer.begin() + 1);

// The gap has moved to the beginning of the buffer, and is now 3 chars
assert(myBuffer.string(true) == "|3|B");

// Indexing the buffer or using iterators 'hides' the gap:
assert(myBuffer[0] == 'B');
assert(*myBuffer.begin() == 'B');
```
See the unit tests for more examples.

## Building
On windows there is a simple config.bat file for Visual Studio 2017.  On other systems, you should just be able to use CMake 
in the usual way.  Example of an out-of-source build setup:
```
mkdir build
cd build
cmake -G "Unix Makefiles" ..
cmake --build .
ctest -V
```

## To do
- There are a few missing container functions at the bottom of gap_buffer.h.  Fill the in and send me a PR ;)
- I haven't done any performance tests yet.
- EnsureGapPosAndSize could probably be more efficient if the gap is both moving and being resize.
- More unit tests

