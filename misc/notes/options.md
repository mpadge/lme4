`lme4` options
======================

# Top-level

These parameters should be all be retained as explicit arguments to `[g]lmer`, because they're very basic and very
commonly used (and more generally, they affect *what* model
is fitted rather than *how* it is fitted).

* `formula`
* `data`
* `control` (? see below)
* `start` (?)
* `verbose`
* `subset`
* `weights`
* `na.action`
* `offset`
* `contrasts`
* `REML` (LMM only)
* `family` (GLMM only)
* `mustart` (GLMM only) (?)
* `etastart` (GLMM only) (?)
* `devFunOnly` (?)

The only **required** argument is `formula`, although `data` should usually be specified and `family` should usually be specified for GLMMs.

The ? arguments could possibly be relegated to the "control" level ...

# Control

There are two levels of control: (1) control arguments that get passed through to the optimizer and (2) control arguments that might be used within the `lme4` code.

At present `control` includes only category #1 (although a few of the control parameters (`rhobeg`, `rhoend`, `xst`, `verbose`) may get manipulated in `optwrap` before being passed through to the optimizer if the default optimizers, `bobyqa` or `Nelder_Mead`, are used).

By analogy with `lme`, we should arguably add an `lmerControl` argument and function, which takes arguments in category #2 and takes category #1 as additional `...` arguments ...

* `sparseX` (stub)
* `optimizer`
* `tolPwrss` (GLMM only)

# Checking control

In general for maximum flexibility we might want checking parameters to allow the options `{"stop", "warn", "ignore"}` (if we were being more flexible, which I wouldn't recommend, we could allow `FALSE` to be equivalent to `"ignore"`).  End users can always use `suppressWarnings` at the top level to get rid of warnings, although allowing suppression of individual warning gives much more flexibility (`suppressWarnings` can't *selectively* suppress warnings).

There are lots of choices for warnings: should this be nested within yet another level of control, or given as a separate list of controls?

* `chk.lev.gtr.1`: "stop"
* `chk.lev.gtr.obs`: "stop"
* `chk.lev.gtr.5`: "warn"
* `chk.rankZ.gtr.obs`: "stop" (should make this "ignore" for large matrices?
* `chk.rankX.gtr.p`: "stop"


### Passing control options among functions

It's a little complicated making sure all the control options get to the right place; when are they stored in `control`, when are they passed as individual arguments, etc.?

* as described above, all arguments other than the most basic ones are passed via `control` at the top level (i.e. into `[gn]lmer`)
* on return from `[gn]lmerControl`, optimization arguments are packed into `optControl` and check controls are packed into `checkControl`
* from there, most arguments are passed individually on to relevant functions
* optional arguments have to be stuffed into `rho`, the environment used for the devfun: `tolPwrss`, `compDev` -- it's a little bit tricky figuring out whether these are being extracted from `control` or whether they should be contained in the environment.  (A cleaner design would probably have them being passed explicitly throughout, as environments can be a bit of a grab bag ... for example, I got in a bit of trouble with `updateGlmerDevfun`, which doesn't pass `control` to `mkdevfun` but rather assumes that everything relevant is already contained in the environment ...)
* need to check `refit` carefully ...

## Starting options

* right now starting values are *supposed* to be set in mkLmerDevfun (although they
weren't because they weren't being passed through properly)