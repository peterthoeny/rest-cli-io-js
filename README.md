# REST CLI I/O (rest-cli-io)

[![GitHub issues](https://img.shields.io/github/issues/peterthoeny/rest-cli-io)](https://github.com/peterthoeny/rest-cli-io/issues)
[![GitHub stars](https://img.shields.io/github/stars/peterthoeny/rest-cli-io)](https://github.com/peterthoeny/rest-cli-io/stargazers)
[![GitHub license](https://img.shields.io/github/license/peterthoeny/rest-cli-io)](https://github.com/peterthoeny/rest-cli-io/blob/master/LICENSE)

The REST Command Line Interface I/O (rest-cli-io) is a node.js application to execute shell scripts and system commands securely via a REST API.

## Getting Started

    $ git clone https://github.com/peterthoeny/rest-cli-io.git # or clone your own fork
    $ cd rest-cli-io
    $ sudo cp -p rest-cli-io.conf /etc   # modify as desired
    $ npm install
    $ node rest-cli-io

Visit http://localhost:8071/ to access the REST CLI I/O API.

## REST CLI I/O API Documentation

The REST CLI I/O API is mainly intended to be used on an Intranet to automate processes.

For security, only registered commands and scripts are available via the REST CLI I/O API. Commands are identified with an ID (symbolic name), and each command has a number of options that define how to execute the command. See [Conf Settings](#conf-settings) below.

### Execute Command

- Endpoint: `GET /api/1/cli/run/<commandID>?<param>=<value>`
  - `<commandID>`: Registered command ID
  - Add optional command specific URI parameters, such as `p=hello+world&q=goodbye`
  - Example: http://localhost:8071/api/1/cli/man?p=which
- Return:
  - If ok: (text output of command, or a JSON object as defined in the settings)
  - If error: `{ "error": "Unrecognized command ID <commandID>" }`
- Optionally add a return content-type:
  - Example: http://localhost:8071/api/1/cli/man?p=which&contentType=text/html
  - Return: Command output, delivered with specified content-type

### List Command IDs

- Endpoint: `GET /api/1/cli/list`
  - Return: `{ "data": [ "<id1>", "<id2>" ], "error": "" }`
  - Example: http://localhost:8071/api/1/cli/list
  - Available only if enabled with `allowCmdList` setting

### Conf Settings

Modify the settings in `rest-cli-io.conf` located in `/etc` or the rest-cli-io application directory.

Example conf settings:

    conf = {
        commands: {
            echo: {
                command:        'echo',
                arguments:      [ '%PARAM{p}%' ],
                options:        {},
                output:         {}
            },
            expr: {
                command:        'expr',
                arguments:      [ '%PARAM{p:split}%' ],
            }
        },
        allowCmdList:   1,
        port:           8071
    };

#### `commands`

The `commands` setting lists all commands. Each command has these properties:

`command`: The actual command or script

- The PATH environmental variable is used as a search path if no path is specified
- Use `./` prefix to point to the rest-cli-io directory root
- The referenced commands must be executable by the rest-cli-io application user
- rest-cli-io has a `bin` directory with a sample `man.sh` script that can be referenced as: `./bin/man.sh`

`arguments`: Array of arguments

- Specify a command specific argument list
- Reference URI parameter values with `%PARAM{<name>}%`, such as `%PARAM{start}%`
- If you want to split a parameter value on spaces, specify `%PARAM{<name>:split}%`, such as `%PARAM{expr:split}%`, which will split up parameter `expr=6+/+2` into arguments `[ '6', '/', '2' ]`

`options`: Spawn options

- The child-process spawn options are optional
- Use as defined in https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options
- *ATTENTION:* Do not use the `shell` property, where any input containing shell metacharacters may be used to trigger arbitrary command execution

`output.body`: Body format on regular command execution

- Optional, defaults to just `%STDOUT%`
- Can be a JSON object, such as:<br/>
  `{ stdout: '%STDOUT%', stderr: '%STDERR%', exit: '%CODE%' }`
- `%STDOUT%`: Is the `stdout` of the command
- `%STDERR%`: Is the `stderr` of the command
- `%CODE%`: Is the exit code of the command, typically `0` on no error

`output.error`: Body format on error

- Optional, defaults to `Error: %STDERR%\nCode: %CODE%`
- Can be a JSON object as in the previous example

`contentType`: Content-Type

- Optional, automatically set based on type of body (`text/plain` for text, or `application/json` for JSON)
- This setting can be redefined with a `contentType` URI parameter

#### `allowCmdList`

The `allowCmdList` setting determines if the command list is available via the `/api/1/cli/list` REST endpoint. Set to `0` to disable.

#### `port`

The `port` setting defines the port number the application is using.

// EOF
