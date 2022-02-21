# BBjThread
A class to assist with Multithreading for BBj

## Usage

* Implement your thread class by extending BBjThread
* implement the run() method in your extended class
* use ::start() to start a background thread
* use setValue and getValue to exchange objects and scalar variables
* use setCallback(BBjThread.ON_THREAD_FINISHED,..) to determine when the background process has finished

The following environment things are rebuilt in the background thread:
* commandline (except arguments list and terminal ID)
* STBL Strings

The following environment things are not rebuilt in the background thread
* OPEN file channels and SQL channels (if needed it can be done! Add an issue in case...)
* Terminal ID ("IO" is used)
* the ARGV lists (they're needed for this implementation)


## Example

This Thread calculates the next prime number for a given number:

```
use ::BBjThread/BBjThread.bbj::BBjThread

class public NextPrimeThread extends BBjThread

    method public NextPrimeThread()
    methodend
    
    method public void run()
        n = #getValue("FINDNEXTPRIME") +1
        while !#isPrim(n) 
            n=n+1
        wend
        #setValue("THENEXTPRIME",n)
    methodend
    
    method public static boolean isPrim(long value!) 
        if value! <= 2 then 
            methodret value! = 2
        endif
        for i=2 to int(sqr(value!))+1
            if mod (value!,i) = 0 then
                methodret Boolean.FALSE
            fi
        next
        methodret Boolean.TRUE
    methodend

classend
```

This is how you start the thread:

```
t! = new NextPrimeThread()
t!.setValue("FINDNEXTPRIME",1000)
t!.setCallback(NextPrimeThread.ON_THREAD_FINISHED,"finished")
t!.start()
```

When the run() method finishes, the ON_THREAD_FINISHED Event is thrown:

```
finished:
    ev! = BBjAPI().getLastEvent() 
    t!= ev!.getObject()
    n = t!.getValue("FINDNEXTPRIME")
    r = t!.getValue("THENEXTPRIME")  
    PRINT "Thread to find next higher prime number for "+str(n)+" has just returned with result "+str(r)
```

## Demos

The demos in the Demo subfolder show how to use the plug-in in Detail. There are two samples:

### Demo.bbj

This is the full demo of the sample snippets above. You can start multiple threads that compute the next prime number in background. It shows the simplest use case that could be applied: start a thread and let it do something in background, and get notified when it's done

### UpdateUIDemo.bbj

This demo demonstrates how the background thread can notify the UI about where it stands, while it's running. The thread also shows how it can check if the UI did already abandon it by using the ```BBjThread::abort()``` method so it can terminate and stop wasting CPU cycles.





