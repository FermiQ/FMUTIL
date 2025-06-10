# rootfinding.f90 (Module: RootFinding)

## Overview

The `RootFinding` module provides numerical routines for finding roots of equations. It includes methods for computing all complex roots of a polynomial and for finding a single root of a general nonlinear function within a given interval.

## Key Components

### Abstract Interface `func`

*   **Purpose:** This abstract interface defines the signature for a user-supplied function whose root is to be found by the `Root` subroutine.
*   **Signature:**
    ```fortran
    abstract interface
        pure function func(x) result(y)
            import :: WP ! Uses the working precision from FMUTILBase
            implicit none
            real(WP), intent(in) :: x !< independent variable
            real(WP) :: y             !< function result f(x)
        end function func
    end interface
    ```
    A user must provide a function that matches this interface (i.e., takes a `real(WP)` scalar `x` and returns a `real(WP)` scalar `y`) to be used with the `Root` subroutine.

### Functions/Subroutines

*   `PolyRoots(c, r, error, BalanceOn)`: Computes all complex roots of a given polynomial.
    *   **Method:** Forms the companion matrix from the polynomial coefficients and then finds its eigenvalues using LAPACK routines (specifically `gebal` for balancing and `hseqr` for eigenvalue computation). This approach is similar to MATLAB's `roots` command.
    *   **Parameters:**
        *   `c` (complex(WP), dimension(1:), intent(in)): Coefficients of the polynomial `p(x) = c(1)*x^n + ... + c(n)*x + c(n+1)`. `c(1)` is the coefficient of the highest degree term.
        *   `r` (complex(WP), dimension(:), allocatable, intent(out)): Allocatable array that will contain the computed complex roots of the polynomial.
        *   `error` (integer, intent(out)): Error status.
            *   `0`: Success.
            *   `-1`: Input coefficients array `c` has length less than two (i.e., polynomial degree <= 0).
            *   `-2`: All coefficients in `c` are zero.
            *   `-3`: Companion matrix balancing (via `gebal`) failed.
            *   `-4`: LAPACK routine (`hseqr`) failed to compute eigenvalues.
        *   `BalanceOn` (logical, intent(in), optional): If `.TRUE.` (default), the companion matrix is balanced before eigenvalue computation. Set to `.FALSE.` if balancing is not desired, though this might affect accuracy for poorly-scaled matrices.

*   `Root(a, b, ya, yb, tol, f, x, fx, niter, error)`: Computes a single root of a user-defined function `f` using Brent's algorithm.
    *   **Method:** Implements Brent's method, which combines bisection, secant method, and inverse quadratic interpolation to find a root robustly and efficiently.
    *   **Parameters:**
        *   `a` (real(WP), intent(in)): Lower bound of the interval suspected to contain a root.
        *   `b` (real(WP), intent(in)): Upper bound of the interval suspected to contain a root.
        *   `ya` (real(WP), intent(in)): Value of the function at `a`, i.e., `f(a)`. (Note: The implementation recomputes `f(a)` and `f(b)` internally, so these might be considered for removal or as initial guesses).
        *   `yb` (real(WP), intent(in)): Value of the function at `b`, i.e., `f(b)`. (Similar to `ya`).
        *   `tol` (real(WP), intent(in)): Desired tolerance for the root `x`. The routine attempts to find `x` such that `abs(f(x))` is close to zero and the interval width is small.
        *   `f` (procedure(func)): The user-supplied function whose root is sought. Must conform to the `func` abstract interface.
        *   `x` (real(WP), intent(out)): The computed root.
        *   `fx` (real(WP), intent(out)): The value of the function `f` at the computed root `x`.
        *   `niter` (integer, intent(out)): Number of iterations performed by the algorithm.
        *   `error` (integer, intent(out)): Error status.
            *   `0`: Success, a root was found to the desired tolerance.
            *   `1`: The root is not bracketed by the initial interval [`a`, `b`] (i.e., `f(a)` and `f(b)` have the same sign).
            *   `2`: Maximum number of iterations (`MAX_ITERATIONS`) reached without achieving the desired tolerance.

### Types/Classes
*   Not applicable. This module does not define or export public types or classes.

## Important Variables/Constants

