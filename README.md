### Minishell

A small, interactive shell written in C that implements a subset of bash behavior. It supports command parsing, pipelines, logical operators, redirections (including heredoc), wildcard expansion, environment variable expansion, a set of builtins, and both interactive and non-interactive modes.

### Features
- **Interactive prompt**: Custom prompt built from `ps_path(shell)` with username and current directory. Prints `exit` on EOF in interactive mode.
- **Execution**:
  - **External commands**: Search via `PATH` and execute.
  - **Pipelines `|`**: Left and right sides executed in separate processes with proper signal forwarding.
  - **Logical operators `&&` and `||`**: Short-circuit evaluation based on exit status.
  - **Subshells `( ... )`**: Parenthesized groups executed in a child process.
- **Redirections**: `<`, `>`, `>>`, `<<` (heredoc). Temporary heredoc files are created in the working directory as `.heredoc_*` and cleaned up on each loop iteration.
- **Quoting and expansion**:
  - Single and double quotes handling.
  - Environment variable expansion `$VAR` and last exit code `$?`.
  - Quote-aware tokenization and expansion before execution.
- **Wildcards**: `*` expansion to matching filesystem entries.
- **Builtins**: `echo`, `cd`, `pwd`, `export`, `unset`, `env`, `exit`, `history`.
- **Environment**:
  - Internal hash-table storage for variables; `SHLVL` managed per session.
  - Exit codes propagated consistently (including signals 130/131).
- **History**: Uses GNU Readline history; also persists parsed lines to an internal history fd for heredoc handling.
- **Signals**: Proper handling of `SIGINT` and `SIGQUIT` in interactive mode and during pipelines/subshells.

### Dependencies
- C compiler (tested with `cc`)
- GNU Readline (development headers)
- `make`

On Debian/Ubuntu:
```bash
sudo apt update && sudo apt install -y build-essential libreadline-dev
```

### Build
- **Binary name**: `minishell`
- From the project root:
```bash
make
```
Useful targets:
- `make clean` – remove object files
- `make fclean` – remove objects and binary
- `make re` – full rebuild

### Run
- Interactive mode:
```bash
./minishell
```
- Non-interactive mode (reads from stdin):
```bash
./minishell < script.sh
# or
cat script.sh | ./minishell
```
Note: Per design, `./minishell` must be run without arguments. Any arguments will cause it to exit with an error message.

### Usage examples
- Pipelines:
```bash
echo "hello" | tr a-z A-Z | rev
```
- Logical operators:
```bash
make && echo "build ok" || echo "build failed"
```
- Subshells:
```bash
( echo one && echo two ) | wc -l
```
- Redirections:
```bash
cat < input.txt > output.txt
echo line >> output.txt
cat << EOF
multi-line
heredoc
EOF
```
- Wildcards:
```bash
ls *.c
```
- Variables and quotes:
```bash
echo "$HOME"
echo last: $?
```
- Builtins:
```bash
cd /tmp
pwd
export FOO=bar
env | grep FOO
unset FOO
echo hello world
history
exit
```

### Project layout
- `Makefile` – build script linking against `-lreadline`
- `includes/` – public headers (`builtin.h`, `parser.h`, `syntax.h`, `token.h`, `types.h`, ...)
- `library/` – vendored libs:
  - `libft/` – utility library
  - `gnl/` – `get_next_line`
- `source/` – implementation:
  - `lexical_analysis/` – tokenization and readline glue
  - `syntax_analysis/` – syntax checks, quote and heredoc handling
  - `parser/` – AST construction (logic, redirections, parentheses)
  - `execution/` – AST execution, redirs, pipes, signals, wildcard
  - `builtin/` – builtins and dispatch
  - `environment/` – env table and initialization
  - `utils/` – helpers and memory management

### Behavior and notes
- Exit status of the shell is the status of the last executed command; signals map to 130 (`SIGINT`) and 131 (`SIGQUIT`).
- In interactive mode, `Ctrl-C` interrupts the current input/execution, `Ctrl-\` prints `Quit (core dumped)` if a child is terminated by `SIGQUIT`.
- Temporary heredoc files are cleaned at each prompt; names start with `.heredoc_` in the current directory.
- No job control (`fg`, `bg`, `jobs`) or advanced bash features.

### Valgrind
A suppression file for Readline is included: `readline_ignore.supp`.
Example invocation:
```bash
valgrind --track-origins=yes --leak-check=full --show-leak-kinds=all \
  --track-fds=yes --suppressions=readline_ignore.supp ./minishell
```

### License
This project is provided for educational purposes. Add a license of your choice if you plan to distribute it.
