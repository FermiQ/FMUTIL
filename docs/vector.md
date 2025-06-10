# vector.f90 (Module: Vectors)

## Overview

The `Vectors` module provides a dynamic vector data structure, analogous to the C++ STL `std::vector`. It is implemented using an array of "buckets," where each bucket is also an array. This structure allows for efficient element addition and access. To use the `Vector` type, users must define their own data type that extends the abstract `VecElem` type and provide an assignment subroutine for this custom type.

## Key Components

### Types/Classes

*   `VecElem` (abstract, public): This is an abstract base type that users must extend to define the type of elements stored in a `Vector`.
    *   **Deferred Procedure:**
        *   `AssignVecElem(lhs, rhs)`: Users *must* implement a subroutine with this interface for their custom element type. This subroutine is responsible for defining how elements of the custom type are assigned (copied). It is then associated with the generic `assignment(=)` operator for the custom type.

*   `InvalidVecElem` (public, extends `VecElem`): A concrete type that can be returned by vector operations in case of an error or exception (e.g., accessing an out-of-bounds index). It has an internal `Invalid` logical flag.

*   `Vector` (public): This is the main public type representing the dynamic vector. It manages a collection of elements derived from `VecElem`.
    *   **Public Component:**
        *   `status` (integer): Indicates the success (0) or failure (negative value) of the last operation.
    *   **Methods (Procedures):**
        *   `Init(NewBktsToAllocate, BktCap, MoldElem)`: Initializes the vector. Can set the number of new buckets to allocate on expansion, the capacity of each bucket, and a mold element for allocation. If not called, the vector initializes with defaults on first data push.
        *   `Size()`: Returns the total number of elements currently stored in the vector.
        *   `Capacity()`: Returns the total number of elements the vector can store before needing to allocate more memory.
        *   `NUsedBkts()`: Returns the number of internal data buckets currently in use.
        *   `Reserve(n)`: Increases the vector's capacity to be able to hold at least `n` elements.
        *   `ShrinkToFit()`: Deallocates unused buckets to reduce memory footprint, making capacity closer to the current size.
        *   `PushBack(velem)`: Adds an element (`velem` of a type extending `VecElem`) to the end of the vector. Returns the 1-based index of the new element, or 0 on error.
        *   `PopBack()`: Removes and returns the last element from the vector.
        *   `ElemAt(Index)`: Returns a pointer to the element at the specified 1-based `Index`. The pointer is not associated if the index is invalid or the element slot is marked as free.
        *   `Front()`: Returns a copy of the first element in the vector.
        *   `Back()`: Returns a copy of the last element in the vector.
        *   `Slice(lower, upper, stride)`: Returns a new array containing a copy of vector elements from `lower` to `upper` (1-based indices) with a given `stride`. This is less efficient due to copying.
        *   `BktSlice(lower, upper)`: Returns a pointer to a contiguous array slice of elements within a single bucket. `upper` might be adjusted if the requested slice crosses bucket boundaries. More efficient as it avoids copying.
        *   `Insert(pos, NewElems, count)`: Inserts one or more `NewElems` (repeated `count` times) at the 1-based `pos`. Existing elements are shifted. This can be inefficient.
        *   `Erase(lower, upper)`: Deletes elements from `lower` to `upper` (inclusive). Existing elements are shifted. This can be inefficient.
        *   `Clear()`: Removes all elements from the vector. Bucket memory is not deallocated immediately but can be by `ShrinkToFit`.
        *   `Destroy()`: Finalizer procedure that frees all memory allocated by the vector and its buckets. Called automatically when a `Vector` variable goes out of scope.
        *   `assignment(=)`: Overloads the assignment operator to allow copying of one `Vector` to another, creating a deep copy of elements and internal state.

*   `Bkt` (private): Internal type representing a single bucket, which holds an array of `VecElem` (`BktData`) and flags for free slots (`SlotFree`).
*   `BktPtr` (private): A type holding a pointer to a `Bkt`. `VecData` in `Vector` is an array of `BktPtr`.

## Important Variables/Constants

*   `DEFAULT_BKTCAP` (integer, parameter): Default capacity (3) for internal buckets when a `Vector` is initialized without a specific `BktCap`.
*   `DEFAULT_NEWBKTSALLOCATED` (integer, parameter): Default number of new buckets (2) to allocate when the vector's capacity is exceeded and needs to grow, if not specified via `Init`.

