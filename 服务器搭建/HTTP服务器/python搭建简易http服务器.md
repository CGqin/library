> - **python2:** python2 -m SimpleHTTPServer port
> - **python3:** python3 -m http.server port

```powershell
PS C:\Users\litibai> python -m http.server -h
usage: server.py [-h] [--cgi] [--bind ADDRESS] [--directory DIRECTORY] [port]

positional arguments:
  port                  specify alternate port (default: 8000)

options:
  -h, --help            show this help message and exit
  --cgi                 run as CGI server
  --bind ADDRESS, -b ADDRESS
                        specify alternate bind address (default: all interfaces)
  --directory DIRECTORY, -d DIRECTORY
                        specify alternate directory (default: current directory)
```

**实例**:

```powershell
PS C:\Users\litibai> python -m http.server 8000
Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

