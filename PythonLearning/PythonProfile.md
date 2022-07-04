# Python Profile

## cProfile

原理：  
    在enable的时候把profiler_callback通过PyEval_SetProfile注册到ThreadState->c_profilefunc中。  
    在call_function时都会调用这个profiler_callback。  
    在一个函数开始调用时，ptrace_enter_call，函数返回时，ptrace_leave_call  
    exception和C函数call的时候也会处理。  

    static int
    profiler_callback(PyObject *self, PyFrameObject *frame, int what,
                    PyObject *arg)
    {
        switch (what) {

        /* the 'frame' of a called function is about to start its execution */
        case PyTrace_CALL:
            ptrace_enter_call(self, (void *)frame->f_code,
                                    (PyObject *)frame->f_code);
            break;

        /* the 'frame' of a called function is about to finish
        (either normally or with an exception) */
        case PyTrace_RETURN:
            ptrace_leave_call(self, (void *)frame->f_code);
            break;

        /* case PyTrace_EXCEPTION:
            If the exception results in the function exiting, a
            PyTrace_RETURN event will be generated, so we don't need to
            handle it. */

        /* the Python function 'frame' is issuing a call to the built-in
        function 'arg' */
        case PyTrace_C_CALL:
            if ((((ProfilerObject *)self)->flags & POF_BUILTINS)
                && PyCFunction_Check(arg)) {
                ptrace_enter_call(self,
                                ((PyCFunctionObject *)arg)->m_ml,
                                arg);
            }
            break;

        /* the call to the built-in function 'arg' is returning into its
        caller 'frame' */
        case PyTrace_C_RETURN:              /* ...normally */
        case PyTrace_C_EXCEPTION:           /* ...with an exception set */
            if ((((ProfilerObject *)self)->flags & POF_BUILTINS)
                && PyCFunction_Check(arg)) {
                ptrace_leave_call(self,
                                ((PyCFunctionObject *)arg)->m_ml);
            }
            break;

        default:
            break;
        }
        return 0;
    }