# Who watches the watchdog

> How would you go about locating the cause of a stack overflow or watchdog timer reset?

## Stack Overflow

NOTE: I've heard "Stack Overflow" used to refer to a buffer overflow that corrupts the stack, and to the call stack overflowing the bounds of memory allocated to the stack.  My answer is tailored to the latter.

I would want to start with a code inspection.  Especially if the reproduction steps give me a clue of where to look.  I would look for recursive functions, large allocations of local memory, and unintentional recursive loops.  

After code inspection, I would go to experimentation.  This is only a good strategy if the overflow is reproducible in a timely manner.  Here are some of the things I would try:   

 - Can I revert to a previous version that doesn't do this? If so, what changes happened between now and then?  Is it possible to narrow down to a particular commit that introduced this bug?  
 - Can I disable individual modules and services? Maybe I can narrow down the overflow to a particular module or combination of modules.
 - What happens if I increase the amount of memory allocated to the stack to double or ten times?  If the overflow happens in the same timeframe, that points to a single event like runaway recursion.  If it doesn't happen at all, then maybe the max call size is just larger than was previously allowed for.

I would really want to get a debugger paused right after the overflow occurred.  I could write a canary to the end of the stack and check it periodically (right before the return of key functions) to see if it has been overwritten.  Then I could use a breakpoint, or write to debug, when I detect the overflow helping to narrow down when it occurs.  If the overflow triggers a fault, the MCU might allow me to create a fault handler and set a breakpoint in it.

## Watchdog Timer

When the watchdog timer resets the device, it often destroys any evidence of why it reset, so the first step is usually to disable the watchdog.  Some MCUs allow you to set up a watch dog handler that is called before or instead of resetting the device.  If so, that can be a good place for a breakpoint so you can investigate with a debugger.

With the watchdog completely disabled, I could try to print to debug or flash and LED every time the watch dog would be fed, then once it looks like the watchdog is no longer being fed, I would pause the debugger and see if I could see anything about the execution state.

This can help you identify an infinite loop really quickly.  If the problem is the system goes to sleep and isn't being woken up properly, the debugger sometimes doesn't know show useful information.

I would also consider printing to a terminal every time a routine was entered and exited.  Can I find an enter without and exit that might tell me where execution is getting stuck?  This strategy has the benefit of still working with the watchdog enabled since a terminal log can be preserved after the device resets.

Of course, the experiments I mentioned earlier like reverting to previous code, or narrowing down modules would also be worth considering here as well.  

In a multithreaded environment, I have set up a task as a watchdog supervisor.  Every individual task needs to "feed" the supervisor to check in.  The supervisor is responsible for feeding the hardware watchdog.  If any device goes too long without checking in, the supervisor creates a log of which task failed, and stops feeding the watchdog.  

## Conclusion

It's hard to answer these questions in the abstract because it depends a lot on what are the known symptoms, how was the bug first detected and reported, and what tools are available.  It matters a lot, for example, whether a reliable debugger is available.  Or a UART to terminal.

Debugging can seem like franticly trying unrelated things until you get some clue that helps you narrow it down.  Sometimes in boils down to "identify something you don't know, and learn that thing."  Because of that, it's important to keep notes of what has been tried and what the results where.  It helps decide whether an experiment is worth going back to an experiment you've already performed because there's a variation you didn't try, or the results where inclusive.
