# project-1-pageserver

## Description

**project-1-pageserver** is a rudimentary web server built using Python. The primary objective of this project is to enhance its capability to serve HTML and CSS files, manage various HTTP status codes, and execute both automated and manual tests to monitor progress.

## Author Credentials

- **Author Name:** Deem Alowairdhi
- **QU Number:** 411214706
- **Email:** 411214706@qu.edu.sa

## Project Details: 

### Objectives
1. Expand a minimal web server in Python to gain a deeper grasp of fundamental web architecture.
2. Incorporate automated tests to monitor progress and supplement them with manual testing.
3. Debug a server-side application.

### Project Structure
1. `pageserver.py:` The central Python script where the web server operations are defined.
2. `config.py:` A configuration module designed to handle server settings via .ini files and command-line inputs.
3. `credentials.ini:` A configuration template for users to outline settings like the author's details, repository link, port number, and document root.
4. `Makefile:` Contains instructions for the "make" build automation tool to manage project-related tasks.
5. `tests.sh:` A bash script that assesses the web server by sending HTTP requests to different URLs and comparing the response status codes and content to the expected values, returning either "Pass" or "Fail".
6. `start.sh:` A bash script to initiate the server using specific or default configurations.
7. `stop.sh:` A bash script to halt the currently running server.
8. `trivia.html:` A basic HTML page.
9. `trivia.css:` A standard CSS page.
10. `,pypid:`  A temporary placeholder file to store the process ID of the running server.

