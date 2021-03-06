<div itemscope itemtype="http://developers.google.com/ReferenceObject">
<meta itemprop="name" content="tfp.optimizer.linesearch.hager_zhang" />
<meta itemprop="path" content="Stable" />
</div>

# tfp.optimizer.linesearch.hager_zhang

``` python
tfp.optimizer.linesearch.hager_zhang(
    value_and_gradients_function,
    initial_step_size=None,
    objective_at_zero=None,
    grad_objective_at_zero=None,
    objective_at_initial_step_size=None,
    grad_objective_at_initial_step_size=None,
    threshold_use_approximate_wolfe_condition=1e-06,
    shrinkage_param=0.66,
    expansion_param=5.0,
    sufficient_decrease_param=0.1,
    curvature_param=0.9,
    step_size_shrink_param=0.1,
    max_iterations=50,
    name=None
)
```

The Hager Zhang line search algorithm.

Performs an inexact line search based on the algorithm of
[Hager and Zhang (2006)][2].
The univariate objective function `value_and_gradients_function` is typically
generated by projecting
a multivariate objective function along a search direction. Suppose the
multivariate function to be minimized is `g(x1,x2, .. xn)`. Let
(d1, d2, ..., dn) be the direction along which we wish to perform a line
search. Then the projected univariate function to be used for line search is

```None
  f(a) = g(x1 + d1 * a, x2 + d2 * a, ..., xn + dn * a)
```

The directional derivative along (d1, d2, ..., dn) is needed for this
procedure. This also corresponds to the derivative of the projected function
`f(a)` with respect to `a`. Note that this derivative must be negative for
`a = 0` if the direction is a descent direction.

The usual stopping criteria for the line search is the satisfaction of the
(weak) Wolfe conditions. For details of the Wolfe conditions, see
ref. [3]. On a finite precision machine, the exact Wolfe conditions can
be difficult to satisfy when one is very close to the minimum and as argued
by [Hager and Zhang (2005)][1], one can only expect the minimum to be
determined within square root of machine precision. To improve the situation,
they propose to replace the Wolfe conditions with an approximate version
depending on the derivative of the function which is applied only when one
is very close to the minimum. The following algorithm implements this
enhanced scheme.

### Usage:

Primary use of line search methods is as an internal component of a class of
optimization algorithms (called line search based methods as opposed to
trust region methods). Hence, the end user will typically not want to access
line search directly. In particular, inexact line search should not be
confused with a univariate minimization method. The stopping criteria of line
search is the satisfaction of Wolfe conditions and not the discovery of the
minimum of the function.

With this caveat in mind, the following example illustrates the standalone
usage of the line search.

```python
  # Define a quadratic target with minimum at 1.3.
  value_and_gradients_function = lambda x: ((x - 1.3) ** 2, 2 * (x-1.3))
  # Set initial step size.
  step_size = tf.constant(0.1)
  ls_result = tfp.optimizer.linesearch.hager_zhang(
      value_and_gradients_function, initial_step_size=step_size)
  # Evaluate the results.
  with tf.Session() as session:
    results = session.run(ls_result)
    # Ensure convergence.
    assert(results.converged)
    # If the line search converged, the left and the right ends of the
    # bracketing interval are identical.
    assert(results.left_pt == result.right_pt)
    # Print the number of evaluations and the final step size.
    print ("Final Step Size: %f, Evaluation: %d" % (results.left_pt,
                                                    results.func_evals))
```

### References:
[1]: William Hager, Hongchao Zhang. A new conjugate gradient method with
  guaranteed descent and an efficient line search. SIAM J. Optim., Vol 16. 1,
  pp. 170-172. 2005.
  https://www.math.lsu.edu/~hozhang/papers/cg_descent.pdf

[2]: William Hager, Hongchao Zhang. Algorithm 851: CG_DESCENT, a conjugate
  gradient method with guaranteed descent. ACM Transactions on Mathematical
  Software, Vol 32., 1, pp. 113-137. 2006.
  http://users.clas.ufl.edu/hager/papers/CG/cg_compare.pdf

[3]: Jorge Nocedal, Stephen Wright. Numerical Optimization. Springer Series in
  Operations Research. pp 33-36. 2006

#### Args:

* <b>`value_and_gradients_function`</b>: A Python callable that accepts a real scalar
    tensor and returns a tuple of scalar tensors of real dtype containing
    the value of the function and its derivative at that point.
    In usual optimization application, this function would be generated by
    projecting the multivariate objective function along some specific
    direction. The direction is determined by some other procedure but should
    be a descent direction (i.e. the derivative of the projected univariate
    function must be negative at 0.).
