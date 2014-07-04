fatomic: helpers for atomic file writes
=======================================

All of the functions in this module work atomically—either they
succeed, or they raise an `OSError` (or pass through some other
exception) and leave the original file (if any) untouched.

The name, of course, is a racist joke about overweight Irish
people. Or maybe it's just sure for "file, atomic".

Common arguments
================

`filename`

For `write*` functions, the filename is the name of the file to
write. Any existing file will be overwritten on success. If there
is no existing file, there is no problem.

For `transform*` functions, the filename is the name of both the
input file and the output file; any existing data will be
replaced with the transformed data. If there is no existing file,
it's a `FileNotFoundError` (or `OSError` with `ENOENT`, for older
versions of Python), as usual for an input file.

For `append*` functions, the filename is again the name of both
the input file and the output file; any existing data will still
be present, and the data data will follow it. If there is no
existing file, there is no problem; a new file will be created
with just the new data.

`binary`

Most functions take a `binary=False` flag. This determines whether
the input and temporary files are read and written in binary mode
or text mode. This is particularly important in Python 3.x, where
text mode means Unicode `str` and binary means `bytes`.

Some functions specify `binary=None` instead. In that case, a
`None` value means `fatomic` will guess the file mode based on
the type of the data: text for `str`, binary for `bytes` or
`bytearray`, text for anything else. (Note that `bytes` and `str`
are the same type in 2.x.)

`func`

The `transform*` functions take a function that will be run on
the whole file, or each line, or each chunk. Whether that means
`str` or `bytes` depends on the `binary` flag. (Of course in 2.x
those are the same type.)

Example
=======

    with fatomic.open('foo', 'a') as f:
        f.write('Hello, ')
    with fatomic.open('foo', 'a') as f:
        f.write('world\n')
    fatomic.transform('foo', lambda line: line.replace('Hello', 'Hi'))

The first time you run that, you should get a new file named
"foo" with the single line "Hi, world". Running it repeatedly
should keep appending lines to it. Hitting ^C or pulling the
power cord or whatever may leave you with a trailing "Hello, "
or a trailing "Hello, world", but can never leave you with
any lost lines before that.

Functions
=========

`replace(src, dst)`

Like [`os.replace`][replace] in Python 3.3+, but works in earlier 
versions. (Of course it doesn't support `src_dir_fd` and `dst_dir_fd`.)

  [replace]: https://docs.python.org/3/library/os.html#os.replace

`open(filename, mode, *args, **kwargs)`

Returns a file-like object that writes to a temporary file, then
overwrites filename atomically when closed.

`filename` is as in the builtin `open`.

`mode` is as in the builtin `open`, but it may not be `r`, `r+`, 
or `w+`, because that would make no sense. In the case of `a`, the
file starts with a copy of the original file.

All other arguments are identical to the builtin `open`.

In addition to the usual file-like-object methods, there will be
a `discard` method that tells it _not_ to overwrite the original
file at exit.

`write(filename, lines, binary=False)`

Write all of `lines`, which can be an iterator, to `filename`.
Just like the `writelines` method on a file object, this does not
add newlines.

`writeall(filename, contents, binary=None)`

Write all of `contents` to `filename`.

`writechunks(filename, chunks, binary=False)`

Write all of `chunks`, which can be an iterator, to `filename`.

Note that this is identical to `writelines`, and is only provided
for API consistency.

`transform(filename, func, binary=False)`

Write `func(line)` for each `line` in the file (without reading 
the entire file into memory all at once).

`transformall(filename, func, binary=False)`

Write `func(contents)` for the entire contents of the file (which
obviously are read completely into memory).

`transformchunks(filename, func, chunksize=None, binary=False)`

Write `func(chunk)` for each `chunk` of up to `chunksize` length
(without reading the entire file into memory all at once). The
transformed chunks do not have to be the same length as the
original chunks.

If `chunksize` is `None`, a reasonable default will be used.

`append(filename, lines, binary=False)`

Append all of `lines`, which can be an iterator, to `filename`.
Just like the `writelines` method on a file object, this does not
add newlines.

`appendall(filename, contents, binary=None)`

Append all of `contents` to `filename`.

`appendchunks(filename, chunks, binary=False)`

Append all of `chunks`, which can be an iterator, to `filename`.

Note that this is identical to `appendlines`, and is only
provided for API consistency.

