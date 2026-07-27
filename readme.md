<h1 align="center">
    <a href="https://github.com/engineer-man/piston">
        <img src="var/docs/images/piston.svg" valign="middle" width="58" height="58" alt="engineer-man piston" />
    </a>
    <span valign="middle">
        Piston
    </span>
</h1>

<h3 align="center">A high performance general purpose code execution engine.</h3>

<br>

<p align="center">
    <a href="https://github.com/engineer-man/piston/commits/master">
    <img src="https://img.shields.io/github/last-commit/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub last commit">
    <a href="https://github.com/engineer-man/piston/issues">
    <img src="https://img.shields.io/github/issues/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub issues">
    <a href="https://github.com/engineer-man/piston/pulls">
    <img src="https://img.shields.io/github/issues-pr-raw/engineer-man/piston.svg?style=for-the-badge&logo=github&logoColor=white"
         alt="GitHub pull requests">
</p>

---

<h4 align="center">
  <a href="#About">About</a> •
  <a href="#Public-API">Public API</a> •
  <a href="#Getting-Started">Getting Started</a> •
  <a href="#Usage">Usage</a> •
  <a href="#Supported-Languages">Supported Languages</a> •
  <a href="#Principle-of-Operation">Principles</a> •
  <a href="#Security">Security</a> •
  <a href="#Standalone-Deployments-Dockerfileprebuilt">Standalone Deployments</a> •
  <a href="#License">License</a> •
  <a href="https://piston.readthedocs.io">Documentation</a>
</h4>

---

<br>

# About

<h4>
    Piston is a high performance general purpose code execution engine. It excels at running untrusted and
    possibly malicious code without fear from any harmful effects.
</h4>

<br>

It's used in numerous places including:

