---
title: Logging stdout, stderr to file
layout: post
---

I needed a logger which automatically logs print statements to logfile in python logging format, without changing many statements in my original code. 

Taking [this link](https://www.electricmonk.nl/log/2011/08/14/redirect-stdout-and-stderr-to-a-logger-in-python/) as inspiration, I wrote a tiny library to log stdout, stderr to logfile.
I will release this as a python library. (#TODO)

```python
import logging
import sys

class StreamToLogger(object):
   """
   Fake file-like stream object that redirects writes to a logger instance.
   """
   def __init__(self, logger, stream=sys.stdout, log_level=logging.INFO):
      self.logger = logger
      self.stream = stream
      self.log_level = log_level
      self.linebuf = ''

   def write(self, buf):
      self.stream.write(buf)
      #self.logger.log(self.log_level, repr(buf))
      self.linebuf += buf
      if buf=='\n':
          self.flush()

   def flush(self):
      #Flush all handlers
      for line in self.linebuf.rstrip().splitlines():
         self.logger.log(self.log_level, line.rstrip())
      self.linebuf = ''
      self.stream.flush()
      for handler in self.logger.handlers:
          handler.flush()

def dual_log(*args, **kwargs):
    """
    Converts all print, raise statements to log to a file
    This function accepts all arguments to logging.basicConfig()
    """
    logging.basicConfig( *args, **kwargs)

    stdout_logger = logging.getLogger('STDOUT')
    sl = StreamToLogger(stdout_logger, sys.stdout, logging.INFO)
    sys.stdout = sl

    stderr_logger = logging.getLogger('STDERR')
    sl = StreamToLogger(stderr_logger, sys.stderr, logging.ERROR)
    sys.stderr = sl

dual_log(
   level=logging.DEBUG,
   format="%(asctime)s [%(threadName)-12.12s] [%(levelname)-5.5s]  %(message)s",
   filename="out.log",
   filemode='a'
)

from time import sleep
print("Test to standard out")
sleep(1)
print("Test to standard out")
sleep(1)
print("Test to standard out")
sleep(1)
print("Test to standard out")
sleep(1)
print("Test to standard out")
sleep(1)
raise Exception('Test to standard error')

```

Run this program and run command  `tail -f out.log`
The file `out.log` will update at same rate as print statement update in terminal.

Output:
```
Test to standard out
Test to standard out
Test to standard out
Test to standard out
Test to standard out
Traceback (most recent call last):
  File "test_stdout.py", line 65, in <module>
    raise Exception('Test to standard error')
Exception: Test to standard error
```

out.log:
```
2018-06-20 13:39:39,901 [MainThread  ] [INFO ]  Test to standard out
2018-06-20 13:39:40,902 [MainThread  ] [INFO ]  Test to standard out
2018-06-20 13:39:41,904 [MainThread  ] [INFO ]  Test to standard out
2018-06-20 13:39:42,905 [MainThread  ] [INFO ]  Test to standard out
2018-06-20 13:39:43,907 [MainThread  ] [INFO ]  Test to standard out
2018-06-20 13:39:44,909 [MainThread  ] [ERROR]  Traceback (most recent call last):
2018-06-20 13:39:44,909 [MainThread  ] [ERROR]    File "test_stdout.py", line 65, in <module>
2018-06-20 13:39:44,909 [MainThread  ] [ERROR]      raise Exception('Test to standard error')
2018-06-20 13:39:44,909 [MainThread  ] [ERROR]  Exception: Test to standard error
```