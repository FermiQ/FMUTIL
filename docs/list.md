# list.f90 (Module: Lists)

## Overview

The `Lists` module provides a dynamic list data structure, similar to Python lists. It allows for flexible collections of items (Fortran `class(*)`) where the size can change during runtime. The list is internally represented using a dynamically allocated array that grows as needed.

## Key Components

### Types/Classes

*   `List`: This is the main public type representing the dynamic list. It encapsulates the data and provides methods to manipulate the list.
    *   **Methods:**
        *   `PushBack(newitem)`: Adds an item to the end of the list. Returns the index of the new item, or 0 on error.
        *   `PopBack()`: Removes and returns the last item from the list.
        *   `Item(Index)`: Returns a pointer to the item at the specified 1-based `Index`. The pointer is not associated if the index is invalid.
        *   `Size()`: Returns the total number of items currently stored in the list.
        *   `Insert(pos, NewItem)`: Inserts `NewItem` at the given 1-based `pos`. Items at and after `pos` are shifted.
        *   `Erase(pos)`: Deletes the item at the given 1-based `pos`. Subsequent items are shifted.
        *   `Clear()`: Removes all items from the list. The underlying memory is not deallocated immediately but can be by `ShrinkToFit`.
        *   `ShrinkToFit()`: Deallocates unused memory so that the list's capacity matches its current size.
        *   `Destroy()`: Finalizer procedure that frees all memory allocated by the list and its items. Called automatically when a `List` variable goes out of scope.
        *   `assignment(=)`: Overloads the assignment operator to allow copying of one `List` to another, creating a deep copy of items.

*   `ListItem`: This is a private type used internally by `List` to store individual items and their validity status. It holds a `class(*), pointer :: item` to the actual data. Users of the `List` type do not interact with `ListItem` directly.

## Important Variables/Constants

*   `DEFAULT_NEWSLOTSALLOCATED` (integer, parameter): Defines the default number of additional storage slots (5) to allocate when the list's capacity is exceeded and needs to grow.

## Usage Examples

```fortran
program example_list_usage
  use Lists
  implicit none

  type(List) :: my_list
  integer :: item_value
  integer :: list_size_val
  class(*), pointer :: retrieved_item_ptr
  integer, pointer :: retrieved_value_ptr

  ! Add items to the list
  call my_list%PushBack(10)
  call my_list%PushBack(20)
  call my_list%PushBack(30)

  ! Get the size of the list
  list_size_val = my_list%Size()
  print *, "List size:", list_size_val ! Output: List size: 3

  ! Retrieve an item (e.g., the second item)
  retrieved_item_ptr => my_list%Item(2)
  if (associated(retrieved_item_ptr)) then
    select type (retrieved_item_ptr)
    class is (integer)
      retrieved_value_ptr => retrieved_item_ptr
      print *, "Retrieved item (index 2):", retrieved_value_ptr ! Output: Retrieved item (index 2): 20
    end select
  else
    print *, "Item not found or index out of bounds."
  end if

  ! Pop an item
  call my_list%PopBack() ! Removes 30
  print *, "List size after PopBack:", my_list%Size() ! Output: List size after PopBack: 2

  ! Clear the list (items destroyed, memory kept until ShrinkToFit or Destroy)
  call my_list%Clear()
  print *, "List size after Clear:", my_list%Size() ! Output: List size after Clear: 0

  ! Destroy is called automatically when my_list goes out of scope
end program example_list_usage
```

## Dependencies and Interactions

*   **Internal Dependencies:**
    *   `FMUTILBase`: The `Lists` module uses `FMUTILBase`. The specific reason is not explicitly detailed in the comments of `list.f90` but `FMUTILBase` likely provides fundamental utilities or type definitions used by the `Lists` module.
*   **External Libraries:**
    *   None explicitly mentioned or used in `list.f90`.
```
