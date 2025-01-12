
**This is a fork of https://github.com/juju/fslock/fork to update go-modules usage**

**Info** when you want to write to a file "text.txt" it is a good idea to create the lock as "text.txt.lock" so you can still open the actual file normally. 

example usage:
```
package main
import (
    "time"
    "fmt"
    fslock "github.com/Zeg0/fslock" 
)

func main() {
    lock := fslock.New("../lock.txt")
    lockErr := lock.TryLock()
    if lockErr != nil {
        // unable to acquire lock
        fmt.Println("falied to acquire lock > " + lockErr.Error())
    } else {
        // ok
        defer lock.Unlock()
        // write to file or something
        fmt.Println("got the lock")
        time.Sleep(1 * time.Minute)
    }
}
```
and use `go mod tidy`



-------------

# fslock [![GoDoc](https://godoc.org/github.com/juju/fslock?status.svg)](https://godoc.org/github.com/juju/fslock)
fslock provides a cross-process mutex based on file locks that works on windows and *nix platforms.


![fslock](https://cloud.githubusercontent.com/assets/3185864/15507515/f3351498-2199-11e6-9f37-bc59657a9e8c.jpg)

<sup><sub>image: [public domain](https://pixabay.com/en/encrypted-privacy-policy-445155/)
(don't ask)
</sub></sup>

fslock relies on LockFileEx on Windows and flock on \*nix systems.  The timeout 
feature uses overlapped IO on Windows, but on \*nix platforms, timing out
requires the use of a goroutine that will run until the lock is acquired,
regardless of timeout.  If you need to avoid this use of goroutines, poll
TryLock in a loop. 



## Variables
``` go
var ErrLocked error = trylockError("fslock is already locked")
```
ErrLocked indicates TryLock failed because the lock was already locked.

``` go
var ErrTimeout error = timeoutError("lock timeout exceeded")
```
ErrTimeout indicates that the lock attempt timed out.


## type Lock
``` go
type Lock struct {
    // contains filtered or unexported fields
}
```
Lock implements cross-process locks using syscalls.


### func New
``` go
func New(filename string) *Lock
```
New returns a new lock around the given file.


### func (\*Lock) Lock
``` go
func (l *Lock) Lock() error
```
Lock locks the lock.  This call will block until the lock is available.

### func (\*Lock) LockWithTimeout
``` go
func (l *Lock) LockWithTimeout(timeout time.Duration) error
```
LockWithTimeout tries to lock the lock until the timeout expires.  If the
timeout expires, this method will return ErrTimeout.

### func (\*Lock) TryLock
``` go
func (l *Lock) TryLock() error
```
TryLock attempts to lock the lock.  This method will return ErrLocked
immediately if the lock cannot be acquired.

### func (\*Lock) Unlock
``` go
func (l *Lock) Unlock() error
```
Unlock unlocks the lock.


