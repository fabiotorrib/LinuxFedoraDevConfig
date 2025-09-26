# Fedora for devs: How to configure
Linux ( Fedora ) System Configuration for Software Engineers.
After installing Fedora, to set up drivers, basic apps, and other utilities, follow this tutorial:  
[GitHub - wz790/Fedora-Noble-Setup: Fedora Linux Noble Setup Guide (Post-Installation)](https://github.com/wz790/Fedora-Noble-Setup?tab=readme-ov-file)

To sign NVIDIA drivers without disabling Secure Boot, follow these steps:

```bash
# 1. Install tools
sudo dnf install -y mokutil openssl

# 2. Create keys (folder + generate certificate)
mkdir -p ~/nvidia-keys && cd ~/nvidia-keys
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der \
  -nodes -days 36500 -subj "/CN=My NVIDIA MOK/"
openssl x509 -in MOK.der -inform DER -out MOK.pem -outform PEM

# 3. Register key in firmware (it will ask for a password to confirm at boot)
sudo mokutil --import MOK.der
sudo reboot
# -> Blue MOK Manager screen: Enroll MOK > Continue > password > reboot

# 4. Sign NVIDIA modules (.ko)
modinfo -n nvidia   # shows the module path
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 \
  ~/nvidia-keys/MOK.priv ~/nvidia-keys/MOK.pem \
  /usr/lib/modules/$(uname -r)/extra/nvidia/*.ko

# 5. Regenerate initramfs so the signed module loads
sudo dracut --force
sudo reboot

# 6. Verificar se o driver foi carregado
dmesg | grep -i nvidia
nvidia-smi
```

---

---

## IMPORTANT PACKAGES

#### Compilation & Toolchain

These packages are the **build and compilation base** on Fedora. They include compilers, linkers, build systems, and packaging tools.

- **@development-tools** — Group with gcc, g++, make, etc.
  
- **cmake / ninja-build** — Modern build systems.
  
- **clang / clang-tools-extra** — C/C++ compiler and extra tools (clangd, clang-format).
  
- **pkg-config** — Manages compile/link flags.
  
- **bear** — Generates `compile_commands.json` for IDEs.
  
- **gdb** — Debugger for C/C++.
  
- **lcov** — Test coverage reports.
  
- **rpm-build / rpmdevtools** — Tools to build RPM packages.
  
- **rust** — Adds the Rust language to the system.
  
- **cargo** — Rust’s package manager.
  

```bash
sudo dnf install -y @development-tools cmake ninja-build clang clang-tools-extra pkg-config bear gdb lcov rpm-build rpmdevtools rust cargo
```

#### Testing & Code Quality

Frameworks and utilities that help **ensure code quality and stability**.

- **boost-devel** — Full Boost library headers.
  
- **gtest-devel / gmock-devel** — Google Test and Google Mock for C++.
  
- **shellcheck** — Analyzes shell scripts and flags common mistakes.
  

```bash
sudo dnf install -y boost-devel gtest-devel gmock-devel shellcheck
```

#### Debugging, Profiling & Diagnostics

Tools to **detect memory leaks, analyze performance, and monitor processes**.

- **valgrind** — Detects memory errors.
  
- **strace / ltrace** — Trace syscalls and library calls.
  
- **perf** — Performance profiler (via kernel-tools).
  
- **htop** — Interactive process monitor.
  

```bash
sudo dnf install -y valgrind strace ltrace perf htop
```

#### Graphics, Multimedia & 3D

Libraries for **games, simulations, and graphical apps**.

- **mesa-libGLES-devel / mesa-libEGL-devel** — OpenGL ES and EGL.
  
- **glm-devel** — Math for 3D graphics.
  
- **SDL2-devel / SDL2_image-devel** — SDL2 development.
  

```bash
sudo dnf install -y mesa-libGLES-devel mesa-libEGL-devel glm-devel SDL2-devel SDL2_image-devel
```

#### System, IPC & Services

APIs and libraries to **interact with Linux and its services**.

- **dbus-devel** — Inter-process communication.
  
- **systemd-devel** — Integration with systemd.
  
- **libcap-devel** — Fine-grained process capabilities.
  
- **expat-devel** — Fast XML parser.
  
- **libuuid-devel** — Generate and handle UUIDs.
  

```bash
sudo dnf install -y dbus-devel systemd-devel libcap-devel expat-devel libuuid-devel
```

#### Containers & Virtualization

Tools to **run isolated environments and virtual machines**.

- **lxc / lxc-devel** — Linux containers.
  
- **podman** — Rootless containers (Fedora native).
  
- **docker / docker-compose** — Classic containers (extra repo needed).
  
- **vagrant** — VM automation.
  
- **qemu-kvm / virt-manager** — Virtualization and GUI manager.
  

```bash
sudo dnf install -y lxc lxc-devel podman vagrant qemu-kvm virt-manager
```

#### Serialization & RPC

Libraries for **efficient data exchange between systems**.

- **protobuf-devel / protobuf-compiler** — Protocol Buffers.

```bash
sudo dnf install -y protobuf-devel protobuf-compiler
```

#### Networking & Web CLI

Tools to **test APIs, monitor networks, and handle cryptography**.

- **curl / wget** — Basic HTTP clients.
  
- **httpie** — Friendly HTTP client.
  
- **nmap** — Network scanner.
  
- **wireshark / wireshark-cli** — Packet sniffer.
  
- **net-tools / iproute** — Network utilities.
  
- **mosh** — Resilient SSH.
  
- **gnupg2** — Encryption and digital signatures.

- **nodejs** — Node.js. 
  

```bash
sudo dnf install -y curl wget httpie nmap wireshark wireshark-cli net-tools iproute mosh gnupg2 nodejs
```

#### Databases & Storage

Packages to **use and integrate databases in development**.

- **sqlite / sqlite-devel** — SQLite.
  
- **postgresql / postgresql-devel** — PostgreSQL client.
  
- **mariadb / mariadb-devel** — MySQL/MariaDB client.
  
- **redis / redis-devel** — Redis.
  

```bash
sudo dnf install -y sqlite sqlite-devel postgresql postgresql-devel mariadb mariadb-devel redis redis-devel
```

#### Python Development

Tools for **modern Python projects**.

- **python3-virtualenv / python3-venv** — Virtual environments.
  
- **ipython** — Interactive REPL.
  
- **jupyter-notebook / jupyterlab** — Scientific notebooks.
  
- **mypy / black / flake8 / isort** — Linters and formatters.
  
- **tk-devel** — Tkinter (GUIs).
  
- **bzip2-devel, gdbm-devel, libffi-devel, xz-devel, ncurses-devel, readline-devel, sqlite-devel, zlib-devel, openssl-devel** — C deps used by Python.
  

```bash
sudo dnf install -y python3-virtualenv python3-venv ipython jupyter-notebook jupyterlab mypy black flake8 isort tk-devel bzip2-devel gdbm-devel libffi-devel xz-devel ncurses-devel readline-devel sqlite-devel zlib-devel openssl-devel
```

#### Documentation & Diagrams

Tools to **generate documentation and diagrams automatically**.

- **doxygen** — C/C++ documentation.
  
- **graphviz** — Graph diagrams.
  
- **pandoc** — Document conversion.
  

```bash
sudo dnf install -y doxygen graphviz pandoc
```

#### Terminal Productivity

Packages that **make the terminal faster and more convenient**.

- **fzf** — Interactive fuzzy search.
  
- **ripgrep** — Modern, fast grep.
  
- **bat** — `cat` with syntax highlighting.
  
- **exa** — Improved `ls` with colors and tree.
  
- **unzip** — Extract ZIP archives.
  

```bash
sudo dnf install -y fzf ripgrep bat exa unzip
```

## One-liner with everything

> Installs all packages at once (except Docker, PlantUML, and MongoDB, which may require extra repositories).

```bash
sudo dnf install -y --skip-unavailable development-tools cmake ninja-build clang clang-tools-extra pkg-config bear gdb lcov rpm-build rpmdevtools rust cargo boost-devel gtest-devel gmock-devel shellcheck valgrind strace ltrace perf htop mesa-libGLES-devel mesa-libEGL-devel glm-devel SDL2-devel SDL2_image-devel xorg-x11-utils dbus-devel systemd-devel libcap-devel expat-devel libuuid-devel lxc lxc-devel podman vagrant qemu-kvm virt-manager protobuf-devel protobuf-compiler curl wget httpie nmap wireshark wireshark-cli net-tools iproute mosh gnupg2 sqlite sqlite-devel postgresql postgresql-devel mariadb mariadb-devel redis redis-devel python3-virtualenv python3-venv ipython jupyter-notebook jupyterlab mypy black flake8 isort tk-devel bzip2-devel gdbm-devel libffi-devel xz-devel ncurses-devel readline-devel sqlite-devel zlib-devel openssl-devel doxygen graphviz pandoc fzf ripgrep bat exa unzip nodejs
```

---

## Anaconda, VSCode, uv and VTM

This script installs three popular tools on Fedora: **Anaconda**, which provides a complete Python environment with data-science libraries; **VTM**, a terminal manager that can be installed via Cargo (requires Rust installed beforehand); and **Visual Studio Code**, Microsoft’s modern code editor.

```bash
# Anaconda
cd ~ && wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh
bash anaconda.sh -b -p $HOME/anaconda3
echo 'export PATH="$HOME/anaconda3/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc

# VTM (needs rust installed)
cargo install vtm
# uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# VS Code
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo dnf install -y https://go.microsoft.com/fwlink/?LinkID=760867
```

---

## Python Libraries

```bash
pip3 install jedi black ruff ipython --user
```

---

## VSCode Plugins

#### Appearance & Themes

- **pkief.material-icon-theme** → Material-style file icons.
  
- **naumovs.color-highlight** → Highlights colors (#fff, rgb, etc.).
  

### Productivity & Utilities

- **aaron-bond.better-comments** → Colorful, highlighted comments.
  
- **alefragnani.bookmarks** → Bookmarks in code.
  
- **gruntfuggly.activitusbar** → Customize the activity bar.
  
- **sleistner.vscode-fileutils** → Fast file operations.
  
- **roscop.activefileinstatusbar** → Shows active file in status bar.
  
- **nick-rudenko.back-n-forth** → Quick navigation between edit points.
  
- **sergeyegorov.folder-color** → Change folder colors in Explorer.
  
- **mgesbert.indent-nested-dictionary** → Better indentation for dictionaries.
  
- **exodiusstudios.comment-anchors** → Visual anchors in comments.
  
- **cliffordfajardo.highlight-line-vscode** → Highlight current line.
  
- **ryu1kn.text-marker** → Mark words/snippets in editor.
  
- **stkb.rewrap** → Smart wrapping for comments/text.
  
- **shardulm94.trailing-spaces** → Highlight/remove trailing spaces.
  
- **timonwong.shellcheck** → ShellCheck integration (bash lint).
  
- **tyriar.sort-lines** → Sort lines quickly.
  
- **mechatroner.rainbow-csv** → Colorful CSV viewing.
  
- **bierner.markdown-preview-github-styles** → GitHub-style Markdown preview.
  
- **tomoki1207.pdf** → View PDFs in VS Code.
  
- **yutengjing.open-in-external-app** → Open files in external apps.
  
- **yutengjing.vscode-archive** → Open ZIP files directly.
  
- **foxundermoon.shell-format** → Shell script formatter.
  
- **sleistner.vscode-fileutils** → Quick file ops.
  
- **christian-kohler.path-intellisense** → Autocomplete file names/paths.
  
- **usernamehw.errorlens** → Show errors/warnings inline without the Problems panel.
  

### Python

- **ms-python.python** → Main Python support.
  
- **ms-python.debugpy** → Python debugging.
  
- **charliermarsh.ruff** → Fast linter/tooling for Python.
  
- **franneck94.vscode-cpython-extension-pack** → Extension pack for C/Python.
  
- **benjamin-simmonds.pythoncpp-debug** → Hybrid Python/C++ debugging.
  
- **ms-python.vscode-pylance** → Better static analysis & IntelliSense for Python.
  
- **ms-toolsai.jupyter** → Jupyter notebooks support in VS Code.
  
- **ms-toolsai.jupyter-keymap** → Jupyter-style keybindings.
  
- **ms-toolsai.jupyter-renderers** → Better visualization in notebooks.
  

### Cython

- **guyskk.language-cython** → Cython syntax.
  
- **ktnrg45.vscode-cython** → Another Cython support extension.
  
- **tcwalther.cython** → Extra Cython extension.
  

### C / C++ / Rust

- **ms-vscode.cpptools** → IntelliSense, debug, CMake, etc.
  
- **llvm-vs-code-extensions.vscode-clangd** → Clangd support.
  
- **cmstead.js-codeformer** → JS refactoring (also handy in C++ workflows).
  

### Java

- **vscjava.vscode-java-pack** → Complete Java pack.
  
- **vscjava.vscode-java-dependency** → Dependency management for Java.
  
- **vscjava.vscode-java-debug** → Java debugging.
  
- **vscjava.vscode-maven** → Maven support.
  

### Databases & Web Tools

- **qwtel.sqlite-viewer** → View SQLite databases.
  
- **mtxr.sqltools** → Integrated SQL client for PostgreSQL, MySQL, SQLite, etc.
  
- **ritwickdey.liveserver** → Local server for live HTML/CSS/JS preview.
  
- **esbenp.prettier-vscode** → Formatter for JS/TS/HTML/CSS.
  

### Terminals & Shell

- **adrianwilczynski.terminal-commands** → Manage terminal commands.
  
- **antiantisepticeye.vscode-color-picker** → Color picker.
  
- **artdiniz.quitcontrol-vscode** → Quick close control.
  
- **foxundermoon.shell-format** → Shell script formatter.
  
- **mads-hartmann.bash-ide-vscode** → Bash IDE.
  

### AI & Automation

- **codeium.codeium** → AI autocompletion.

### Git, Diff & Collaboration

- **jinsihou.diff-tool** → Compare files.
  
- **l13rary.l13-diff** → Another diff tool.
  
- **vsls-contrib.gistfs** → Open GitHub Gists as a filesystem.
  
- **github.codespaces** → GitHub Codespaces integration.
  
- **eamodio.gitlens** → Advanced Git tool (blame, history, compare).
  

### Extras

- **tyriar.luna-paint** → Pixel art editor.
  
- **tldraw-org.tldraw-vscode** → tldraw (diagrams) integration.
  
- **tomoki1207.vscode-input-sequence** → Generate input sequences.
  
- **guiextensions.tosingleline** → Join multiple lines into one.
  
- **wscats.command-runner** → Run preconfigured commands.
  
- **cmstead.js-codeformer** → JS refactoring.
  

### Installing plugins

```bash
code --install-extension pkief.material-icon-theme \
--install-extension naumovs.color-highlight \
--install-extension aaron-bond.better-comments \
--install-extension alefragnani.bookmarks \
--install-extension gruntfuggly.activitusbar \
--install-extension sleistner.vscode-fileutils \
--install-extension roscop.activefileinstatusbar \
--install-extension nick-rudenko.back-n-forth \
--install-extension sergeyegorov.folder-color \
--install-extension mgesbert.indent-nested-dictionary \
--install-extension exodiusstudios.comment-anchors \
--install-extension cliffordfajardo.highlight-line-vscode \
--install-extension ryu1kn.text-marker \
--install-extension stkb.rewrap \
--install-extension shardulm94.trailing-spaces \
--install-extension timonwong.shellcheck \
--install-extension tyriar.sort-lines \
--install-extension mechatroner.rainbow-csv \
--install-extension bierner.markdown-preview-github-styles \
--install-extension tomoki1207.pdf \
--install-extension yutengjing.open-in-external-app \
--install-extension yutengjing.vscode-archive \
--install-extension foxundermoon.shell-format \
--install-extension christian-kohler.path-intellisense \
--install-extension usernamehw.errorlens \
--install-extension ms-python.python \
--install-extension ms-python.debugpy \
--install-extension charliermarsh.ruff \
--install-extension franneck94.vscode-cpython-extension-pack \
--install-extension benjamin-simmonds.pythoncpp-debug \
--install-extension ms-python.vscode-pylance \
--install-extension ms-toolsai.jupyter \
--install-extension ms-toolsai.jupyter-keymap \
--install-extension ms-toolsai.jupyter-renderers \
--install-extension guyskk.language-cython \
--install-extension ktnrg45.vscode-cython \
--install-extension tcwalther.cython \
--install-extension ms-vscode.cpptools \
--install-extension llvm-vs-code-extensions.vscode-clangd \
--install-extension cmstead.js-codeformer \
--install-extension vscjava.vscode-java-pack \
--install-extension vscjava.vscode-java-dependency \
--install-extension vscjava.vscode-java-debug \
--install-extension vscjava.vscode-maven \
--install-extension qwtel.sqlite-viewer \
--install-extension mtxr.sqltools \
--install-extension ritwickdey.liveserver \
--install-extension esbenp.prettier-vscode \
--install-extension adrianwilczynski.terminal-commands \
--install-extension antiantisepticeye.vscode-color-picker \
--install-extension artdiniz.quitcontrol-vscode \
--install-extension mads-hartmann.bash-ide-vscode \
--install-extension codeium.codeium \
--install-extension jinsihou.diff-tool \
--install-extension l13rary.l13-diff \
--install-extension vsls-contrib.gistfs \
--install-extension github.codespaces \
--install-extension eamodio.gitlens \
--install-extension tyriar.luna-paint \
--install-extension tldraw-org.tldraw-vscode \
--install-extension tomoki1207.vscode-input-sequence \
--install-extension guiextensions.tosingleline \
--install-extension wscats.command-runner
```

### VSCode config JSON

```json
{
    "code-runner.runInTerminal": true,
    "editor.autoClosingBrackets": "always",
    "editor.bracketPairColorization.independentColorPoolPerBracketType": true,
    "cmake.showOptionsMovedNotification": false,
    "workbench.startupEditor": "none",
    "terminal.explorerKind": "external",
    "python.terminal.activateEnvironment": false,
    "terminal.integrated.gpuAcceleration": "on",
    "terminal.integrated.scrollback": 1000000,
    "accessibility.verbosity.terminal": false,
    "accessibility.dimUnfocused.opacity": 1,
    "terminal.integrated.accessibleViewFocusOnCommandExecution": true,
    "terminal.integrated.accessibleViewPreserveCursorPosition": true,
    "terminal.external.windowsExec": "/bin/bash",
    "editor.accessibilityPageSize": 1,
    "editor.accessibilitySupport": "off",
    "editor.definitionLinkOpensInPeek": true,
    "editor.emptySelectionClipboard": false,
    "editor.experimentalWhitespaceRendering": "off",
    "editor.fastScrollSensitivity": 4,
    "editor.foldingMaximumRegions": 500,
    "editor.gotoLocation.alternativeTypeDefinitionCommand": "editor.action.goToImplementation",
    "editor.gotoLocation.multipleDeclarations": "gotoAndPeek",
    "editor.guides.bracketPairs": true,
    "editor.inlayHints.padding": true,
    "editor.mouseWheelScrollSensitivity": 1.1,
    "editor.multiCursorLimit": 50000,
    "editor.padding.bottom": 1,
    "editor.padding.top": 1,
    "editor.tabCompletion": "on",
    "editor.stickyTabStops": true,
    "editor.unfoldOnClickAfterEndOfLine": true,
    "editor.unicodeHighlight.nonBasicASCII": false,
    "editor.wrappingIndent": "indent",
    "editor.fontWeight": "normal",
    "editor.formatOnPaste": true,
    "diffEditor.experimental.showMoves": true,
    "diffEditor.hideUnchangedRegions.enabled": true,
    "diffEditor.hideUnchangedRegions.minimumLineCount": 1,
    "diffEditor.maxComputationTime": 0,
    "diffEditor.maxFileSize": 0,
    "multiDiffEditor.experimental.enabled": true,
    "editor.minimap.scale": 1,
    "editor.inlineSuggest.showToolbar": "always",
    "editor.quickSuggestions": {
        "other": "on",
        "comments": "on",
        "strings": "on"
    },
    "editor.quickSuggestionsDelay": 3,
    "editor.screenReaderAnnounceInlineSuggestion": false,
    "editor.suggest.filterGraceful": false,
    "editor.suggest.matchOnWordStartOnly": false,
    "editor.suggest.preview": true,
    "editor.suggest.shareSuggestSelections": true,
    "editor.suggestSelection": "first",
    "files.autoGuessEncoding": true,
    "files.defaultLanguage": "Python",
    "files.hotExit": "off",
    "files.participants.timeout": 0,
    "files.readonlyFromPermissions": true,
    "files.simpleDialog.enable": true,
    "workbench.commandPalette.experimental.suggestCommands": true,
    "workbench.commandPalette.history": 50000,
    "workbench.commandPalette.preserveInput": true,
    "workbench.enableExperiments": false,
    "workbench.experimental.share.enabled": true,
    "workbench.list.fastScrollSensitivity": 6,
    "workbench.list.scrollByPage": true,
    "workbench.list.smoothScrolling": true,
    "workbench.localHistory.maxFileEntries": 5000,
    "workbench.localHistory.maxFileSize": 8192,
    "workbench.reduceMotion": "on",
    "workbench.tips.enabled": false,
    "workbench.tree.enableStickyScroll": true,
    "workbench.editor.autoLockGroups": {
        "terminalEditor": false
    },
    "workbench.editor.limit.enabled": false,
    "workbench.settings.enableNaturalLanguageSearch": false,
    "workbench.editor.wrapTabs": true,
    "workbench.editor.limit.value": 10000,
    "workbench.editor.pinnedTabsOnSeparateRow": true,
    "workbench.editor.sharedViewState": true,
    "window.enableMenuBarMnemonics": false,
    "window.restoreFullscreen": true,
    "accessibility.dimUnfocused.enabled": true,
    "search.searchEditor.reusePriorSearchConfiguration": true,
    "search.seedWithNearestWord": true,
    "search.showLineNumbers": true,
    "search.smartCase": true,
    "search.useGlobalIgnoreFiles": true,
    "debug.allowBreakpointsEverywhere": true,
    "debug.autoExpandLazyVariables": "on",
    "debug.console.closeOnEnd": true,
    "debug.inlineValues": "on",
    "scm.alwaysShowRepositories": true,
    "scm.inputMaxLines": 50,
    "terminal.integrated.allowMnemonics": false,
    "terminal.integrated.autoReplies": {
        "Terminate batch job (Y/N)": "Y\r"
    },
    "terminal.integrated.confirmOnKill": "never",
    "terminal.integrated.copyOnSelection": false,
    "terminal.integrated.cursorBlinking": true,
    "terminal.integrated.cursorStyleInactive": "block",
    "terminal.integrated.defaultLocation": "editor",
    "terminal.integrated.defaultProfile.windows": "Command Prompt",
    "terminal.integrated.ignoreBracketedPasteMode": true,
    "terminal.integrated.windowsEnableConpty": true,
    "accessibility.hideAccessibleView": true,
    "security.workspace.trust.untrustedFiles": "open",
    "pieces.setCopilotLocation": true,

    // ===== Python =====
    "python.languageServer": "Jedi",
    "python.defaultInterpreterPath": "/usr/bin/python3",
    "python.analysis.userFileIndexingLimit": -1,
    "python.analysis.autoImportCompletions": true,
    "python.analysis.enablePytestSupport": false,
    "python.analysis.gotoDefinitionInStringLiteral": true,
    "python.analysis.completeFunctionParens": true,
    "python.analysis.inlayHints.variableTypes": false,
    "python.analysis.inlayHints.pytestParameters": false,
    "python.analysis.inlayHints.functionReturnTypes": false,
    "python.analysis.persistAllIndices": false,
    "python.analysis.logLevel": "Error",
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true
    },

    // ===== Ruff =====
    "ruff.organizeImports": true,

    // ===== JavaScript / TypeScript =====
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.formatOnSave": true
    },
    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.formatOnSave": true
    },
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit"
    },
    "prettier.useEditorConfig": true,

    // ===== C/C++ / Clangd =====
    "C_Cpp.intelliSenseEngine": "disabled",
    "clangd.checkUpdates": true,
    "clangd.path": "/usr/bin/clangd",
    "clangd.arguments": [
        "--background-index",
        "--clang-tidy",
        "--header-insertion=never",
        "--completion-style=detailed"
    ],

    // ===== Error Lens =====
    "errorLens.enabled": true,
    "errorLens.fontStyleItalic": true,
    "errorLens.followCursor": "closestProblem",
    "errorLens.margin": "100",

    // ===== GitLens =====
    "gitlens.blame.inline.enabled": true,
    "gitlens.currentLine.enabled": true,
    "gitlens.hovers.currentLine.over": "line",
    "gitlens.hovers.annotations.enabled": false,

    // ===== SQLTools =====
    "sqltools.useNodeRuntime": true,
    "sqltools.results.saveAs": "csv",

    // ===== Diversos =====
    "bookmarks.useWorkaroundForFormatters": true,
    "bookmarks.keepBookmarksOnLineDelete": true,
    "bookmarks.sideBar.expanded": true,
    "bashIde.shellcheckPath": "/usr/bin/shellcheck",
    "codeium.enableConfig": {
        "*": true
    },
    "checkpoints.addCheckpointOnSave": true,
    "checkpoints.locale": "pt-BR",
    "rewrap.wrappingColumn": 72,
    "vscode-color-picker.languages": ["python", "javascript", "typescript", "C"],
    "material-icon-theme.files.color": "#26a69a",
    "workbench.iconTheme": "material-icon-theme",
    "workbench.activityBar.location": "top",
    "workbench.editorAssociations": {
        "*.jpg": "luna.editor"
    },
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.mouseWheelZoom": true,
    "window.zoomLevel": -1,
    "debug.internalConsoleOptions": "neverOpen",
    "clipboard-manager.snippet.enabled": false,
    "codeium.aggressiveShutdown": true,
    "shellcheck.exclude": ["SC3054", "SC3030"],
    "files.eol": "\n",
    "workbench.editor.highlightModifiedTabs": true,
    "luna.retainContextWhenHidden": false,
    "prettier.resolveGlobalModules": true,
    "git.openRepositoryInParentFolders": "never",
    "search.quickAccess.preserveInput": true,
    "window.menuBarVisibility": "compact",
    "files.autoSave": "afterDelay",

    // ===== Exclude (Performance) =====
    "files.watcherExclude": {
        "**/.git/**": true,
        "**/node_modules/**": true,
        "**/.venv/**": true,
        "**/__pycache__/**": true
    },
    "search.exclude": {
        "**/node_modules": true,
        "**/.venv": true,
        "**/.git": true,
        "**/__pycache__": true
    }
}
```

---

## ENV Launcher

An interactive script that centralizes Python environment management on Linux. It lets you create new environments with **Conda** or **uv**, list existing ones, and open any of them directly in **VS Code** with the interpreter set automatically.

#### ~/condalinux.sh

```bash
vim ~/condalinux.sh
```

#### content of ~/condalinux.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

# ========== Config ==========
CONDA_ENVS_DIR="${HOME}/anaconda3/envs"
UV_ENVS_DIR="${HOME}/uv-envs"
CODE_USER_SETTINGS="${HOME}/.config/Code/User/settings.json"
PY_GLOBAL="$(command -v python3 || true)"

# ========== Utils ==========
have() { command -v "$1" >/dev/null 2>&1; }
ensure_dir() { [ -d "$1" ] || mkdir -p "$1"; }
msg(){ printf "\n\033[1;36m%s\033[0m\n" "$*"; }
err(){ printf "\n\033[1;31m[ERROR]\033[0m %s\n" "$*" >&2; }

# VS Code outside this script -> always use global Python
set_vscode_global_python() {
  [ -n "${PY_GLOBAL}" ] || return 0
  ensure_dir "$(dirname "$CODE_USER_SETTINGS")"
  if have jq && [ -f "$CODE_USER_SETTINGS" ]; then
    tmp="$(mktemp)"; jq --arg p "$PY_GLOBAL" '. + {"python.defaultInterpreterPath": $p}' "$CODE_USER_SETTINGS" > "$tmp" || true; mv "$tmp" "$CODE_USER_SETTINGS"
  elif have jq; then
    jq -n --arg p "$PY_GLOBAL" '{ "python.defaultInterpreterPath": $p }' > "$CODE_USER_SETTINGS"
  else
    # simple fallback (does not validate existing JSON)
    printf '{\n  "python.defaultInterpreterPath": "%s"\n}\n' "$PY_GLOBAL" > "$CODE_USER_SETTINGS"
  fi
}

# Create pyexe runner INSIDE the env (like your original script), launching IPython via VTM
make_env_pyexe() {
  local env_dir="$1" env_name="$2" pybin="$3"
  local logfile="${env_dir}/ipython.log"
  cat >"${env_dir}/pyexe" <<EOF
#!/bin/sh
# IPython runner inside VTM (env: ${env_name})
export DONT_PROMPT_WSL_INSTALL=1
touch _____tmp.py 2>/dev/null
vtm -r term bash -lc '"${pybin}" -m ipython -i _____tmp.py \
  --colors=Linux \
  --Completer.use_jedi=True \
  --Completer.greedy=True \
  --Completer.suppress_competing_matchers=True \
  --Completer.limit_to__all__=False \
  --Completer.jedi_compute_type_timeout=10000 \
  --HistoryManager.hist_file="${HOME}/ipython_hist.sqlite" \
  --HistoryManager.db_cache_size=0 \
  --TerminalInteractiveShell.xmode=Verbose \
  --TerminalInteractiveShell.space_for_menu=40 \
  --TerminalInteractiveShell.history_load_length=10000 \
  --TerminalInteractiveShell.history_length=100000 \
  --InteractiveShell.history_load_length=10000 \
  --InteractiveShell.history_length=100000 \
  --logappend="${logfile}" --logfile="${logfile}"'
EOF
  chmod +x "${env_dir}/pyexe"
}

# Ensure ipython is available in the environment
ensure_ipython() {
  local pybin="$1"
  "$pybin" - <<'PY' || "$pybin" -m pip install -q --upgrade ipython >/dev/null
import importlib, sys
sys.exit(0 if importlib.util.find_spec("IPython") else 1)
PY
}

# Open VS Code in the env with interpreter pinned at the workspace level
fix_workspace_interpreter_and_open_code() {
  local env_dir="$1" pybin="$2"
  local vsdir="${env_dir}/.vscode"; local vssettings="${vsdir}/settings.json"
  ensure_dir "$vsdir"
  if have jq && [ -f "$vssettings" ]; then
    tmp="$(mktemp)"; jq --arg p "$pybin" '. + {"python.defaultInterpreterPath": $p}' "$vssettings" > "$tmp" || true; mv "$tmp" "$vssettings"
  else
    printf '{\n  "python.defaultInterpreterPath": "%s"\n}\n' "$pybin" > "$vssettings"
  fi
  code "$env_dir" >/dev/null 2>&1 &
}

# ========== Flows ==========

# 0) GLOBAL Python in VTM (no env)
open_global_ipython_vtm() {
  have vtm || { err "vtm not found (install with 'cargo install vtm' or your distro package)"; exit 1; }
  [ -n "$PY_GLOBAL" ] || { err "global python3 not found"; exit 1; }
  ensure_ipython "$PY_GLOBAL"
  msg "Opening GLOBAL IPython in VTM..."
  vtm -r term bash -lc "'$PY_GLOBAL' -m ipython -i --colors=Linux \
    --HistoryManager.hist_file='${HOME}/ipython_hist.sqlite' \
    --HistoryManager.db_cache_size=0 \
    --TerminalInteractiveShell.xmode=Verbose \
    --TerminalInteractiveShell.space_for_menu=40 \
    --TerminalInteractiveShell.history_load_length=10000 \
    --TerminalInteractiveShell.history_length=100000 \
    --InteractiveShell.history_load_length=10000 \
    --InteractiveShell.history_length=100000"
}

# 1) Create venv with uv (with pyexe and VS Code)
create_uv_env() {
  have uv || { err "uv not installed. E.g.: curl -LsSf https://astral.sh/uv/install.sh | sh"; exit 1; }
  ensure_dir "$UV_ENVS_DIR"
  read -rp "Env name (uv): " name; [ -n "$name" ] || { err "Invalid name."; exit 1; }
  read -rp "Python version (e.g., 3.11) [empty = default]: " pyver || true
  local env_dir="${UV_ENVS_DIR}/${name}"
  [ -d "$env_dir" ] && { err "Already exists: ${env_dir}"; exit 1; }
  msg "Creating uv env '${name}'..."
  if [ -n "${pyver:-}" ]; then uv venv --python "python${pyver}" "$env_dir"; else uv venv "$env_dir"; fi
  local pybin="${env_dir}/bin/python"
  "$pybin" -m ensurepip --upgrade >/dev/null 2>&1 || true
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "uv:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Use: ${env_dir}/pyexe  # to open IPython in VTM for this env"
}

# 2) Create env with conda (with conda init/activate, pyexe and VS Code)
create_conda_env() {
  have conda || { err "conda not found. Install Miniconda/Anaconda and run 'conda init'."; exit 1; }
  read -rp "Env name (conda): " name; [ -n "$name" ] || { err "Invalid name."; exit 1; }
  read -rp "Python version (e.g., 3.11): " pyver; [ -n "$pyver" ] || { err "Provide a Python version."; exit 1; }
  msg "Creating conda env '${name}'..."
  conda create -y -n "$name" "python=${pyver}" ipython >/dev/null
  local env_dir="${CONDA_ENVS_DIR}/${name}" pybin="${env_dir}/bin/python"
  # Keep conda init/activate behavior from your original flow
  conda init >/dev/null 2>&1 || true
  # Create pyexe and open Code
  make_env_pyexe "$env_dir" "conda:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Use: ${env_dir}/pyexe  # to open IPython in VTM for this env"
}

# Enter existing Conda env (with conda init/activate, pyexe and VS Code)
enter_conda_env() {
  local name="$1"
  local env_dir="${CONDA_ENVS_DIR}/${name}"
  local pybin="${env_dir}/bin/python"
  [ -x "$pybin" ] || { err "Python for conda env '${name}' not found at ${pybin}"; exit 1; }
  # Preserve your behavior: conda init and (if desired) conda activate in the VTM shell
  conda init >/dev/null 2>&1 || true
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "conda:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  # Besides opening Code, also open IPython now:
  msg "Opening IPython in VTM for conda env '${name}'..."
  "${env_dir}/pyexe"
}

# Enter existing uv env (with pyexe and VS Code)
enter_uv_env() {
  local name="$1"
  local env_dir="${UV_ENVS_DIR}/${name}"
  local pybin="${env_dir}/bin/python"
  [ -x "$pybin" ] || { err "Python for uv env '${name}' not found at ${pybin}"; exit 1; }
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "uv:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Opening IPython in VTM for uv env '${name}'..."
  "${env_dir}/pyexe"
}

# ========== Menu ==========
main_menu() {
  set_vscode_global_python

  local conda_envs=() uv_envs=()
  if [ -d "$CONDA_ENVS_DIR" ]; then
    while IFS= read -r d; do conda_envs+=("$(basename "$d")"); done < <(find "$CONDA_ENVS_DIR" -maxdepth 1 -mindepth 1 -type d | sort)
  fi
  ensure_dir "$UV_ENVS_DIR"
  while IFS= read -r d; do uv_envs+=("$(basename "$d")"); done < <(find "$UV_ENVS_DIR" -maxdepth 1 -mindepth 1 -type d | sort)

  echo
  echo "==================== CondaLinux Menu ===================="
  echo "0) Open GLOBAL Python (outside envs) in IPython (via VTM)"
  echo "1) Create venv with uv"
  echo "2) Create env with conda"
  echo "---------------------------------------------------------"
  local idx=3
  if ((${#conda_envs[@]})); then
    echo "Conda envs:"
    for name in "${conda_envs[@]}"; do printf "%d) %s\n" "$idx" "$name"; idx=$((idx+1)); done
  fi
  if ((${#uv_envs[@]})); then
    echo "uv envs:"
    for name in "${uv_envs[@]}"; do printf "%d) %s\n" "$idx" "$name"; idx=$((idx+1)); done
  fi
  echo "========================================================="
  read -rp "Choice: " choice
  [[ "$choice" =~ ^[0-9]+$ ]] || { err "Invalid choice."; exit 1; }

  case "$choice" in
    0) open_global_ipython_vtm ;;
    1) create_uv_env ;;
    2) create_conda_env ;;
    *)
      local conda_end=$((2 + ${#conda_envs[@]}))
      if (( choice >= 3 && choice <= conda_end )); then
        local pos=$((choice - 3))
        enter_conda_env "${conda_envs[$pos]}"
        exit 0
      fi
      local uv_pos=$((choice - conda_end - 1))
      if (( uv_pos >= 0 && uv_pos < ${#uv_envs[@]} )); then
        enter_uv_env "${uv_envs[$uv_pos]}"
        exit 0
      fi
      err "Option out of range."
      exit 1
      ;;
  esac
}

main_menu
```

Grant execute permission

```bash
chmod +x ~/condalinux.sh
```
