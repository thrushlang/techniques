# Automatic Memory Managament

By default, the Thrush programming language manages memory automatically.

## What is its approach?

It doesn't use a garbage collector, a borrow checker to abuse the stack, and it doesn't use the heap for everything that exists.

The Thrush programming language takes a similar approach to Rust but is much simpler, without resorting to a borrow checker to abuse the stack.

## The technique of memory deallocation

Although this technique was named arbitrarily, its name is "**scope-bound heap deallocation**",  as its name indicates:

- Manages memory according to the returns and context of a function.
- Destructors are automatically created to deallocate any value found on the heap.
- Frees from memory any type that is stored recursively in a complex type.

#### Example:

```
struct RecursiveStructure {
     recursive RecursiveStructure; // Heaped structure by default.
};

fn main() {

    local recursive_structure_one: RecursiveStructure = new RecursiveStructure {
        recursive nullT // Barrier of heap deallocation.
    };    
    
    local recursive_structure_two: RecursiveStructure = new RecursiveStructure {
        recursive recursive_structure_one
    }; // Assigned by default to the stack, since the variable, which has a relative in the heap, is not used.   

    // The compiler:
    
    /*
        1. The compiler takes the structure " RecursiveStructure ".
        2. The compiler takes the variable " recursive_structure_one ".
        3. The compiler creates a map of the types contained in the structure.
        4. The compiler creates recursive deallocation functions if necessary (part of the destructors).
        5. The compiler calls or deallocates the contained types assigned within the structure.    
    */

    // End of function context (EFC reached)
    // All heaped pointers not used are freeded.  
 
}
```

## The technique of memory deallocation (Stage #2)

In the case of a return pointing to a local variable, depending on whether it is from the stack or the heap, this variable is excluded by the compiler's **deallocator**, and its value is extracted (if it is from the stack) or simply its pointer is passed, in the case of the heap.

```
struct Structure {
     a s64;
}; // LLVM IR: { i64 }

fn probe() Structure {
       
    local structure: Structure = new Structure {
        a 15092007
    };  

    // End of function context (EFC reached)   
    // All heaped pointers not used are freeded.  
    
    // ret { i64 } LLVM value;
    return structure;
}

fn main() {

   probe();

} 
```

In this case the compiler creates the return value as if it were a value, `{ i64 }` and returns it as a value. Instead of converting it to the heap.

## The technique of memory allocation

This technique has a fixed particularity in the compilation context. With certain conditionalities, the compiler decides whether to allocate a structure to the heap or the stack.

By default the types ALWAYS assigned to the stack are:

- `s8, s16, s32, s64` | `u8, u16, u32, u64`
- `f32, f64`
- `bool`
- `Array<[T; N]>` 

These types are always allocated on the stack, by default, and their value is "popped" to be stored in another pointer. 

>  `%xs = load s32, s32 %X -> store s32 %xs, ptr %xs2`.

Other existing types can switch between being allocated on the stack or the heap. Ultimately, the condition for any type to be allocated to the heap is that it exceeds the 128-byte threshold.

#### Example:

```
// Total structure size: 128 bytes
// Target Triple: x86_64-unknown-linux-gnu
// 64 bits integer size: 8 bytes.

struct HeapedStructure {
    a s64;
    b s64;
    c s64;
    d s64;
    f s64;
    g s64;
    h s64;
    i s64;
    j s64;
    k s64;
    l s64;
    m s64;
    n s64;
    ñ s64;
    o s64;
    p s64;
};    

fn main() {

    local heaped_strucuture: HeapedStructure = new HeapedStructure {
        ...
    }; // Heap allocated.

}
```
