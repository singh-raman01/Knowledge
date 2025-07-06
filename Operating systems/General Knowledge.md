
*Why do we need to open and close things in the operating system?*

Doing something like:
`file,err := os.Open("someFile.txt")` , we ask the OS to allocate a file descriptor,
which is a limited low-level resource.

| Resource     | Open/Create                 | Why Close?                         |
| ------------ | --------------------------- | ---------------------------------- |
| **File**     | `os.Open`, `os.Create`      | Free file descriptor, flush buffer |
| **DB**       | `sql.Open`, `pgx.Connect`   | Close TCP sockets, free pool       |
| **Network**  | `net.Dial`, `http.Get`      | Free port/socket, prevent leak     |
| **Channels** | `make(chan)`                | `close(ch)` to signal done         |
| **Mutex**    | `Lock()`                    | `Unlock()` to prevent deadlock     |
| **Memory**   | `new`, `make`, or `&Type{}` | Let GC collect when unreferenced   |
Not closing them, means the OS doesn't know when we are done with them.