# Logger

A class that makes writing to a log file from an application written in Dyalog APL easy.

## Overview 

This class is designed to write log files as ANSI (default) or UTF-8 files. 

You can create an instance without specifying any parameters at all but you might specify up to 6 parameters.
Some but not all of these parameters can be changed later on as well.

By default an instance of the class creates a log file with the name "yyyymmdd.log". When a new day puts in an appearance this file is closed and a new one is opened automatically.

Instead of accepting the defaults you can also let the class create "yyyymm.log" files or even "yyyy.log".

Note that the main method, `Log`, does not do any kind of fancy formatting. It just accepts vectors of any kind as well as text matrices; performance is considered to be paramount. However, the method `LogError`  is different and does some formatting.

Note that by default the class makes intense use of error trapping to make sure that neither `Log` nor `LogError` will ever effect the hosting application.

## Encoding 

Note that for ANSI files with the Unicode version of Dyalog all non-ANSI characters are replaced by "?".

Generally the encoding it determined by the interpreter. You can make sure that only ANSI chars are written to the log file by specifying "ANSI". "ANSI" might save you a bit space but normally not much since most characters in a log file are ANSI anyway.

"ASCII" was an option in earlier version of `Logger`, when the Classic version of Dyalog was still supported. However, now it deprecated. You may still specify it, but it will be converted to "ANSI" internally anyway.


## The methods 

### The Log method 

``` 
      {r}←Log msg 
```

Writes `msg` to the Log File. Note that `msg` must be either a character scalar or a simple character vector, otherwise it is ignored. Note that `LogError` gives your more freedom in this respect - see there.

`r` gets the message written to the log file together with the time stamp and thread no.

`msg` can be one of:
 * A vector
 * A matrix
 * A vector of vectors

For best performance this function does not do any formatting. If you need a special formatting consider writing a cover function for `Log` or your own class deriving from `Logger`.

### The LogError method 

This method is useful to log an error. This is a cover function of `Log`.

You can specify 2-3 parameters:
 1. The return code. Single integer. 0 means that `LogError` should not do anything at all.
 1. The message (msg). This can be any array containig text or numeric data as long as the depth and rank are both lower than 3.
 1. More information (more); this is optional. This can be any kind of array.

In case `rc ←→ 0`, `LogError` is doing nothing at all. Otherwise it writes `msg` into the log file and marks it up as an error. `msg` can be simple, a matrix or a nested vector although simple is recommended. `more` can be any array.

If all is fine `r` is empty, otherwise it returns the message written to the log file.

That allows you to do something like this:

```
    MyLoggerInstance.LogError 1 'The error message' ⎕DM
```

or even:

```
    MyLoggerInstance.LogError rc 'FATAL ERROR' ('hello word' (1 2 3))
```

While the `Log` method is fairly restrictive in order to avoid any performance penalties the method `LogError` offers more freedom because this is hardly causing any harm: you won't have thousands of errors per second, and even if you have them performance is the least of your worries then.

## Notes 

### Important defaults 
By default a file "{yyyymmdd}.log" is created within "path" or opened if it already  exists. When a new day comes along that file is closed and a new one is created. 

This default behaviour can be switched off by setting `autoReOpen` to 0.

### Error Trapping 
By default all possible errors - accept invalid calls - are trapped withing the `Log`  method: a logging mechanism cannot be allowed to break an application which it should support. 

One exception: when creating an instance of ""Logger"" fails that causes a crash but that means it was called with invalid parameters.

However, by setting `debug` and/or `printToSession` and/or `timestamp` the `Logger` class can be debugged.

### Preconditions 

`Logger` needs the scripts `APLTreeUtils` and `WinFile`. While `APLTreeUtils` **must** be situated on the same level as `Logger` (because it is `:Included`), `WinFile` is expected to be found either on the same level as the `Logger`script or in `#` or in the namespace `Logger` got instantiated from.

If neither of this is appropriate one can specify a reference `refToUtils` pointing to the correct namespace.

## Sample session 

This code:

```
 myLogger←⎕NEW #.Logger
⍝ Exercise the "Log" method"
 myLogger.Log'this is my first entry!'
 myLogger.Log'Even' 'more' 'entries'
 myLogger.Log⊃'A' 'text' 'matrix'
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

Note that for the log entry written from its own thread the thread number is reported in the log file.

## Constructors, fields, properties and methods 

```
      ]ADOC.List Logger 
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
