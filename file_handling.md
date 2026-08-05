# FILE HANDLING
- opening a file: open(f_name, mode)
- 4 modes: read(r) , write(w), append(a) and create(x)
- read mode: just reading permission to a file [default mode]; creates a new file if the given filename doesn't exist
- write mode: just to write to a file but if the script is ran again, the previous data is overwritten; creates a new file if the given filename doesn't exist
- append mode: acts like write mode but allows the data to be saved and adds upon it
- create mode: creates a specified file and returns an error if the file exists
- we can also specify how the data is handled
- text mode(t): parses data in normal text mode [default mode]
- binary mode(b): parses data in binary; used for images
- Important Methods: open(), close(), readline(), read(), readlines(), write(), writeline(), writelines()
- open(): opens a file
- close(): closes a file and saves the content of a file
- readline(): reads a line from the start of the file
- readlines(): reads the lines of the file and returns the lines in the form of a list/array and each line is separated by '\n'
- read(): reads the content of a file
- write(): writes data to a file
- writeline(): writes a line of data to the file
- writelines(): writes multiple lines of data to the file; lines are given in a list and separated by '\n'

# OS lib usage
- important methods: remove(), path.exists(), rmdir()
- remove(): deletes a file by its relative or absolute path
- path.exists(): checks if a file exists or a directory exists
- rmdir(): removes a directory
