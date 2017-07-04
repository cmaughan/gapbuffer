If you arrived here by accident, and want to know what a gap buffer is:
https://en.wikipedia.org/wiki/Gap_buffer

This is my attempt at an STL-friendly gap buffer.  It is designed to be used in an editor, but since it is a templated container, you could store anything in it - not just chars.
Most of the standard STL operations are included.  From the outside, looking in, this buffer looks a lot like a std::vector.  The difference is that under the covers, a 'Gap' is maintained which makes insertion and removal of elements inexpensive.  Iterators behave as you expect them to, but they may effectively skip over the gap when you modify them.

An internal find_first_of, find_first_not_of is provided for a little extra speed.

Unit tests are provided to check on the basic behavior.  This project just builds a test executable which runs the tests.

