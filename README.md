# Logger


## Overview 

`Logger` is designed to write log files as ANSI (default) or UTF-8 files. 

You can create an instance without specifying any parameters at all, but you might specify up to 6 parameters.
Some but not all of these parameters can be changed later on as well.

By default an instance of the class creates a log file with the name "yyyymmdd.log". When a new day puts in an appearance, this file is closed and a new one is opened automatically.

Instead of accepting the defaults you can also let the class create "yyyymm.log" files or even "yyyy.log".

The main method, `Log`, does not do any kind of fancy formatting. It just accepts vectors of any kind as well as text matrices; performance is considered to be paramount. However, the method `LogError`  is different and does some formatting.

By default the class makes intense use of error trapping in order to make sure that neither `Log` nor `LogError` will ever affect the hosting application.

## Encoding 

By default "ANSI" is used as encoding option when opening/creating a log file. One reason is that this might well save some space. More importantly, many tools for processing log files are still not capable of dealing with UTF8 files.

"ASCII" was an option in earlier versions of `Logger`, when the Classic version of Dyalog was still supported. However, now it is deprecated. You may still specify it, but it will be converted to "ANSI" internally anyway.

For "UTF8" you may also specify "UTF-8" but it is converted into "UTF8".


## The methods 

### The Log method 

``` 
      {r}←Log msg 
```

`Log` writes `msg` to the Log File. `msg` can be one of:

* A character scalar (though this does not make too much sense)
* A character vector
* A character matrix
* A vector of character vectors

`r` gets the message written to the log file together with the time stamp and thread number.

For best performance this function does not do any formatting. If you need special formatting, consider writing a cover function for `Log` or your own class deriving from `Logger`.

### The LogError method 

This method is useful to log an error. It is a cover function of `Log`.

You can specify 2-3 parameters:
 1. A  single integer that is considered a return code. 0 means that `LogError` should not do anything at all.
 1. The message (msg). This can be any array containing text or numeric data as long as the depth and rank are both lower than `|3`.
 1. More information (more); this is optional. This can be any kind of array.

In case `rc ←→ 0`, `LogError` is doing nothing at all. Otherwise it writes `msg` into the log file and marks it up as an error.

If `rc ←→ 0` then `r` is empty, otherwise it returns the message written to the log file.

Examples:

```
    MyLoggerInstance.LogError 1 'The error message' ⎕DM
```


```
    MyLoggerInstance.LogError rc 'FATAL ERROR' ('hello word' (1 2 3))
```

While the `Log` method is fairly restrictive in order to avoid any performance penalties, the method `LogError` offers more freedom because this is hardly causing any harm: you won't have thousands of errors per second, and even if you have them, performance is the least of your worries then.

## Notes 

### Important defaults 
By default a file "{yyyymmdd}.log" is created within "path" or opened if it already  exists. When a new day comes along that file is closed and a new one is created. 

This default behaviour can be switched off by setting `autoReOpen` to 0.

### Error Trapping 
By default all possible errors - accept invalid calls - are trapped within the `Log` and `LogError` methods: a logging mechanism cannot be allowed to break an application. 

One exception: when creating an instance of `Logger` fails that causes a crash, but that means it was called with invalid parameters.

However, by setting `debug` and/or `printToSession` and/or `timestamp` the `Logger` class can be debugged.

### Preconditions 

Since version 5 `Logger` is expected to be consumed as a Tatin package, so you don't need to worry about dependencies.

## Sample session 

This code:

```
 myLogger←⎕NEW #.Logger
⍝ Exercise the "Log" method"
 myLogger.Log'this is my first entry!'
 myLogger.Log'Even' 'more' 'entries'
 myLogger.Log↑'A' 'text' 'matrix'
 myLogger.Log 1 2 3
 myLogger.Log('String')(⍳6)('Another string')
 myLogger.Log(1 2)(2 3⍴⍳6) ⍝ causes an error (trapped!)
 {myLogger.Log'Log entry written in a thread'}&⍬
⍝ Exercise the "LogError" method
 msg←'An error has occured'
 rc←0
 myLogger.LogError rc msg  ⍝ This has no effect: rc is 0
 rc←2
 myLogger.LogError rc msg
 more←'A fatal error has occured'(20 1009)((1 2)'FATAL'(2 3⍴⍳6))
 myLogger.LogError rc msg more ⍝ "more" can be any array
```

results in this log file:

```
2011-05-29 07:29:36 *** Log File opened                 
2011-05-29 07:29:36 this is my first entry!             
2011-05-29 07:29:36 Even                                
2011-05-29 07:29:36 more                                
2011-05-29 07:29:36 entries                             
2011-05-29 07:29:36 A                                   
2011-05-29 07:29:36 text                                
2011-05-29 07:29:36 matrix                              
2011-05-29 07:29:36 1 2 3                               
2011-05-29 07:29:36 String                              
2011-05-29 07:29:36 1 2 3 4 5 6                         
2011-05-29 07:29:36 Another string                      
2011-05-29 07:29:36 (2) Log entry written in a thread   
2011-05-29 07:29:36 *** ERROR RC=2; An error has occured
2011-05-29 07:29:36 *** ERROR RC=2; An error has occured
2011-05-29 07:29:36           A fatal error has occured 
2011-05-29 07:29:36           20 1009                   
2011-05-29 07:29:36            1 2  FATAL  1 2 3        
2011-05-29 07:29:36                        4 5 6        
```

Note that for the log entry written from its own thread, the thread number reported in the log file is non-zero.

## Constructors, fields, properties and methods 

```
      ]ADOC.List Logger -summary
*** Logger (Class) ***

Constructors:
  make0
  make1(pathOrCommandSpace)
  make2(path_ encoding_)
  make3(path_ encoding_ filenameType_)
  make4(path_ encoding_ filenameType_ debug_)
  make5(path_ encoding_ filenameType_ debug_ timestamp_)
  make6(path_ encoding_ filenameType_ debug_ timestamp_ refToUtils_)
Instance Properties:
  active
  autoReOpen
  debug
  encoding (ReadOnly)
  errorCounter (ReadOnly)
  errorPrefix
  extension
  fileFlag
  filenameDescriptor (ReadOnly)
  filenamePostfix
  filenamePrefix
  filenameType
  filename (ReadOnly)
  path (ReadOnly)
  printToSession
  refToUtils
  timestamp
Instance Methods:
  Close
  r ← FullFilename
  {r} ← LogError y
  {r} ← Log Msg
Shared Methods:
  r ← Copyright
  r ← CreateParms
  r ← History
  r ← Version

```
