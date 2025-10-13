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
sudo dnf install -y --skip-unavailable development-tools cmake ninja-build clang clang-tools-extra pkg-config bear gdb lcov rpm-build rpmdevtools rust cargo boost-devel gtest-devel gmock-devel shellcheck valgrind strace ltrace perf htop mesa-libGLES-devel mesa-libEGL-devel glm-devel SDL2-devel SDL2_image-devel xorg-x11-utils dbus-devel systemd-devel libcap-devel expat-devel libuuid-devel lxc lxc-devel podman vagrant qemu-kvm virt-manager protobuf-devel protobuf-compiler curl wget httpie nmap wireshark wireshark-cli net-tools iproute mosh gnupg2 sqlite sqlite-devel postgresql postgresql-devel mariadb mariadb-devel redis redis-devel python3-virtualenv python3-venv ipython jupyter-notebook jupyterlab mypy black flake8 isort tk-devel bzip2-devel gdbm-devel libffi-devel xz-devel ncurses-devel readline-devel sqlite-devel zlib-devel openssl-devel doxygen graphviz pandoc fzf ripgrep bat exa unzip nodejs git
```

---

## Anaconda, VSCode, VTM

This script installs three popular tools on Fedora: **Anaconda**, which provides a complete Python environment with data-science libraries; **VTM**, a terminal manager; and **Visual Studio Code**, Microsoft’s modern code editor.

```bash
# Anaconda
cd ~ && wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh
bash anaconda.sh -b -p $HOME/anaconda3
echo 'export PATH="$HOME/anaconda3/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc

# VTM (needs rust installed)
vtm_link="https://github.com/directvt/vtm/releases/download/v0.9.99.70/vtm_linux_x86_64.zip"
vtm_file="$HOME/vtm_linux_x86_64.zip"
wget "$vtm_link" -O "$vtm_file"

unzip "$vtm_file"
tar -xf vtm_linux_x86_64.tar
sudo ./vtm --install

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

An interactive script that centralizes Python environment management on Linux. It lets you create new environments with **Conda**, list existing ones, and open any of them directly in **VS Code** with the interpreter set automatically.

#### ~/condalinux.sh

```bash
vim ~/condalinux.sh
```

#### content of ~/condalinux.sh

```bash
#!/bin/bash
set -euo pipefail

folderPath="$HOME/anaconda3/envs"

# Lista envs como nomes simples (sem caminho)
mapfile -t envs < <(find "$folderPath" -maxdepth 1 -mindepth 1 -type d -printf "%f\n" | sort)

# Construção do menu
echo "0. Create new env"
i=1
for name in "${envs[@]}"; do
  echo "$i. $name"
  ((i++))
done
max_index=$(( ${#envs[@]} ))  # último índice de escolha

# Lê escolha
choice=""
while [[ -z "${choice:-}" ]]; do
  read -rp "Enter your choice: " input
  if [[ "$input" =~ ^[0-9]+$ && "$input" -le "$max_index" ]]; then
    choice="$input"
  fi
done

# Função: carregar hook do conda neste shell
ensure_conda_hook() {
  if ! command -v conda >/dev/null 2>&1; then
    if [ -f "$HOME/anaconda3/etc/profile.d/conda.sh" ]; then
      . "$HOME/anaconda3/etc/profile.d/conda.sh"
    else
      eval "$($HOME/anaconda3/bin/conda shell.bash hook)"
    fi
  else
    eval "$(conda shell.bash hook)"
  fi
}

if [[ "$choice" -eq 0 ]]; then
  read -rp "Name of new env: " envname
  read -rp "Python version (ex: 3.11): " pyversion
  read -rp "Packages to install (space-separated): " packages

  ensure_conda_hook
  # Não é preciso ajustar canais toda vez; faça isso uma vez fora do script, se quiser.
  conda create -y -n "$envname" "python=$pyversion" pip ipython $packages

  env_dir="$folderPath/$envname"
  logfile="${env_dir}/ipythonlog.log"

  cat >"${env_dir}/pyexe" <<EOF
#!/bin/sh
# executa ipython dentro do env sem precisar ativar
conda run -n "$envname" ipython -i _____tmp.py -c="run \$1" \
  --Completer.use_jedi=True \
  --Completer.greedy=True \
  --Completer.suppress_competing_matchers=True \
  --Completer.limit_to__all__=False \
  --Completer.jedi_compute_type_timeout=10000 \
  --Completer.evaluation='dangerous' \
  --Completer.auto_close_dict_keys=True \
  --PlainTextFormatter.max_width=9999 \
  --HistoryManager.hist_file="\$HOME/ipython_hist.sqlite" \
  --HistoryManager.db_cache_size=0 \
  --TerminalInteractiveShell.xmode='Verbose' \
  --TerminalInteractiveShell.space_for_menu=20 \
  --TerminalInteractiveShell.history_load_length=10000 \
  --TerminalInteractiveShell.history_length=100000 \
  --TerminalInteractiveShell.display_page=True \
  --TerminalInteractiveShell.autoformatter='black' \
  --TerminalInteractiveShell.auto_match=True \
  --logappend="$logfile" \
  --logfile="$logfile" \
  --InteractiveShell.history_load_length=10000 \
  --InteractiveShell.history_length=100000 \
  --cache-size=100000 \
  --BaseIPythonApplication.log_level=30 \
  --colors=Linux
EOF
  chmod +x "${env_dir}/pyexe"
  echo "Env '$envname' criado. Executável helper: ${env_dir}/pyexe"

else
  # Escolha de um env existente
  selected="${envs[$((choice-1))]}"
  echo "You chose: $selected"

  # a) Ativar no shell ATUAL (se rodar com 'source condalinux.sh')
  ensure_conda_hook
  conda activate "$selected"

  # b) Abrir VSCode usando o Python do env sem ativar (opcional):
  # conda run -n "$selected" code "$folderPath/$selected"

  # c) Abrir novo terminal com o env ativo (opcional):
  # gnome-terminal -- bash -ic "eval \"\$(conda shell.bash hook)\"; conda activate \"$selected\"; exec bash"
fi

```

Grant execute permission

```bash
chmod +x ~/condalinux.sh
```

After open shell in VSCode, remember to activate conda venv with

```bash
activate conda <env_name>
```

To acess interative mode in python, run the file with 

```bash
. pyexe file_name.py
```
###Configuring Terminal

1. Install Kitty (terminal emulator)
sudo dnf install kitty


Set Kitty as default terminal (KDE Settings → Default Applications → Terminal).
Or via command:

sudo update-alternatives --config x-terminal-emulator

2. Install Zsh
sudo dnf install zsh
chsh -s $(which zsh)   # make zsh default


If chsh doesn’t work, add this to ~/.bash_profile:

if [ -t 1 ] && [ -x /usr/bin/zsh ]; then
  exec /usr/bin/zsh -l
fi


Reboot or log out/in.

3. Install Oh My Zsh
sudo dnf install git curl -y
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

4. Add fzf
sudo dnf install fzf
echo '[ -f /usr/share/fzf/shell/key-bindings.zsh ] && source /usr/share/fzf/shell/key-bindings.zsh' >> ~/.zshrc
autoload -Uz compinit && compinit

5. Add fzf-tab (suggestions in a navegable pop-up)
git clone https://github.com/Aloxaf/fzf-tab ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/fzf-tab

6. Add highlighting (red color word means incorrect command)
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

7. Enable Plugins

Edit ~/.zshrc → add:

plugins=(git fzf fzf-tab zsh-autosuggestions zsh-syntax-highlighting)


Reload Zsh:

exec zsh -l