* <b>`initial_step_size`</b>: (Optional) Scalar positive `Tensor` of real dtype. The
    initial value to try to bracket the minimum. Default is `1.` as a float32.
    Note that this point need not necessarily bracket the minimum for the line
    search to work correctly but the supplied value must be greater than
    0. A good initial value will make the search converge faster.
* <b>`objective_at_zero`</b>: (Optional) Scalar `Tensor` of real dtype. If supplied,
    the value of the function at `0.`. If not supplied, it will be computed.
* <b>`grad_objective_at_zero`</b>: (Optional) Scalar `Tensor` of real dtype. If
    supplied, the derivative of the  function at `0.`. If not supplied, it
    will be computed.
* <b>`objective_at_initial_step_size`</b>: (Optional) Scalar `Tensor` of real dtype.
    If supplied, the value of the function at `initial_step_size`.
    If not supplied, it will be computed.
* <b>`grad_objective_at_initial_step_size`</b>: (Optional) Scalar `Tensor` of real
    dtype. If supplied, the derivative of the  function at
    `initial_step_size`. If not supplied, it will be computed.
* <b>`threshold_use_approximate_wolfe_condition`</b>: Scalar positive `Tensor`
    of real dtype. Corresponds to the parameter 'epsilon' in
    [Hager and Zhang (2006)][2]. Used to estimate the
    threshold at which the line search switches to approximate Wolfe
    conditions.
* <b>`shrinkage_param`</b>: Scalar positive Tensor of real dtype. Must be less than
    `1.`. Corresponds to the parameter `gamma` in
    [Hager and Zhang (2006)][2].
    If the secant**2 step does not shrink the bracketing interval by this
    proportion, a bisection step is performed to reduce the interval width.
* <b>`expansion_param`</b>: Scalar positive `Tensor` of real dtype. Must be greater
    than `1.`. Used to expand the initial interval in case it does not bracket
    a minimum. Corresponds to `rho` in [Hager and Zhang (2006)][2].
* <b>`sufficient_decrease_param`</b>: Positive scalar `Tensor` of real dtype.
    Bounded above by the curvature param. Corresponds to `delta` in the
    terminology of [Hager and Zhang (2006)][2].
* <b>`curvature_param`</b>: Positive scalar `Tensor` of real dtype. Bounded above
    by `1.`. Corresponds to 'sigma' in the terminology of
    [Hager and Zhang (2006)][2].
* <b>`step_size_shrink_param`</b>: Positive scalar `Tensor` of real dtype. Bounded
    above by `1`. If the supplied step size is too big (i.e. either the
    objective value or the gradient at that point is infinite), this factor
    is used to shrink the step size until it is finite.
* <b>`max_iterations`</b>: Positive scalar `Tensor` of integral dtype or None. The
    maximum number of iterations to perform in the line search. The number of
    iterations used to bracket the minimum are also counted against this
    parameter.
* <b>`name`</b>: (Optional) Python str. The name prefixed to the ops created by this
    function. If not supplied, the default name 'hager_zhang' is used.


#### Returns:

* <b>`results`</b>: A namedtuple containing the following attributes.
* <b>`converged`</b>: Boolean scalar `Tensor`. Whether a point satisfying
      Wolfe/Approx wolfe was found.
* <b>`func_evals`</b>: Scalar int32 `Tensor`. Number of function evaluations made.
* <b>`left_pt`</b>: Scalar `Tensor` of same dtype as `initial_step_size`. The
      left end point of the final bracketing interval. If converged is True,
      it is equal to `right_pt`. Otherwise, it corresponds to the last
      interval computed.
* <b>`objective_at_left_pt`</b>: Scalar `Tensor` of same dtype as
      `objective_at_initial_step_size`. The function value at the left
      end point. If converged is True, it is equal to `objective_at_right_pt`.
      Otherwise, it corresponds to the last interval computed.
* <b>`grad_objective_at_left_pt`</b>: Scalar `Tensor` of same dtype as
      `grad_objective_at_initial_step_size`. The derivative of the function
      at the left end point. If converged is True,
      it is equal to `grad_objective_at_right_pt`. Otherwise it
      corresponds to the last interval computed.
* <b>`right_pt`</b>: Scalar `Tensor` of same dtype as `initial_step_size`.
      The right end point of the final bracketing interval.
      If converged is True, it is equal to 'step'. Otherwise,
      it corresponds to the last interval computed.
* <b>`objective_at_right_pt`</b>: Scalar `Tensor` of same dtype as
      `objective_at_initial_step_size`.
      The function value at the right end point. If converged is True, it
      is equal to fn_step. Otherwise, it corresponds to the last
      interval computed.
    grad_objective_at_right_pt'  Scalar `Tensor` of same dtype as
      `grad_objective_at_initial_step_size`.
      The derivative of the function at the right end point.
      If converged is True, it is equal to the dfn_step.
      Otherwise it corresponds to the last interval computed.