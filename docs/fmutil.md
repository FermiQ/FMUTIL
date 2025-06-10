# fmutil.F90 (Module: FMUtil)

## Overview

The `FMUtil` module is the primary public interface for the FMUTIL library. It consolidates various utilities, making them accessible to the end-user through a single `use FMUtil` statement. As described in the library's main documentation, FMUTIL provides miscellaneous utilities such as data structures (List, Vector), root-finding algorithms for polynomials and nonlinear equations, and search algorithms. By using `FMUtil`, users gain access to all these functionalities without needing to individually `use` each specific submodule.

## Key Components

The `FMUtil` module itself does not define new procedures or types. Instead, it makes entities from other modules within the FMUTIL library publicly available. When you `use FMUtil`, you are primarily gaining access to the public entities of the following modules:

*   `FMUTILBase`: Provides fundamental base types, parameters, or utility procedures that might be used by other modules in the library. (The exact public entities would be detailed in `fmutilbase.md`).
*   `RootFinding`: Offers tools for finding roots of equations. This includes algorithms for polynomial roots (potentially using companion matrices, possibly with dependencies like Intel MKL) and roots of nonlinear equations (e.g., Brent's algorithm).
*   `Search`: Contains algorithms for searching within data structures, such as binary search for sorted arrays.
*   `Vectors`: Provides the `Vector` dynamic array data structure (similar to C++ STL Vector), which requires users to define a custom element type extending `VecElem`. (Detailed in `vector.md`).
*   `Lists`: Provides the `List` dynamic list data structure (similar to Python List), capable of storing items of different types (`class(*)`). (Detailed in `list.md`).

## Important Variables/Constants

The `fmutil.F90` file itself does not define any public variables or constants. Any such entities would be part of the modules it re-exports (e.g., `FMUTILBase` might define some).

## Usage Examples

Using `FMUtil` simplifies access to the library's data structures and algorithms. Here's how you might use it to work with a `List` or `Vector` (these examples are conceptual and draw from the detailed examples in `list.md` and `vector.md`):

```fortran
program Test_With_FMUtil
  use FMUtil ! This single 'use' statement provides access to List, Vector, VecElem, etc.

  ! Example using List (adapted from list.md usage)
  implicit none
  type(List) :: my_generic_list
  integer :: list_idx
  real :: r_val = 3.14

  list_idx = my_generic_list%PushBack(10)          ! Add an integer
  list_idx = my_generic_list%PushBack(r_val)       ! Add a real
  list_idx = my_generic_list%PushBack("hello")     ! Add a character string

  print *, "My generic list size:", my_generic_list%Size()

  ! Example using Vector (requires a custom element type definition)
  ! First, define your custom element type and its assignment operator
  ! (This would typically be in its own module, as shown in vector.md)
  module MyCustomElements
    use FMUtil ! VecElem is available via FMUtil
    implicit none
    private
    public :: MyDataElement, assign_my_data_element ! Only export what's needed

    type, extends(VecElem) :: MyDataElement
      integer :: id
      real :: value
    contains
      procedure :: AssignVecElem => assign_my_data_element
    end type MyDataElement
  contains
    subroutine assign_my_data_element(lhs, rhs)
      class(MyDataElement), intent(out) :: lhs
      class(VecElem), intent(in) :: rhs
      select type (rhs)
      class is (MyDataElement)
        lhs%id = rhs%id
        lhs%value = rhs%value
      end select
    end subroutine assign_my_data_element
  end module MyCustomElements

  ! Now use the custom element with Vector
  use MyCustomElements
  type(Vector) :: my_data_vector
  type(MyDataElement) :: data_item
  integer :: vec_idx

  ! Initialize vector (optional, if MoldElem is needed before first PushBack)
  call my_data_vector%Init(MoldElem=MyDataElement(0, 0.0))

  data_item%id = 1
  data_item%value = 10.5
  vec_idx = my_data_vector%PushBack(data_item)

  data_item%id = 2
  data_item%value = 20.8
  vec_idx = my_data_vector%PushBack(data_item)

  print *, "My data vector size:", my_data_vector%Size()

  ! ... further operations on my_generic_list and my_data_vector ...

end program Test_With_FMUtil
```
For more detailed examples of how to use specific components like `Vector` (including defining `AssignVecElem` for custom types) or `List`, please refer to their respective documentation files (`vector.md`, `list.md`) and the extensive examples in the `fmutil.F90` source file comments.

## Dependencies and Interactions

The `FMUtil` module acts as a central aggregation point or facade for the FMUTIL library. Its primary role is to provide a convenient, single point of access to the functionalities offered by other modules.

*   **Internal Dependencies:**
    *   `FMUTILBase`: Relies on it for base definitions.
    *   `RootFinding`: Relies on it to provide root-finding capabilities.
    *   `Search`: Relies on it to provide search algorithms.
    *   `Vectors`: Relies on it to provide the `Vector` data structure.
    *   `Lists`: Relies on it to provide the `List` data structure.
*   **External Libraries:**
    *   None directly by `FMUtil` itself, but sub-modules like `RootFinding` might have dependencies (e.g., Intel MKL for polynomial root-finding, as mentioned in `fmutil.F90` comments).
```
