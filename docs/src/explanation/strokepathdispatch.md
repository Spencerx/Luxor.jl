# Custom behavior for strokepath and fillpath

You can customize the behvaiour of the functions that "paint" the current path to the drawing, such as `strokepath` , `strokepreserve` , `fillpath`, and `fillpreserve`.

Luxor provides a way to add some custom behaviour to the standard operation of these functions. 

Internally, these four functions are defined so that they can be connected to a different function using a `dispatcher` facility. For example, the Luxor function `fillpath()` is defined as:

```julia
fillpath() = fillpath(DISPATCHER[1])
fillpath(::DefaultLuxor) = Cairo.fill(_get_current_cr())
fillpath(::LDispatcher) = fillpath(DefaultLuxor())
```

`DISPATCHER[1]` is defined as an instance of a struct (with no fields) `DefaultLuxor`. 

The datatype `DefaultLuxor` is a subtype of `LDispatcher`.

`DISPATCHER` as such is defined as an array of `LDispatcher`. 
This is to make it mutable. Only the first element
i.e. `DISPATCHER[1]` is ever used.

To add some custom behavior for these functions, do the following:

1. Define a new struct `MyDispatcher <: Luxor.LDispatcher` (it needn't have any fields).

2. Define the function that dispatches on the above struct and 
does what you want it to do.

3. Finally, change `Luxor.DISPATCHER[1]` to an instance of your struct.

Here's an example that changes the behavior of all calls
to `strokepath()` such that the current color is printed to the terminal just before each path is stroked.

```julia
struct MyDispatcher <: Luxor.LDispatcher end

# define your own strokepath() function
function Luxor.strokepath(::MyDispatcher)
    # new behavior 
    println("$(Luxor.get_current_color())")
    # you can call the default behavior too
    return Luxor.strokepath(Luxor.DefaultLuxor())
end

Luxor.DISPATCHER[1] = MyDispatcher()

@draw begin
    for i in 1:20
        randomhue()
        star(rand(BoundingBox()), 30, 6, 0.5, action = :stroke)
    end
end
```

Now, all calls to `strokepath()` - whether explicitly called 
or through other functions (for example, with the `:stroke` action) -
will print the current color in the REPL just before the path is stroked.

Similar dispatches can be written for `strokepreserve`, `fillpath`,
`fillpreserve`, `clip`, `clippreserve`, and `paint`.

Functions which don't have methods defined for the types will default
to calling `funcname(Luxor.DefaultLuxor())`
