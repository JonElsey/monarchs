Common errors and debugging
***************************************

Debugging
=========
When using the Python debugger, you need to ensure that you run without Numba and in serial mode.
I like to take advantage of the fact that the model setup script is a Python script and add in an extra variable
``debug_mode``, which when set to ``True`` sets via an ``if`` block  ``use_numba``, ``parallel``, ``use_mpi`` and
``reload_state`` to ``False``/``False``/``False``/``True`` respectively , so that I can speedily load in the
state immediately before the model crashes and run it in a debugger to diagnose the problem.

A feature to allow for selection of a single column (denoted by its `x` and `y` coordinates on the grid) is WIP.
This would allow for users to run only on the cells that crash out, so they can debug.

If an error happens during the lateral movement step, and you don't want to run the entire single-column phase in
serial so that you can debug the lateral bit that happens at the end, you can add the `dump_data_pre_lateral_movement`
flag and set it to `True` in `model_setup`. This will create a dump file directly after the single column physics
finishes, rather than at the end of the model day.

A possible workflow if you see a lateral movement error might be to:
-   restart the run in parallel with `dump_data_pre_lateral_movement = True`
-   wait for it to finish, then re-run in serial with a debugger loading in from this dump file, ensuring that
`single_column_toggle` is set to `False` so the model goes straight to the lateral bit you want to debug.



Common errors
=============

``PermissionError`` : typically appears when you have two model runs in progress at the same time that are using the
same netCDF file for the input/output steps. Ensure that a) you only have one MONARCHS process running on a particular
set of files and b) that if running multiple runs at the same time your filepaths are correct.

``ValueError`` : can happen if the initial conditions are ill-posed. Typically, this is because enough firn has melted
that the amount of melt occurring in a timestep is higher than the vertical resolution. While MONARCHS has ways of
handling this, if the firn depth is small then this can cause the column to melt very quickly.
It can also arise if the solid or liquid fraction in a cell goes above 1. A full fix to this is in progress - however
it can also happen again if the initial conditions are ill-posed.

``ImportError`` : One of the required modules is missing. If this is NumbaMinpack, then likely your system does not have
``CMake`` and a compatible Fortran compiler installed. You can still run the model however using ``use_numba = False``,
in which case this ``ImportError`` should disappear. If you are running with ``use_numba = False`` and still get
``ImportError`` on NumbaMinpack, please contact Jon Elsey (j.d.elsey@leeds.ac.uk).