### Tasks
1. Adapt `pageserver.py` to incorporate the following features:
* **200 OK Status:** Respond with a 200 OK status if a URL terminates with `<file-name>.html` or `<file-name>.css` and the file exists in the specified document path (as outlined by DOCROOT).
* **404 Not Found Status:** If `<file-name>.html` is absent in the current directory, return a 404 Not Found status.
* **403 Forbidden Status:** Should a URL commence with prohibited symbols (like ~, //, ..), return a 403 Forbidden status.

To implement these scenarios, we need to modify the `respond` function:

* `respond` function is intended to process HTTP GET requests, fetch the requested file from the server's directory, and send an appropriate response back to the client. It is a basic representation of a function you might find in a simple HTTP server. Here, I'll break down each part of the code to explain what's happening in detail:
 
```python
import os
def respond(sock: socket):
```

This defines a function called `respond` which accepts a socket object as its parameter. The `sock: socket` syntax is a type hint indicating that the parameter `sock` should be of type `socket` and we need to import the `os` library, which allows you to interact with the operating system, including accessing file paths, reading files, etc..

```python
    request = sock.recv(1024)  # We accept only short requests
    request = str(request, encoding='utf-8', errors='strict')
```

Here, data (1024 bytes maximum) is received from the client through the socket. This data is then converted to a UTF-8 encoded string.

```python
    log.info("--- Received request ----")
    log.info(f"Request was {request}\n***\n")
```

Logging the reception of a request and its details for tracking and debugging purposes.

```python
    parts = request.split()
    if len(parts) > 1 and parts[0] == "GET":
```

The request string is split into individual parts (separated by spaces). If the request is well-formed and is a GET request, it proceeds to the next steps.

```python
        file_path = parts[1]
```

Extracting the file path from the request (usually the second part of the request line in an HTTP request).

```python
        if any(forbidden in file_path for forbidden in ("~", "//", "..")):
            transmit(STATUS_FORBIDDEN, sock)
        else:
```

Checking if the requested file path contains forbidden symbols to prevent potential directory traversal attacks. If found, it sends a 403 Forbidden status; otherwise, it proceeds.

```python
            options = get_options()
            doc_root = options.DOCROOT
            absolute_file_path = os.path.join(doc_root, file_path[1:])
```

Retrieving the document root directory from the options and forming the absolute path to the requested file.

```python
            if os.path.isfile(absolute_file_path):
```

Checks if the file exists in the server's directory.

```python
                _, ext = os.path.splitext(absolute_file_path)
                if ext in [".html", ".css"]:
                    with open(absolute_file_path, 'r') as f:
                        content = f.read()
                    transmit(STATUS_OK, sock)
                    transmit(content, sock)
```

If the file exists and its extension is either `.html` or `.css`, it reads the file content and sends a 200 OK status followed by the file content as the response.

```python
                else:
                    transmit(STATUS_NOT_FOUND, sock)
            else:
                transmit(STATUS_NOT_FOUND, sock)
```

If the file doesn't exist or the extension is not `.html` or `.css`, it sends a 404 Not Found status.

```python
    else:
        log.info(f"Unhandled request: {request}")
        transmit(STATUS_NOT_IMPLEMENTED, sock)
        transmit(f"\nI don't handle this request: {request}\n", sock)
```

If the request is not a GET request, it logs an unhandled request message and sends a 401 Not Implemented status along with an explanatory message.

```python
    sock.shutdown(socket.SHUT_RDWR)
    sock.close()
```

Finally, the socket connection is shutdown and closed, allowing resources to be freed up.

This function, thus, is a core part of a simple web server application, handling the reception and response of HTTP requests. It uses basic Python file I/O operations and socket programming concepts to accomplish its task.

### Running `pageserver`
To run the `pageserver`, a specific port number above 1000 (e.g., 1100) is required. Ensure you're in the correct directory.

To start:
```bash
$ make run
```
or
```bash
$ make start -p 1100
```
On successful execution, the following lines should appear:
```bash
Attempting to accept a connection on <socket.socket fd=3, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('0.0.0.0', 1100)> 
```

To stop:
```bash
$ make stop -p 1100
```

### Test `pageserver`

To test `pageserve`, we need to execute the `tests.sh` script. This bash script serves as an automated testing tool for a web server, ensuring the server's responses have the correct content and HTTP status codes for various URLs. The core of this script consists of specific functions designed for these validation checks.

#### Functions

1. **`expect_body`**:
    - **Purpose**: Validates that the body (content) of the server's response for a given URL contains a specific string.
    - **Parameters**:
        - `path`: The URL path to test.
        - `expect`: The string expected to be in the server's response.
    - **How it works**: It uses the `curl` command to fetch the content of a given URL and then uses `grep` to search for the expected string in that content.
    - **Output**: Displays "Pass" if the expected string is found in the response; otherwise, displays "Fail".

2. **`expect_status`**:
    - **Purpose**: Validates that the server's response for a given URL has a specific HTTP status.
    - **Parameters**:
        - `path`: The URL path to test.
        - `expect`: The expected HTTP status.
    - **How it works**: It uses `curl` to fetch the headers of the server's response and then uses `grep` to check if it contains the expected status.
    - **Output**: Displays "Pass" if the expected status is found; otherwise, displays "Fail".

3. **`real_status`**:
    - **Purpose**: A more refined version of `expect_status` that specifically checks the HTTP status line in the server's response.
    - **Parameters**:
        - `path`: The URL path to test.
        - `expect`: The expected HTTP status.
    - **How it works**: It uses `curl` to fetch both the headers and body of the server's response. Then, using `grep`, it checks the first line (the status line) of the response for the expected status.
    - **Output**: Displays "Pass" if the expected status is in the status line; otherwise, displays "Fail".



#### How to Test the `pageserver`?

* **Automated Test**

   Ensure you're in the same directory as the test file. Before executing the test commands, confirm that the `pageserver` is running.

   Run the following commands:

   ```bash
   $ ./tests.sh localhost:<portNumber>
   # For example:
   $ ./tests.sh localhost:1100
   ```

   The output will indicate the status of each test – "Pass" or "Fail":

   ![Automated Test Output](https://drive.google.com/uc?export=view&id=1g24twxyRCoZaYAy9V7yCm2xk67nKWFWR)

* **Manual Test**

   Manual testing can be done through the command line or a browser.

   1. **Command-line Testing**

      Run the following commands:

      ```bash
      $ curl http://localhost:1100/trivia.html                 # Test for a valid HTML file
      $ curl http://localhost:1100/trivia.css                  # Test for a valid CSS file
      $ curl http://localhost:1100/nonexistentfile.html        # Test for a 404 error
      $ curl http://localhost:1100/~somefile.html              # Test for a 403 error (forbidden due to "~")
      $ curl http://localhost:1100//somefile.html              # Test for a 403 error (forbidden due to "//")
      $ curl http://localhost:1100/..somefile.html             # Test for a 403 error (forbidden due to "..")
      $ curl -X POST http://localhost:1100                     # Test for a 401 error (Not Implemented due to POST request)
      ```

   2. **Browser Testing**

      Manually enter the provided URLs into your browser's address bar to perform the same tests. Omit the `curl` part of the command.

      ![Browser Testing Example](https://drive.google.com/uc?export=view&id=1dCf3AIT9G4mSQ_HUcOMe5lfSswwzMFBc)


## Challenges
Occasionally, after shutting down the pageserver, the port remains reserved for a specific process ID, saved in a file named `.pypid`. The following error may appear:
```bash
File "/your/page/server/path/pageserver.py", line 64, in listen
    server_socket.bind(('', port_num))
OSError: [Errno 98] Address already in use
```

### Solution 
To resolve this, you first need to list all processes using a specific network port on a Unix-like OS to subsequently terminate the process:
```bash
$ sudo lsof -i :portnumber
```
This command should yield output similar to:
```bash
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 51817 deem    3u  IPv4 394151      0t0  TCP *:1100 (LISTEN)
```
To terminate the process, use the process ID:
```bash
$ sudo kill -9 PID
$ sudo kill -9 51817
```

## Tools
- [Environment SetUp](https://github.com/it492/project-0-config-Deemowe#readme)
## References
- [dev.to: Rails server is already running](https://dev.to/truemark/fix-rails-server-is-already-running-532g?comments_sort=latest)
