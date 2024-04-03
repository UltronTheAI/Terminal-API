# Terminal API

The Terminal API is a Flask-based application that provides an interface for executing commands in a terminal-like environment. It is designed to be used by AI systems that need to interact with a terminal or command line interface.

## Usage

To use the Terminal API, send HTTP requests to the appropriate endpoints with the required JSON objects. The API runs on `localhost` at port `5000`.

Please note that this API is intended for use by AI systems and should not be exposed to untrusted clients, as it allows arbitrary command execution. Always ensure that the API is secured appropriately.

## Modules Used

```python
from flask import Flask, request, jsonify
import subprocess
import time
import threading
```

## Endpoints

Here are the available endpoints and their descriptions:

1. **POST /runCommand**
   - This endpoint accepts a POST request with a JSON body containing a `command` field. It executes the command in a terminal-like environment. If the terminal is not already running, it starts a new one. The output of the command is read in a separate thread to avoid blocking. The endpoint returns a JSON response indicating that the command was executed.

```python
@app.route('/runCommand', methods=['POST'])
def run_command():
    command = request.json['command'].encode('utf-8')
    if terminal is None:
        terminal = subprocess.Popen('cmd', stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
        threading.Thread(target=read_output, args=(terminal,)).start()
    terminal.stdin.write(command)
    terminal.stdin.write(b'\n')
    terminal.stdin.flush()
    return jsonify({'status': 'Command executed'}), 200
```

2. **GET /getOutput**
   - This endpoint returns the output of the last executed command. If no command has been executed yet, it returns an error message. The output is read in a separate thread to avoid blocking.

```python
@app.route('/getOutput', methods=['GET'])
def get_output():
    if terminal is not None:
        threading.Thread(target=read_output, args=(terminal,)).start()
        if output == '':
            return jsonify({'error': 'All lines have been read'}), 400
        else:
            return jsonify({'output': output}), 200
    else:
        return jsonify({'error': 'No command has been executed yet'}), 400
```

3. **GET /new**
   - This endpoint creates a new terminal process and returns a JSON response indicating that the new terminal was created successfully.

```python
@app.route('/new', methods=['GET'])
def new_terminal():
    global terminal
    terminal = subprocess.Popen('cmd', stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
    return jsonify({'status': 'New terminal created'}), 200
```

4. **GET /reset**
   - This endpoint resets the terminal by killing the current terminal process and setting it to `None`. It returns a JSON response indicating that the terminal was reset successfully.

```python
@app.route('/reset', methods=['GET'])
def reset_terminal():
    global terminal
    if terminal is not None:
        terminal.kill()
        terminal = None
    return jsonify({'status': 'Terminal reset'}), 200
```

5. **POST /userinput**
   - This endpoint accepts a POST request with a JSON body containing an `input` field. It sends the user input to the terminal and returns a JSON response indicating that the user input was sent successfully.

```python
@app.route('/userinput', methods=['POST'])
def user_input():
    global terminal
    user_input = request.json['input'].encode('utf-8')
    if terminal is not None:
        terminal.stdin.write(user_input)
        terminal.stdin.write(b'\n')
        terminal.stdin.flush()
        return jsonify({'status': 'User input sent'}), 200
    else:
        return jsonify({'error': 'No command has been executed yet'}), 400
```

6. **POST /changedir**
   - This endpoint accepts a POST request with a JSON body containing a `directory` field. It changes the current directory in the terminal to the specified directory and returns a JSON response indicating that the directory was changed successfully.

```python
@app.route('/changedir', methods=['POST'])
def change_dir():
    global terminal
    directory = request.json['directory']
    if terminal is not None:
        terminal.stdin.write(f'cd {directory}'.encode())
        terminal.stdin.write(b'\n')
        terminal.stdin.flush()
        return jsonify({'status': 'Directory changed'}), 200
    else:
        return jsonify({'error': 'No command has been executed yet'}), 400
```

7. **POST /runApp**
   - This endpoint accepts a POST request with a JSON body containing an `app_name` field. It starts the specified application in the terminal and returns a JSON response indicating that the app was started successfully.

```python
@app.route('/runApp', methods=['POST'])
def run_app():
    global terminal, output
    app_name = request.json['app_name']
    if terminal is None:
        terminal = subprocess.Popen(app_name, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
        threading.Thread(target=read_output, args=(terminal,)).start()
    return jsonify({'status': 'App started'}), 200
```

8. **POST /postInputToApp**
   - This endpoint accepts a POST request with a JSON body containing an `input` field. It sends the user input to the running application and returns a JSON response indicating that the user input was sent to the app successfully.

```python
@app.route('/postInputToApp', methods=['POST'])
def post_input_to_app():
    global terminal
    user_input = request.json['input'].encode('utf-8')
    if terminal is not None:
        terminal.stdin.write(user_input)
        terminal.stdin.write(b'\n')
        terminal.stdin.flush()
        terminal.stdin.close()
        time.sleep(1)
        return jsonify({'status': 'User input sent to app'}), 200
    else:
        return jsonify({'error': 'No app has been started yet'}), 400
```

9. **GET /closeApp**
   - This endpoint closes the currently running app by killing the terminal process and setting it to `None`. It returns a JSON response indicating that the app was closed successfully.

```python
@app.route('/closeApp', methods=['GET'])
def close_app():
    global terminal
    if terminal is not None:
        terminal.kill()
        terminal = None
        return jsonify({'status': 'App closed'}), 200
    else:
        return jsonify({'error': 'No app has been started yet'}), 400
```

10. **GET /outputofapp**
    - This endpoint returns the output of the currently running app. If no app is running, it returns an error message.

```python
@app.route('/outputofapp', methods=['GET'])
def output_of_app():
    global terminal, output
    if terminal is not None:
        with output_lock:
            if output == '':
                return jsonify({'error': 'All lines have been read'}), 400
            else:
                result = output
                output = ""
                return jsonify({'output': result}), 200
    else:
        return jsonify({'error': 'No app has been started yet'}), 400
```
