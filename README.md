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

```bbj
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

```bbj
t! = new NextPrimeThread()
t!.setValue("FINDNEXTPRIME",1000)
t!.setCallback(NextPrimeThread.ON_THREAD_FINISHED,"finished")
t!.start()
```

When the run() method finishes, the ON_THREAD_FINISHED Event is thrown:

```bbj
finished:
    ev! = BBjAPI().getLastEvent() 
    t!= ev!.getObject()
    n = t!.getValue("FINDNEXTPRIME")
    r = t!.getValue("THENEXTPRIME")  
    PRINT "Thread to find next higher prime number for "+str(n)+" has just returned with result "+str(r)
```

## Exchanging Data between Parent and Child Thread

The Thread class implements a ```setValue()``` and ```getValue()``` which allows to put variables. The communication is implemented internally using a BBjNameSpace, so the according rules in terms of CustomObjects apply. 

## Events

There are two events: 

ON_THREAD_FINISHED is fired when the background thread has finished the ```run()``` method. All variables put with ```setValue()``` before the run() method's end can then be retrieved in the parent thread using ```getValue()```. The Event is a BBjCustomEvent and passes the Thread object as its payload.

ON_THREAD_UPDATE is fired whenever the child thread executes ```#update()```. Before that, it can have set variables that the parent thread can use e.g. to update the UI.

## Controlling the Child Thread

When the parent thread executes ```abort()``` the background thread can query that instruction by a call to the ```#shouldAbort()``` method. If this returns true, the thread may gracefully terminate.

The ```kill()``` method can be called in the parent to force a hard termination of the background thread. It may be applied with care as the thread is internally ended with Java's Thread::stop() which shall only be a last resort for background processes made of legacy code that is not prepared to terminate gracefully by checking the ```#shouldAbort()`` method frequently while in close loops.

## Demos

The demos in the Demo subfolder show how to use the plug-in in Detail. There are two samples:

### Demo.bbj

This is the full demo of the sample snippets above. You can start multiple threads that compute the next prime number in background. It shows the simplest use case that could be applied: start a thread and let it do something in background, and get notified when it's done

### UpdateUIDemo.bbj

This demo demonstrates how the background thread can notify the UI about where it stands, while it's running. The thread also shows how it can check if the UI did already abandon it by using the ```BBjThread::abort()``` method so it can terminate and stop wasting CPU cycles.





