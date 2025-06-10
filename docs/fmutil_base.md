# fmutil_base.F90 (Module: FMUTILBase)

## Overview

The `FMUTILBase` module serves as a foundational component for the FMUTIL library. It provides basic objects, numerical precision parameters, and commonly used constants that are utilized by other internal modules within FMUTIL. Its main purpose is to establish a consistent set of definitions for floating-point precision and mathematical constants.

## Key Components

The `FMUTILBase` module primarily exports parameters (constants). It does not define any public procedures (functions/subroutines) or derived types/classes.

### Functions/Subroutines

*   Not applicable. This module does not export public procedures.

### Types/Classes

*   Not applicable. This module does not export public types or classes.

## Important Variables/Constants

This module defines several key public parameters for numerical precision and mathematical constants:

*   `DP` (integer, parameter): Kind parameter for double precision floating-point numbers, typically corresponding to `real64` from `iso_fortran_env`.
*   `QP` (integer, parameter): Kind parameter for quadruple precision floating-point numbers, typically corresponding to `real128` from `iso_fortran_env`. Useful for calculations requiring very high precision.
*   `WP` (integer, parameter): Default "Word Precision" kind parameter used throughout the FMUTIL library for real numbers. By default, it is set to `DP` (double precision).
*   `EPS` (real(WP), parameter): The smallest positive real number of kind `WP` such that `1.0_WP + EPS > 1.0_WP`. This is effectively the machine epsilon for the default working precision.
*   `INF` (real(WP), parameter): Represents infinity for real numbers of kind `WP`. It is defined as `huge(1.0_WP)`.
*   `I3` (real(WP), dimension(3,3), parameter): A 3x3 identity matrix with elements of kind `WP`.
*   `I6` (real(WP), dimension(6,6), parameter): A 6x6 identity matrix with elements of kind `WP`.

## Usage Examples

Here's how another module or a user's program might utilize the constants defined in `FMUTILBase`:

```fortran
module MyCalculationModule
  ! Use specific constants from FMUTILBase
  use FMUTILBase, only: WP, INF, EPS, DP
  implicit none

  real(WP) :: my_variable
  real(DP) :: another_double_precision_var
  logical :: is_too_small

  public :: perform_calculation

contains

  subroutine perform_calculation(x, y)
    real(WP), intent(in) :: x
    real(WP), intent(out) :: y

    if (abs(x) < EPS) then
      is_too_small = .true.
      y = INF ! Assign infinity if x is too small
    else
      is_too_small = .false.
      y = 1.0_WP / x
    end if

    print *, "Is x too small? ", is_too_small
    print *, "Calculated y: ", y

    ! Using a different precision
    another_double_precision_var = dble(x) ! Convert from WP if WP is not DP
    print *, "x in double precision: ", another_double_precision_var

  end subroutine perform_calculation

end module MyCalculationModule

program Test_FMUTILBase_Usage
  use MyCalculationModule
  ! FMUTILBase constants are used within MyCalculationModule
  implicit none

  call perform_calculation(1.0e-8_WP, 기본값_결과_변수)  ! WP will be taken from FMUTILBase via MyCalculationModule
  call perform_calculation(1.0e-40_WP, 기본값_결과_변수_작음) ! Assuming WP is DP, this might be smaller than EPS

end program Test_FMUTILBase_Usage
```

## Dependencies and Interactions

*   **Internal Dependencies:**
    *   None directly, as this is a base module. However, many other modules within the FMUTIL library (e.g., `Vectors`, `Lists`, `RootFinding`) depend on `FMUTILBase` for consistent precision kinds and fundamental constants.
*   **External Libraries:**
    *   `iso_fortran_env` (intrinsic module): Used to obtain standard kind parameters for `real64` (double precision) and `real128` (quadruple precision), which are then used to define `DP` and `QP`.
```
