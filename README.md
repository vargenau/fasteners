Fasteners
=========

[![Build status](https://travis-ci.org/harlowja/fasteners.png?branch=master)](https://travis-ci.org/harlowja/fasteners)
[![Documentation status](https://readthedocs.org/projects/fasteners/badge/?version=latest)](https://readthedocs.org/projects/fasteners/?badge=latest)
[![Downloads](https://img.shields.io/pypi/dm/fasteners.svg)](https://pypi.python.org/pypi/fasteners/)
[![Latest version](https://img.shields.io/pypi/v/fasteners.svg)](https://pypi.python.org/pypi/fasteners/)

Cross platform locks for threads and processes.

🔩 Install
----------

```
pip install fasteners
```

🔩 Usage
--------
Lock for processes has the same API as the 
[threading.Lock](https://docs.python.org/3/library/threading.html#threading.Lock)
for threads:
```python
import fasteners
import threading

lock = threading.Lock()                                 # for threads
lock = fasteners.InterProcessLock('path/to/lock.file')  # for processes

with lock:
    ... # exclusive access

# or alternatively    

lock.acquire()
... # exclusive access
lock.release()
```

Reader Writer lock has a similar API, which is the same for threads or processes:

```python
import fasteners

rw_lock = fasteners.ReaderWriterLock()                                 # for threads
rw_lock = fasteners.InterProcessReaderWriterLock('path/to/lock.file')  # for processes

with rw_lock.write_locked():
    ... # write access

with rw_lock.read_locked():
    ... # read access

# or alternatively

rw_lock.acquire_read_lock()
... # read access
rw_lock.release_read_lock()

rw_lock.acquire_write_lock()
... # write access
rw_lock.release_write_lock()
```

🔩 Overview
-----------

Python standard library provides a lock for threads (both a reentrant one, and a
non-reentrant one, see below). Fasteners extends this, and provides a lock for
processes, as well as Reader Writer locks for both threads and processes.

The specifics of the locks are as follows:

### Process locks

The `fasteners.InterProcessLock` uses [fcntl](https://man7.org/linux/man-pages/man2/fcntl.2.html) on Unix-like systems and 
msvc [_locking](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/locking?view=msvc-160) on Windows. 
As a result, if used cross-platform it guarantees an intersection of their features:

| lock | reentrant | mandatory |
|------|-----------|-----------|
| fcntl                        | ✘ | ✘ |
| _locking                     | ✔ | ✔ |
| fasteners.InterProcessLock   | ✘ | ✘ |


The `fasteners.InterProcessReaderWriterLock` also uses fcntl on Unix-like systems and 
[LockFileEx](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-lockfileex) on Windows. Their 
features are as follows:

| lock | reentrant | mandatory | upgradable | preference | 
|------|-----------|-----------|------------|------------|
| fcntl                                    | ✘ | ✘ | ✔ | reader |
| LockFileEx                               | ✔ | ✔ | ✘ | reader |
| fasteners.InterProcessReaderWriterLock   | ✘ | ✘ | ✘ | reader |


### Thread locks

Fasteners do not provide a simple thread lock, but for the sake of comparison note that the `threading` module
provides both a reentrant and non-reentrant locks:

| lock | reentrant | mandatory |
|------|-----------|-----------|
| threading.Lock  | ✘ | ✘ |
| threading.RLock | ✔ | ✘ |


The `fasteners.ReaderWriterLock` at the moment is as follows:

| lock | reentrant | mandatory | upgradable | preference | 
|------|-----------|-----------|-------------|------------|
| fasteners.ReaderWriterLock | ✔ | ✘ | ✘ | reader |

🔩 Glossary
-----------
To learn more about the various aspects of locks, check the wikipedia pages for 
[locks](https://en.wikipedia.org/wiki/Lock_(computer_science)) and 
[readers writer locks](https://en.wikipedia.org/wiki/Readers%E2%80%93writer_lock) as well as the
[resources](https://github.com/harlowja/fasteners/blob/master/doc/source/api/process_lock.rst) listed in the 
documentation. Here we briefly mention the main notions used above.

* **Lock** - a mechanism that prevents two or more threads or processes from running the same code at the same time.
* **Readers writer lock** - a mechanism that prevents two or more threads from having write (or write and read) access, 
while allowing multiple readers.
* **Reentrant lock** - a lock that can be acquired (and then released) multiple times, as in:

```python
with lock:
    with lock:
        ... # some code
```
* **Mandatory lock** (as opposed to advisory lock) - a lock that is enforced by the operating system, rather than
by the cooperation between threads or processes
* **Upgradable readers writer lock** - a readers writer lock that can be upgraded from reader to writer (or downgraded
from writer to reader) without losing the lock that is already held, as in:
```python
with rw_lock.read_locked():
    ... # read access
    with rw_lock.write_locked():
        ... # write access
    ... # read access
```
* **Readers writer lock preference** - describes the behaviour when multiple threads or processes are waiting for 
access. Some of the patterns are:
    * **Reader preference** - If lock is held by readers, then new readers will get immediate access. This can result
    in writers waiting forever (writer starvation)
    * **Writer preference** - If writer is waiting for a lock, then all the new readers (and writers) will be queued
    after the writer.
    * **Phase fair** - Lock that alternates between readers and writers.

