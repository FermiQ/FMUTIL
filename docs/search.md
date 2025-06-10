# search.f90 (Module: Search)

## Overview

The `Search` module provides utility functions for searching within data structures. Currently, it offers a binary search function for finding elements in sorted arrays.

## Key Components

### Functions/Subroutines

*   `BinarySearch(SortedX, SortedDataArray)` (pure function): Performs a binary search for multiple elements within a sorted array.
    *   **Purpose:** Given an array of values (`SortedX`) to search for, and a target array (`SortedDataArray`) that is sorted in ascending order, this function returns the indices in `SortedDataArray` where the elements of `SortedX` would be located.
    *   **Parameters:**
        *   `SortedX` (real(WP), dimension(1:), intent(in)): An array of elements to be found. The implementation assumes that this array (`SortedX`) is also sorted in ascending order for efficient processing of multiple search values, as the search for each element `SortedX(ctr)` initializes its search range based on the result of `SortedX(ctr-1)`.
        *   `SortedDataArray` (real(WP), dimension(:), intent(in)): The array to be searched. This array **must** be sorted in ascending order for the binary search algorithm to work correctly.
    *   **Returns:** (integer, dimension(size(SortedX))): An array of integers of the same size as `SortedX`. Each element `BinarySearch(i)` contains the index `m` in `SortedDataArray` such that `SortedDataArray(m) <= SortedX(i) < SortedDataArray(m+1)`. If `SortedX(i)` is smaller than the first element of `SortedDataArray`, the index will typically be 0 (or 1, depending on loop specifics if `m` is adjusted before assignment, needs careful check of implementation details if `SortedX(i)` is out of lower bound). If `SortedX(i)` is equal to `SortedDataArray(m)`, it returns `m`. If `SortedX(i)` is larger than all elements, it will be the index of the last element. Essentially, it finds the position where `SortedX(i)` fits or is located.

### Abstract Interface `func`
*   The `Search` module includes an `abstract interface func` definition:
    ```fortran
    abstract interface
        pure function func(x) result(y)
            import :: WP
            implicit none
            real(WP), intent(in) :: x !< independent variable
            real(WP) :: y
        end function func
    end interface
    ```
    This interface appears to be unused by the `BinarySearch` function and might be a leftover from development or copied from another module (like `RootFinding`). It does not affect the functionality of `BinarySearch`.

### Types/Classes
*   Not applicable. This module does not define or export public types or classes.

## Important Variables/Constants

*   `MAX_ITERATIONS` (integer, parameter): A constant `MAX_ITERATIONS = 50` is defined in this module.
    Similar to the `func` interface, this constant does not seem to be used by the `BinarySearch` function, which is based on direct array indexing rather than an iterative refinement process that would need such a limit. This might also be a leftover from development.

## Usage Examples

### `BinarySearch` Example

```fortran
program Test_BinarySearch
  use Search
  use FMUTILBase, only: WP ! For WP kind parameter
  implicit none

  real(WP), dimension(10) :: data_array = [ 1.0_WP, 3.0_WP, 5.0_WP, 7.0_WP, 9.0_WP, &
                                           11.0_WP, 13.0_WP, 15.0_WP, 17.0_WP, 19.0_WP ]
  real(WP), dimension(4)  :: items_to_find = [ 3.0_WP, 8.0_WP, 19.0_WP, 0.5_WP ]
  ! Note: For optimal use as per implementation detail, items_to_find should also be sorted.
  ! Let's sort it:
  real(WP), dimension(4)  :: sorted_items_to_find = [ 0.5_WP, 3.0_WP, 8.0_WP, 19.0_WP ]

  integer, dimension(size(sorted_items_to_find)) :: indices
  integer :: i

  print *, "Data Array:", data_array
  print *, "Items to find (sorted):", sorted_items_to_find

  indices = BinarySearch(sorted_items_to_find, data_array)

  print *, "Found indices (positions):"
  do i = 1, size(sorted_items_to_find)
    print *, "Item ", sorted_items_to_find(i), " found at/near index ", indices(i)
    ! Interpretation: indices(i) is the index m such that data_array(m) <= items_to_find(i)
    ! If items_to_find(i) < data_array(1), index might be 1 (or 0 if result was adjusted, check actual output).
    ! If items_to_find(i) == data_array(indices(i)), it's an exact match.
    ! If data_array(indices(i)) < items_to_find(i), then items_to_find(i) would be inserted after data_array(indices(i)).
  end do

  ! Example for an unsorted items_to_find (less optimal for this specific implementation)
  ! but still works element by element if the internal search resets properly for each item.
  ! The current implementation's 'a = m' suggests it tries to optimize for sorted 'SortedX'.
  ! If SortedX is not sorted, the search for SortedX(ctr) starts from where SortedX(ctr-1) was found,
  ! which might not be efficient or correct if SortedX is not sorted.
  ! For safety and correctness with the current code, SortedX should be sorted.

end program Test_BinarySearch
```

## Dependencies and Interactions

*   **Internal Dependencies:**
    *   `FMUTILBase`: Uses the `WP` kind parameter for defining the precision of real numbers used in the search.
*   **External Libraries:**
    *   None. The `BinarySearch` function is implemented using standard Fortran array operations.
```