*   `MAX_ITERATIONS` (integer, parameter): Defines the maximum number of iterations allowed for the Brent's algorithm in the `Root` subroutine (value is 50). This prevents infinite loops in cases where convergence is too slow or fails.

## Usage Examples

### `PolyRoots` Example

```fortran
program Test_PolyRoots
  use RootFinding
  use FMUTILBase, only: WP ! For WP kind parameter
  implicit none

  ! Polynomial: x^3 - 6x^2 + 11x - 6 = 0
  ! Roots should be 1, 2, 3
  complex(WP), dimension(4) :: coeffs = [ (1.0_WP, 0.0_WP), & ! c(1) for x^3
                                          (-6.0_WP, 0.0_WP), & ! c(2) for x^2
                                          (11.0_WP, 0.0_WP), & ! c(3) for x^1
                                          (-6.0_WP, 0.0_WP) ]  ! c(4) for x^0
  complex(WP), dimension(:), allocatable :: roots_found
  integer :: err_status
  integer :: i

  call PolyRoots(coeffs, roots_found, err_status)

  if (err_status == 0) then
    print *, "Roots found for P(x) = x^3 - 6x^2 + 11x - 6:"
    do i = 1, size(roots_found)
      print '(F8.4, SP, F8.4, "i")', real(roots_found(i)), aimag(roots_found(i))
    end do
  else
    print *, "PolyRoots failed with error code: ", err_status
  end if

  if (allocated(roots_found)) deallocate(roots_found)

end program Test_PolyRoots
```

### `Root` Example (Brent's Algorithm)

```fortran
module MyFunctions
  use FMUTILBase, only: WP
  implicit none

  ! Define a function that matches the 'func' interface
  pure function my_user_function(x_val) result(y_val)
    real(WP), intent(in) :: x_val
    real(WP) :: y_val
    ! Example: f(x) = x^2 - 4
    y_val = x_val**2 - 4.0_WP
  end function my_user_function

end module MyFunctions

program Test_Root
  use RootFinding
  use FMUTILBase, only: WP
  use MyFunctions, only: my_user_function ! Import our function
  implicit none

  real(WP) :: lower_bound, upper_bound
  real(WP) :: val_at_lower, val_at_upper ! Not strictly needed as input as Root recomputes
  real(WP) :: tolerance
  real(WP) :: found_root, val_at_root
  integer  :: iterations_taken, err_status

  lower_bound = 0.0_WP
  upper_bound = 5.0_WP
  tolerance = 1.0e-6_WP

  ! ya and yb are technically inputs but Root recalculates f(a) and f(b)
  ! So, their input values here are not critical.
  val_at_lower = my_user_function(lower_bound)
  val_at_upper = my_user_function(upper_bound)

  print *, "Seeking root for f(x) = x^2 - 4 in [", lower_bound, ",", upper_bound, "]"
  call Root(lower_bound, upper_bound, val_at_lower, val_at_upper, &
            tolerance, my_user_function,                           &
            found_root, val_at_root, iterations_taken, err_status)

  if (err_status == 0) then
    print *, "Root found: ", found_root
    print *, "f(root): ", val_at_root
    print *, "Iterations: ", iterations_taken
  elseif (err_status == 1) then
    print *, "Root not bracketed in the given interval."
    print *, "f(a)=", my_user_function(lower_bound), " f(b)=", my_user_function(upper_bound)
  elseif (err_status == 2) then
    print *, "Maximum iterations reached. Current approximation: ", found_root
  else
    print *, "Root finding failed with unknown error code: ", err_status
  end if

end program Test_Root
```

## Dependencies and Interactions

*   **Internal Dependencies:**
    *   `FMUTILBase`: Uses the `WP` kind parameter for defining the precision of real and complex numbers.
*   **External Libraries:**
    *   `lapack95` (Intel MKL): The `PolyRoots` subroutine relies on LAPACK routines (`gebal`, `hseqr`) for eigenvalue computation of the companion matrix. Therefore, linking against a LAPACK implementation (like Intel MKL, as indicated by `use lapack95`) is necessary for `PolyRoots` to function. The `Root` subroutine (Brent's method) does not have this external dependency.
```