## Usage Examples

To use the `Vector` type, you first need to define your own data type that extends `VecElem` and provide an assignment procedure for it.

```fortran
module MyVectorElements
  use Vectors ! Import the Vectors module to access VecElem
  implicit none

  ! 1. Define your custom element type
  type, extends(VecElem) :: MyIntElement
    integer :: value
  contains
    procedure :: AssignVecElem => MyIntElement_Assign
    ! You can add other type-bound procedures specific to MyIntElement here
  end type MyIntElement

  ! Interface for the assignment procedure (already defined in Vectors module)
  ! abstract interface
  !   subroutine assign_vecelem(lhs, rhs)
  !     import :: VecElem
  !     class(VecElem), intent(out) :: lhs
  !     class(VecElem), intent(in)  :: rhs
  !   end subroutine
  ! end interface

contains

  ! 2. Implement the assignment subroutine for your custom type
  subroutine MyIntElement_Assign(lhs, rhs)
    class(MyIntElement), intent(out) :: lhs
    class(VecElem), intent(in) :: rhs ! RHS must be class(VecElem) as per interface

    select type (rhs)
    class is (MyIntElement) ! Check if RHS is actually of MyIntElement type
      lhs%value = rhs%value
    type is (InvalidVecElem)
      ! Handle assignment from InvalidVecElem if necessary,
      ! or signal an error. For simplicity, we might just not assign.
      print *, "Warning: Attempting to assign from InvalidVecElem"
    class default
      ! This case should ideally not happen if only MyIntElement instances
      ! are added to a Vector of MyIntElement.
      error stop "MyIntElement_Assign: Type mismatch."
    end select
  end subroutine MyIntElement_Assign

end module MyVectorElements

program Test_MyVector
  use Vectors
  use MyVectorElements ! Import your custom element module
  implicit none

  type(Vector) :: vec
  type(MyIntElement) :: my_elem
  class(VecElem), pointer :: p_elem
  integer :: i, sz, cap, new_idx

  ! Optional: Initialize the vector with custom parameters
  ! call vec%Init(MoldElem=MyIntElement(0)) ! Provide a mold for allocation

  ! 3. Add elements to the vector
  my_elem%value = 10
  new_idx = vec%PushBack(my_elem)
  print *, "Pushed ", my_elem%value, " at index ", new_idx

  my_elem%value = 20
  new_idx = vec%PushBack(my_elem)
  print *, "Pushed ", my_elem%value, " at index ", new_idx

  my_elem%value = 30
  new_idx = vec%PushBack(my_elem)
  print *, "Pushed ", my_elem%value, " at index ", new_idx


  ! Get size and capacity
  sz = vec%Size()
  cap = vec%Capacity()
  print *, "Vector size:", sz       ! Output: Vector size: 3
  print *, "Vector capacity:", cap  ! Output depends on DEFAULT_BKTCAP & DEFAULT_NEWBKTSALLOCATED

  ! 4. Retrieve an element
  p_elem => vec%ElemAt(2) ! Get pointer to element at index 2
  if (associated(p_elem)) then
    select type (p_elem)
    class is (MyIntElement)
      print *, "Element at index 2:", p_elem%value ! Output: Element at index 2: 20
    end select
  else
    print *, "Element at index 2 not found or invalid."
  end if

  ! Pop an element
  call vec%PopBack()
  print *, "Vector size after PopBack:", vec%Size() ! Output: Vector size after PopBack: 2

  ! Clear the vector
  call vec%Clear()
  print *, "Vector size after Clear:", vec%Size()   ! Output: Vector size after Clear: 0

  ! Vector is automatically destroyed when 'vec' goes out of scope
end program Test_MyVector
```

## Dependencies and Interactions

*   **Internal Dependencies:**
    *   `FMUTILBase`: The `Vectors` module uses `FMUTILBase`. The specific reason is not explicitly detailed in the comments of `vector.f90`, but `FMUTILBase` likely provides fundamental utilities or type definitions used by the `Vectors` module.
*   **External Libraries:**
    *   None explicitly mentioned or used in `vector.f90`.
```
