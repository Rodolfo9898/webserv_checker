*This project has been created as part of the 42 curriculum by rperez-t.*

# webserv_checker

## Description

The provided testers of the webserv project, with one small program in front of them. The subject gives a tester that checks a
running server, and a CGI program (`cgi_test` in the subject) that the server has to run for the files ending in `.bla`. Here
the two are kept together, and `tester_OS` is the single name to call for either of them, on Linux and on macOS alike.

| File | What it is |
|---|---|
| `tester_OS` | A `sh` script at the root. Runs the program below that you ask for: directly on Linux x86-64, in a Linux container (docker) anywhere else |
| `src/tester` | The tester, as downloaded |
| `src/cgi_tester` | The CGI program, as downloaded |

The tester takes the address of the server as its argument (`http://localhost:4442`), prints the configuration it expects, sends
its requests and shows the answers in the terminal. It ends with `GG, So far so good! Run your own tests now! :D`, and says
itself that it is the minimum before an evaluation, not the whole of it. The CGI program is not something you start: the
configuration of the server names it, and the server runs it, with the request in the CGI environment and the body on its
standard input.

## Instructions

### Requirements

- Linux on x86-64: nothing to install. Both programs are Linux x86-64 programs (ELF).
- Anything else (macOS, Apple Silicon or Intel, an ARM Linux): docker, running. The programs then run in a
  `debian:bookworm-slim` container started with `--platform linux/amd64`; the first run downloads the image. On Apple Silicon
  docker emulates x86-64, so it starts a little slower.
- Nothing to build.

### Setting up the server

The tester asks for this, in its own words (its text is printed when it starts):

- `/` answers `GET` only;
- any file ending in `.bla` answers `POST` by calling the `cgi_tester` executable;
- `/post_body` answers anything to `POST` with a maxBody of 100;
- `/directory/` answers `GET`, its root is the `YoupiBanane` folder, and with no file requested it looks for `youpi.bad_extension`.

And a `YoupiBanane` folder, whose files can have any content:

```sh
mkdir -p YoupiBanane/nop YoupiBanane/Yeah
touch YoupiBanane/youpi.bad_extension YoupiBanane/youpi.bla
touch YoupiBanane/nop/youpi.bad_extension YoupiBanane/nop/other.pouic
touch YoupiBanane/Yeah/not_happy.bad_extension
```

The path of the CGI program for the configuration is `src/cgi_tester`, in the folder of this repository.

### Running

Clone the repository (or copy `tester_OS` and `src/` together into a folder: the script looks for the programs in the `src/` next
to it). Start the server, then call `tester_OS` in another terminal with its address:

```sh
./tester_OS http://localhost:4442
```

The first word can name the program: `tester_OS tester http://localhost:4442` is the same, and `tester_OS cgi_tester` is the CGI
program. Arguments, standard input, output and exit status are those of the program. On Linux x86-64 `tester_OS` replaces
itself with it (`exec`).

In the webserv repository this is a submodule, `eval_helper/webserv_checker`, and `make` links the script at the root, so
`./tester_OS` works from there. To call it from anywhere, link it to a folder of your `PATH`. The link is followed back to the
real files:

```sh
ln -s "$PWD/tester_OS" ~/.local/bin/tester_OS
```

### What `tester_OS` does

1. Finds its own folder, following links, so it works from any directory.
2. Takes the first argument for the name of the program when it is `tester` or `cgi_tester`; otherwise the program is the tester
   and every argument is for it.
3. Makes the programs executable if they are not: a download or a zip does not keep that.
4. On Linux x86-64, runs the program with your arguments.
5. Anywhere else, needs docker and runs the same program in a container: `docker run --rm -i --platform linux/amd64`, the folder
   `src/` mounted read-only, `TERM` passed on, and a terminal (`-t`) when yours is one, because the tester draws in the terminal
   ([termbox](https://github.com/nsf/termbox-go)). In the arguments, an address that starts with `http://localhost`,
   `https://localhost`, `http://127.0.0.1` or `https://127.0.0.1` becomes `host.docker.internal`: inside the container localhost is
   the container, and the server runs on the machine that runs docker. Set `TESTER_IMAGE` to use another image than
   `debian:bookworm-slim`.

### The limit of the container

The tester reaches a server that runs on the Mac through `host.docker.internal`, so everything that is a request works. What the
server runs itself does not: the server on the Mac cannot execute `src/cgi_tester`, a Linux program, for the `.bla` files. For
that part run the server on Linux too: on a Linux machine, or in a container that has both (in webserv,
`make docker-shell`, then start the server and call `./tester_OS http://localhost:4442` in the same container).

### Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `tester_OS: Darwin arm64 cannot run the testers (Linux x86-64 programs)` | docker is not installed or not in the `PATH`. Install it, or use a Linux x86-64 machine |
| `Cannot connect to the Docker daemon` | Docker Desktop is not running |
| `Before starting please verify that the server is launched` | The server does not answer at that address: start it, and check the port |
| `termbox: TERM environment variable not set` | The tester needs a terminal. Run it in one, with `TERM` set (`export TERM=xterm-256color`) |
| `exec format error`, `cannot execute binary file`, `Bad CPU type in executable` | You ran `src/tester` directly on a machine that is not Linux x86-64. Call `tester_OS` |
| `permission denied` on `tester_OS` | `chmod +x tester_OS` |
| The tester cannot reach the server on a Linux with docker | `localhost` is the container: use the address of the machine, or `host.docker.internal` (`tester_OS` adds the name for you) |

## Resources

- The webserv subject, and the evaluation sheet: where the tester and the CGI program are asked for.
- `man uname`, `man 3 exec`
- `docker run`, the options `--platform` and `--add-host` - https://docs.docker.com/reference/cli/docker/container/run/
- termbox-go, the terminal library the tester is built with - https://github.com/nsf/termbox-go
