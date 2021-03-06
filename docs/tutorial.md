# The Alloc class

The Alloc class is a class templated on the type of storage, and the rank of the array.
Declaring a 2D allocatable array of floats:

```c++
Alloc<float,2> a;  // currently not allocated

a.allocate(10,10); // now allocated to be a 10x10 array
a.deallocate();    // memory has been deallocated 
```

`Alloc` also provides a constructor which takes the size of the array, performing the 
two previous steps in one:

```c++
Alloc<float,2> a(10,10); // constructed and allocated
a.deallocate();          // memory deallocated
```
By default, the allocated memory is not initialized. 
The whole array can be set to a single value by assignment:

```c++
Alloc<float,2> a(10,10); 
a = 2.0;                 // all 100 elements of a now set to 2.0
```

Elements in the array can be accessed using the `()` and `[]` operators. 
`()` provides multi-dimensional access, while `[]` accesses elements in
a linear fashion. 

```c++
Alloc<int,3> a(10,10,10);

// This loop sets the elements of a in
// a linear fashion.
for(int i=0;i<a.size();i++){
  a[i] = i;
}

// This prints out `true`
cout <<  a(5,0,0) == a[6] << endl;
```

Contrary to Fortran, (which starts at 1 and goes until N), 
indexing starts at 0 and runs until N-1, for a more natural fit with C++.
However, in typical Fortran fashion, FortCpp defaults to using Column-Major
Ordering. Row-Major Ordering is fully supported, however, and can be specified
in the `Options` template parameter.

```c++
Alloc<bool,3,RowMajor> a(10,10,10);
```

## Views

Views of Alloc arrays can be created with the `view` member function.
`view` takes as its arguments a `Slice` and integer indices. 
A `Slice` specifies a range of a given dimension, while integer indices
specify a location in the given dimension.
A `Slice` is generated using the provided `Slice` function:

```c++
FullSlice S     = Slice();
ContigSlice cS  = Slice(2,7);
StridedSlice sS = Slice(0,8,2);
```

There are three different types of `Slices`: `FullSlice`, which
specifies that the entire dimension is to be used in a view.
`ContigSlice` specifies that the dimension from the starting index
(2 in the above example), up to, but not including the final index
(6 in the above example) is to be used in the view. A `StridedSlice`
adds a third argument, which is the step of the slice. In the above
example, `sS` specifies a slice starting at 0, and moving up to 7 
in steps of 2, or [0,2,4,6]. As a convenience, FortCpp provides
two special numbers, `BEG` and `END`. These numbers only have meaning
to a Slice, and refer to the beginning and end of a dimension, 
respectively.

In practice, it is easier to use the `auto` keyword to catch the result 
from the `Slice()` function. Slices can be used as:

```c++
Alloc<int,4> a(10,10,10,10);
auto S = Slice();

Alloc<int,2> b = a.view(S,S,4,6);
```

A View in this case is not a separate class, but generates another
`Alloc` object. This object does not own the data of the original array,
but is mapped to the specific region of the original data. Data of the 
original array can be read and modified as if it was the original array.

```c++
auto S = Slice(BEG,END,2); // view the even indices along a dimension
Alloc<int,1> a(100);
Alloc<int,1> b = a(S);

a = 1;
b = 0;

cout << a << endl; 
```

The above example will output an alternating sequence of 0's and 1's. 


### Debug

By default, FortC++ is in Debug mode. This includes bounds-checking and whole-array operation size checking.
If an error does occur, FortC++ throws an ArrayException. An ArrayException will print out an error message,
and then exit the program. 

To turn debugging off, define the macro NDEBUG. Use this for release code, as debugging does incur a performance hit.

### Compiling and Optimization

This library uses features from the C++11 standard, so you have to use a C++11 compliant compiler. 
With G++, this means compiling with the -`std=c++11` option. This library also relies heavily on the ability
of the compiler to inline everything, or else performance goes to zilch. For G++, this means compiling with the `-O3` flag.