-   [EMKC Challenges](https://emkc.org/challenges)
-   [EMKC Weekly Contests](https://emkc.org/contests)
-   [Engineer Man Discord Server](https://discord.gg/engineerman)
-   Web IDEs
-   200+ direct integrations

<br>

### Official Extensions

The following are approved and endorsed extensions/utilities to the core Piston offering.

-   [I Run Code](https://github.com/engineer-man/piston-bot), a Discord bot used in 4100+ servers to handle arbitrary code evaluation in Discord. To get this bot in your own server, go here: https://emkc.org/run.
-   [Piston CLI](https://github.com/Shivansh-007/piston-cli), a universal shell supporting code highlighting, files, and interpretation without the need to download a language.
-   [Node Piston Client](https://github.com/dthree/node-piston), a Node.js wrapper for accessing the Piston API.
-   [Piston4J](https://github.com/the-codeboy/Piston4J), a Java wrapper for accessing the Piston API.
-   [Pyston](https://github.com/ffaanngg/pyston), a Python wrapper for accessing the Piston API.
-   [Go-Piston](https://github.com/milindmadhukar/go-piston), a Golang wrapper for accessing the Piston API.
-   [piston_rs](https://github.com/Jonxslays/piston_rs), a Rust wrapper for accessing the Piston API.
-   [piston_rspy](https://github.com/Jonxslays/piston_rspy), Python bindings for accessing the Piston API via `piston_rs`.

<br>

# Public API

-   Requires no installation but must obtain an authorization token to use it, see important note below on how to do this.
-   Reference the Runtimes/Execute sections below to learn about the request and response formats.

<br>

When using the public Piston API, use the following two URLs:

```
GET  https://emkc.org/api/v2/piston/runtimes
POST https://emkc.org/api/v2/piston/execute
```

> Important Note: The Piston API is no longer freely available to the public (as of Feb 15, 2026). To obain authorization, please reach out to EngineerMan on [Discord](https://discord.gg/engineerman). Authorization is only granted for non-commercial use (no paid products), low volume, and particular those in the educational space. Keys **are not** granted for individual projects, portfolio projects, university assignments, conceptual projects, vibe-coded ai slop projects, or projects that generally don't benefit anyone. I reserve complete discretion on key issuance. If a key is not issued, you are more than welcome to run your own instance of Piston, as it is open source, after all.

<br>

# Getting Started

## All In One

### Host System Package Dependencies

-   Docker
-   Docker Compose
-   Node JS (>= 15)
-   cgroup v2 enabled, and cgroup v1 disabled

### After system dependencies are installed, clone this repository:

```sh
# clone and enter repo
git clone https://github.com/engineer-man/piston
```

> [!NOTE]
>
> Ensure the repository is cloned with LF line endings

### Installation

```sh
# Start the API container
docker-compose up -d api

# Install all the dependencies for the cli
cd cli && npm i && cd -
```

The API will now be online with no language runtimes installed. To install runtimes, [use the CLI](#cli).

## Just Piston (no CLI)

### Host System Package Dependencies

-   Docker

### Installation

```sh
docker run \
    --privileged \
    -v $PWD:'/piston' \
    -dit \
    -p 2000:2000 \
    --name piston_api \
    ghcr.io/engineer-man/piston
```

## Piston for testing packages locally

### Host System Package Dependencies

-   Same as [All In One](#All-In-One)

### Installation

```sh
# Build the Docker containers
./piston start

# For more help
./piston help
```

<br>

# Usage

### CLI

The CLI is the main tool used for installing packages within piston, but also supports running code.

You can execute the cli with `cli/index.js`.

```sh
# List all available packages
cli/index.js ppman list

# Install latest python
cli/index.js ppman install python

# Install specific version of python
cli/index.js ppman install python=3.9.4

# Run a python script using the latest version
echo 'print("Hello world!")' > test.py
cli/index.js run python test.py

# Run a python script using a specific version
echo 'print("Hello world!")' > test.py
cli/index.js run python test.py -l 3.9.4
cli/index.js run python test.py -l 3.x
cli/index.js run python test.py -l 3
```

If you are operating on a remote machine, add the `-u` flag like so:

```sh
cli/index.js -u http://piston.server:2000 ppman list
```

### API

The container exposes an API on port 2000 by default.
This is used by the CLI to carry out running jobs and package management.

#### Runtimes Endpoint

`GET /api/v2/runtimes`
This endpoint will return the supported languages along with the current version and aliases. To execute
code for a particular language using the `/api/v2/execute` endpoint, either the name or one of the aliases must
be provided, along with the version.
Multiple versions of the same language may be present at the same time, and may be selected when running a job.

```json
HTTP/1.1 200 OK
Content-Type: application/json

[
    {
        "language": "bash",
        "version": "5.1.0",
        "aliases": [
            "sh"
        ]
    },
    {
        "language": "brainfuck",
        "version": "2.7.3",
        "aliases": [
            "bf"
        ]
    },
    ...
]
```

#### Execute Endpoint

`POST /api/v2/execute`
This endpoint requests execution of some arbitrary code.

-   `language` (**required**) The language to use for execution, must be a string and must be installed.
-   `version` (**required**) The version of the language to use for execution, must be a string containing a SemVer selector for the version or the specific version number to use.
-   `files` (**required**) An array of files containing code or other data that should be used for execution. The first file in this array is considered the main file.
-   `files[].name` (_optional_) The name of the file to upload, must be a string containing no path or left out.
-   `files[].content` (**required**) The content of the files to upload, must be a string containing text to write.
-   `files[].encoding` (_optional_) The encoding scheme used for the file content. One of `base64`, `hex` or `utf8`. Defaults to `utf8`.
-   `stdin` (_optional_) The text to pass as stdin to the program. Must be a string or left out. Defaults to blank string.
-   `args` (_optional_) The arguments to pass to the program. Must be an array or left out. Defaults to `[]`.
-   `compile_timeout` (_optional_) The maximum wall-time allowed for the compile stage to finish before bailing out in milliseconds. Must be a number or left out. Defaults to `10000` (10 seconds).
-   `run_timeout` (_optional_) The maximum wall-time allowed for the run stage to finish before bailing out in milliseconds. Must be a number or left out. Defaults to `3000` (3 seconds).
-   `compile_cpu_time` (_optional_) The maximum CPU-time allowed for the compile stage to finish before bailing out in milliseconds. Must be a number or left out. Defaults to `10000` (10 seconds).
-   `run_cpu_time` (_optional_) The maximum CPU-time allowed for the run stage to finish before bailing out in milliseconds. Must be a number or left out. Defaults to `3000` (3 seconds).
-   `compile_memory_limit` (_optional_) The maximum amount of memory the compile stage is allowed to use in bytes. Must be a number or left out. Defaults to `-1` (no limit)
-   `run_memory_limit` (_optional_) The maximum amount of memory the run stage is allowed to use in bytes. Must be a number or left out. Defaults to `-1` (no limit)

```json
{
    "language": "js",
    "version": "15.10.0",
    "files": [
        {
            "name": "my_cool_code.js",
            "content": "console.log(process.argv)"
        }
    ],
    "stdin": "",
    "args": ["1", "2", "3"],
    "compile_timeout": 10000,
    "run_timeout": 3000,
    "compile_cpu_time": 10000,
    "run_cpu_time": 3000,
    "compile_memory_limit": -1,
    "run_memory_limit": -1
}
```

A typical response upon successful execution will contain 1 or 2 keys `run` and `compile`.
`compile` will only be present if the language requested requires a compile stage.

Each of these keys has an identical structure, containing both a `stdout` and `stderr` key, which is a string containing the text outputted during the stage into each buffer.
It also contains the `code` and `signal` which was returned from each process. It also includes a nullable human-readable `message` which is a description of why a stage has failed and a two-letter `status` that is either:

-   `RE` for runtime error
-   `SG` for dying on a signal
-   `TO` for timeout (either via `timeout` or `cpu_time`)
-   `OL` for stdout length exceeded
-   `EL` for stderr length exceeded
-   `XX` for internal error

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "language": "js",
    "version": "15.10.0",
    "run": {
        "stdout": "[\n  '/piston/packages/node/15.10.0/bin/node',\n  '/piston/jobs/9501b09d-0105-496b-b61a-e5148cf66384/my_cool_code.js',\n  '1',\n  '2',\n  '3'\n]\n",
        "stderr": "",
        "output": "[\n  '/piston/packages/node/15.10.0/bin/node',\n  '/piston/jobs/9501b09d-0105-496b-b61a-e5148cf66384/my_cool_code.js',\n  '1',\n  '2',\n  '3'\n]\n",
        "code": 0,
        "signal": null,
        "message": null,
        "status": null,
        "cpu_time": 8,
        "wall_time": 154,
        "memory": 1160000
    }
}
```

If a problem exists with the request, a `400` status code is returned and the reason in the `message` key.

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "message": "html-5.0.0 runtime is unknown"
}
```

#### Interactive execution endpoint (not available through the public API)

To interact with running processes in real time, you can establish a WebSocket connection at `/api/v2/connect`. This allows you to both receive output and send input to active processes.

Each message is structured as a JSON object with a `type` key, which indicates the action to perform. Below is a list of message types, their directions, and descriptions:

-   **init** (client -> server): Initializes a job with the same parameters as the `/execute` endpoint, except that stdin is discarded.
-   **runtime** (server -> client): Provides details on the runtime environment, including the version and language.
-   **stage** (server -> client): Indicates the current execution stage, either "compile" or "run."
-   **data** (server <-> client): Exchanges data between the client and server, such as stdin, stdout, or stderr streams.
-   **signal** (client -> server): Sends a signal (e.g., for termination) to the running process, whether it's in the "compile" or "run" stage.
-   **exit** (server -> client): Signals the end of a stage, along with the exit code or signal.
-   **error** (server -> client): Reports an error, typically right before the WebSocket is closed.

An example of this endpoint in use is depicted below (**<** = client to server, **>** = server to client)

1. Client establishes WebSocket connection to `/api/v2/connect`
2. **<** `{"type":"init", "language":"bash", "version":"*", "files":[{"content": "cat"}]}`
3. **>** `{"type":"runtime","language": "bash", "version": "5.1.0"}`
4. **>** `{"type":"stage", "stage":"run"}`
5. **<** `{"type":"data", "stream":"stdin", "data":"Hello World!"}`
6. **>** `{"type":"data", "stream":"stdout", "data":"Hello World!"}`
7. _time passes_
8. **>** `{"type":"exit", "stage":"run", "code":null, "signal": "SIGKILL"}`

Errors may return status codes as follows:

-   **4000: Already Initialized**: Sent when a second `init` command is issued.
-   **4001: Initialization Timeout**: No `init` command was sent within 1 second of connection.
-   **4002: Notified Error**: A fatal error occurred, and an `error` packet was transmitted.
-   **4003: Not yet Initialized**: A non-`init` command was sent without a job context.
-   **4004: Can only write to stdin**: The client attempted to write to a stream other than stdin.
-   **4005: Invalid Signal**: An invalid signal was sent in a `signal` packet.

<br>

# Supported Languages

`awk`,
`bash`,
`befunge93`,
`brachylog`,
`brainfuck`,
`bqn`,
`c`,
`c++`,
`cjam`,
`clojure`,
`cobol`,
`coffeescript`,
`cow`,
`crystal`,
`csharp`,
`csharp.net`,
`d`,
`dart`,
`dash`,
`dragon`,
`elixir`,
`emacs`,
`emojicode`,
`erlang`,
`file`,
`forte`,
`forth`,
`fortran`,
`freebasic`,
`fsharp.net`,
`fsi`,
`go`,
`golfscript`,
`groovy`,
`haskell`,
`husk`,
`iverilog`,
`japt`,
`java`,
`javascript`,
`jelly`,
`julia`,
`kotlin`,
`lisp`,
`llvm_ir`,
`lolcode`,
`lua`,
`matl`,
`nasm`,
`nasm64`,
`nim`,
`ocaml`,
`octave`,
`osabie`,
`paradoc`,
`pascal`,
`perl`,
`php`,
`ponylang`,
`powershell`,
`prolog`,
`pure`,
`pyth`,
`python`,
`python2`,
`racket`,
`raku`,
`retina`,
`rockstar`,
`rscript`,
`ruby`,
`rust`,
`samarium`,
`scala`,
`smalltalk`,
`sqlite3`,
`swift`,
`typescript`,
`basic`,
`basic.net`,
`vlang`,
`vyxal`,
`yeethon`,
`zig`,

<br>

# Principle of Operation

Piston uses [Isolate](https://www.ucw.cz/moe/isolate.1.html) inside Docker as the primary mechanism for sandboxing. There is an API within the container written in Node
which takes in execution requests and executes them within the container safely.
High level, the API writes any source code and executes it inside an Isolate sandbox.
The source file is either ran or compiled and ran (in the case of languages like c, c++, c#, go, etc.).

<br>

# Security

Piston uses Isolate which makes use of Linux namespaces, chroot, multiple unprivileged users, and cgroup for sandboxing and resource limiting. Code execution submissions on Piston shall not be aware of each other, shall not affect each other and shall not affect the underlying host system. This is ensured through multiple steps including:

-   Disabling outgoing network interaction by default
-   Capping max processes at 256 by default (resists `:(){ :|: &}:;`, `while True: os.fork()`, etc.)
-   Capping max files at 2048 (resists various file based attacks)
-   Cleaning up all temp space after each execution (resists out of drive space attacks)
-   Running each submission as a different unprivileged user
-   Running each submission with its own isolated Linux namespaces
-   Capping runtime execution at 3 seconds by default (CPU-time and wall-time)
-   Capping the peak memory that all the submission's processes can use
-   Capping stdout to 1024 characters by default (resists yes/no bombs and runaway output)
-   SIGKILLing misbehaving code

<br>

# Standalone Deployments (Dockerfile.prebuilt)

The normal setup (`api/Dockerfile` + `docker-compose.dev.yaml`) installs runtimes at
container **start time**: an `api` container and a `repo` container share a Docker
volume, and `ppman install <language>=<version>` downloads a package from the `repo`
container's index into that volume. This is flexible -- add or swap a language without
rebuilding the API image -- but it depends on having both a sidecar container and a
persistent/shared volume.

Some platforms don't offer either. AWS App Runner, for example, runs a single
container per service with no sidecar and no writable volume that survives a
redeploy. For that class of deployment, `Dockerfile.prebuilt` (repo root) builds a
single self-contained image with a **fixed** set of runtimes compiled and baked
directly into `/piston/packages` at `docker build` time, so the container is ready to
execute code the moment it boots -- no `repo` service, no volume, no `ppman install`
call required.

```
docker build -f Dockerfile.prebuilt -t my-piston-prebuilt .
docker run --privileged -p 2000:2000 my-piston-prebuilt
```

The tradeoff: the runtime set is fixed at build time. Adding a language, or bumping a
version, means editing the Dockerfile and rebuilding the image -- there's no live
install step.

### Adding another runtime

`Dockerfile.prebuilt` has three stages: `pkgbuilder` (compiles the packages),
`isolate` (builds the sandbox binary, unchanged from `api/Dockerfile`), and the final
image. To bake in an additional language, touch only the `pkgbuilder` build and the
two blocks in the final stage that copy a built package in and register it.

As an example, here's adding `ruby=3.0.1` (an existing package under
`packages/ruby/3.0.1/`) alongside the four runtimes already baked in:

1. **Add it to the `make` target list** in the `pkgbuilder` stage, so it actually gets
   compiled during the build:

   ```diff
    RUN cd packages && make -j"$(nproc)" \
            python-3.14.0.pkg.tar.gz \
            node-20.11.1.pkg.tar.gz \
            typescript-5.0.3.pkg.tar.gz \
            bash-5.2.0.pkg.tar.gz \
   +        ruby-3.0.1.pkg.tar.gz \
            PLATFORM=docker-debian
   ```

2. **Copy the built package dir** into the final image, in the `COPY --from=pkgbuilder`
   block:

   ```diff
    COPY --from=pkgbuilder /piston/packages/bash/5.2.0        /piston/packages/bash/5.2.0
   +COPY --from=pkgbuilder /piston/packages/ruby/3.0.1         /piston/packages/ruby/3.0.1
   ```

3. **Add it to the `for pkgdir in ...` loop** that generates `.env` and
   `.ppman-installed` for each package (these two files are normally only created by a
   live `ppman install` -- see `api/src/package.js`'s `install()` -- so the loop
   recreates them at build time instead):

   ```diff
    RUN for pkgdir in \
            /piston/packages/python/3.14.0 \
            /piston/packages/node/20.11.1 \
            /piston/packages/typescript/5.0.3 \
            /piston/packages/bash/5.2.0 \
   +        /piston/packages/ruby/3.0.1 \
        ; do \
   ```

That's it -- no other stage needs to change. If the language doesn't already exist
under `packages/`, follow `packages/CONTRIBUTING.MD` to add it first (copy the nearest
existing version dir, adjust `build.sh`/`run`/`metadata.json`), confirm it builds and
passes its `test.*` via the normal dev flow (`./piston build-pkg`, `./piston ppman
install`, `./piston run`), *then* add it to `Dockerfile.prebuilt` using the steps
above.

<br>

# License

Piston is licensed under the MIT license.
