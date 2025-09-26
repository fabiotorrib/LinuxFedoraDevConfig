# LinuxFedoraDevConfig
Linux ( Fedora ) System Configuration for Software Engineers

# Fedora for devs : How to config

Depois da instalação do Fedora para configurar drivers, apps básicos e outros utilitários, siga este tutorial:
[GitHub - wz790/Fedora-Noble-Setup: Fedora Linux Noble Setup Guide (Post-Installation)](https://github.com/wz790/Fedora-Noble-Setup?tab=readme-ov-file)

Para assinar drivers da NVIDIA sem precisar desativar o secure boot voce deve seguir as seguintes instruções:

```bash
# 1. Instalar ferramentas
sudo dnf install -y mokutil openssl

# 2. Criar chaves (pasta + gerar certificado)
mkdir -p ~/nvidia-keys && cd ~/nvidia-keys
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der \
  -nodes -days 36500 -subj "/CN=My NVIDIA MOK/"
openssl x509 -in MOK.der -inform DER -out MOK.pem -outform PEM

# 3. Registrar chave no firmware (vai pedir senha para confirmar no boot)
sudo mokutil --import MOK.der
sudo reboot
# -> Tela azul MOK Manager: Enroll MOK > Continue > senha > reboot

# 4. Assinar módulos da NVIDIA (.ko)
modinfo -n nvidia   # mostra caminho do módulo
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 \
  ~/nvidia-keys/MOK.priv ~/nvidia-keys/MOK.pem \
  /usr/lib/modules/$(uname -r)/extra/nvidia/*.ko

# 5. Regenerar initramfs para carregar módulo assinado
sudo dracut --force
sudo reboot

# 6. Verificar se o driver foi carregado
dmesg | grep -i nvidia
nvidia-smi
```

---

## PACOTES IMPORTANTES

#### Compilação & Toolchain

Esses pacotes formam a **base de compilação e build** no Fedora. Incluem compiladores, linkers, sistemas de build e ferramentas para empacotar software.

- **@development-tools** — Grupo com gcc, g++, make, etc.
  
- **cmake / ninja-build** — Sistemas de build modernos.
  
- **clang / clang-tools-extra** — Compilador C/C++ e ferramentas extras (clangd, clang-format).
  
- **pkg-config** — Gerencia flags de compilação/links.
  
- **bear** — Gera `compile_commands.json` para IDEs.
  
- **gdb** — Debugger para C/C++.
  
- **lcov** — Relatórios de cobertura de testes.
  
- **rpm-build / rpmdevtools** — Ferramentas para empacotar em RPM.
  
- **rust** — adiciona a linguagem rust ao sistema
  
- **cargo** — gerenciador de pacotes do rust
  

```bash
sudo dnf install -y @development-tools cmake ninja-build clang clang-tools-extra pkg-config bear gdb lcov rpm-build rpmdevtools rust cargo
```

#### Testes & Qualidade de Código

Frameworks e utilitários que ajudam a **garantir qualidade e estabilidade** do código.

- **boost-devel** — Inclui toda a biblioteca Boost.
  
- **gtest-devel / gmock-devel** — Google Test e Google Mock para C++.
  
- **shellcheck** — Analisa scripts shell e aponta erros.
  

```bash
sudo dnf install -y boost-devel gtest-devel gmock-devel shellcheck
```

#### Debug, Profiling & Diagnóstico

Ferramentas para **detectar vazamentos de memória, analisar performance e monitorar processos**.

- **valgrind** — Detecta erros de memória.
  
- **strace / ltrace** — Traçam syscalls e chamadas de bibliotecas.
  
- **perf** — Perfilador de performance (via kernel-tools).
  
- **htop** — Monitor de processos interativo.
  

```bash
sudo dnf install -y valgrind strace ltrace perf htop
```

#### Gráficos, Multimídia & 3D

Bibliotecas para **jogos, simulações e aplicações gráficas**.

- **mesa-libGLES-devel / mesa-libEGL-devel** — OpenGL ES e EGL.
  
- **glm-devel** — Matemática para gráficos 3D.
  
- **SDL2-devel / SDL2_image-devel** — Desenvolvimento com SDL2.
  

```bash
sudo dnf install -y mesa-libGLES-devel mesa-libEGL-devel glm-devel SDL2-devel SDL2_image-devel
```

#### Sistema, IPC & Serviços

APIs e libs que permitem **interagir com o Linux e seus serviços**.

- **dbus-devel** — Comunicação entre processos.
  
- **systemd-devel** — Integração com o systemd.
  
- **libcap-devel** — Permissões granulares de processos.
  
- **expat-devel** — Parser XML rápido.
  
- **libuuid-devel** — Geração e manipulação de UUIDs.
  

```bash
sudo dnf install -y dbus-devel systemd-devel libcap-devel expat-devel libuuid-devel
```

#### Containers & Virtualização

Ferramentas para **rodar ambientes isolados e simular máquinas**.

- **lxc / lxc-devel** — Containers Linux.
  
- **podman** — Containers rootless (nativo no Fedora).
  
- **docker / docker-compose** — Containers clássicos (precisa repo extra).
  
- **vagrant** — Automação de VMs.
  
- **qemu-kvm / virt-manager** — Virtualização e interface gráfica.
  

```bash
sudo dnf install -y lxc lxc-devel podman vagrant qemu-kvm virt-manager
```

#### Serialização & RPC

Bibliotecas para **troca de dados eficiente entre sistemas**.

- **protobuf-devel / protobuf-compiler** — Protocol Buffers.

```bash
sudo dnf install -y protobuf-devel protobuf-compiler
```

#### Rede & CLI Web

Ferramentas para **testar APIs, monitorar rede e criptografia**.

- **curl / wget** — Clientes HTTP básicos.
  
- **httpie** — Cliente HTTP amigável.
  
- **nmap** — Scanner de rede.
  
- **wireshark / wireshark-cli** — Sniffer de pacotes.
  
- **net-tools / iproute** — Ferramentas de rede.
  
- **mosh** — SSH melhorado.
  
- **gnupg2** — Criptografia e assinatura digital.
  

```bash
sudo dnf install -y curl wget httpie nmap wireshark wireshark-cli net-tools iproute mosh gnupg2
```

#### Bancos de Dados & Armazenamento

Pacotes para **usar e integrar bancos de dados no desenvolvimento**.

- **sqlite / sqlite-devel** — SQLite.
  
- **postgresql / postgresql-devel** — Cliente PostgreSQL.
  
- **mariadb / mariadb-devel** — Cliente MySQL/MariaDB.
  
- **redis / redis-devel** — Redis.
  

```bash
sudo dnf install -y sqlite sqlite-devel postgresql postgresql-devel mariadb mariadb-devel redis redis-devel
```

#### Desenvolvimento Python

Ferramentas para **projetos Python modernos**.

- **python3-virtualenv / python3-venv** — Ambientes virtuais.
  
- **ipython** — REPL interativo.
  
- **jupyter-notebook / jupyterlab** — Notebooks científicos.
  
- **mypy / black / flake8 / isort** — Linters e formatadores.
  
- **tk-devel** — Tkinter (GUIs).
  
- **bzip2-devel, gdbm-devel, libffi-devel, xz-devel, ncurses-devel, readline-devel, sqlite-devel, zlib-devel, openssl-devel** — Dependências C usadas pelo Python.
  

```bash
sudo dnf install -y python3-virtualenv python3-venv ipython jupyter-notebook jupyterlab mypy black flake8 isort tk-devel bzip2-devel gdbm-devel libffi-devel xz-devel ncurses-devel readline-devel sqlite-devel zlib-devel openssl-devel
```

### Documentação & Diagramas

Ferramentas para **gerar documentação e diagramas automáticos**.

- **doxygen** — Documentação C/C++.
  
- **graphviz** — Diagramas de grafos.
  
- **pandoc** — Conversão de documentos.
  

```bash
sudo dnf install -y doxygen graphviz pandoc
```

#### Produtividade no Terminal

Pacotes que **tornam o terminal mais rápido e prático**.

- **fzf** — Busca fuzzy interativa.
  
- **ripgrep** — Grep moderno e veloz.
  
- **bat** — Cat com highlight.
  
- **exa** — ls melhorado com cores e árvore.
  
- **unzip** — Extrair arquivos ZIP.
  

```bash
sudo dnf install -y fzf ripgrep bat exa unzip
```

## Linha única com tudo

> Instala todos os pacotes de uma vez (exceto Docker, PlantUML e MongoDB que podem precisar de repositórios extras).

```bash
sudo dnf install -y \ @development-tools cmake ninja-build clang clang-tools-extra pkg-config bear gdb lcov rpm-build rpmdevtools \ boost-devel gtest-devel gmock-devel shellcheck \ valgrind strace ltrace perf htop \ mesa-libGLES-devel mesa-libEGL-devel glm-devel SDL2-devel SDL2_image-devel xorg-x11-utils \ dbus-devel systemd-devel libcap-devel expat-devel libuuid-devel \ lxc lxc-devel podman vagrant qemu-kvm virt-manager \ protobuf-devel protobuf-compiler \ curl wget httpie nmap wireshark wireshark-cli net-tools iproute mosh gnupg2 \ sqlite sqlite-devel postgresql postgresql-devel mariadb mariadb-devel redis redis-devel \ python3-virtualenv python3-venv ipython jupyter-notebook jupyterlab mypy black flake8 isort tk-devel \ bzip2-devel gdbm-devel libffi-devel xz-devel ncurses-devel readline-devel sqlite-devel zlib-devel openssl-devel \ doxygen graphviz pandoc \ fzf ripgrep bat exa unzip
```

---

## Anaconda, VSCode, uv and VTM

Este script instala três ferramentas populares no Fedora: **Anaconda**, que fornece um ambiente completo de Python com bibliotecas de ciência de dados; **VTM**, um gerenciador de terminais que pode ser instalado via Cargo (necessita do Rust instalado previamente); e **Visual Studio Code**, editor de código moderno da Microsoft.

```bash
# Anaconda
cd ~ && wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh
bash anaconda.sh -b -p $HOME/anaconda3
echo 'export PATH="$HOME/anaconda3/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc

# VTM (precisa ter Rust instalado antes)
cargo install vtm
# uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# VS Code
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo dnf install -y https://go.microsoft.com/fwlink/?LinkID=760867
```

---

## Bibliotecas Python

```bash
pip3 install jedi black ruff ipython --user
```

---

## VSCode Plugins

#### Aparência e Temas

- **pkief.material-icon-theme** → Ícones de arquivos estilo Material.
  
- **naumovs.color-highlight** → Destaca cores (#fff, rgb, etc.).
  

### Produtividade e Utilidades

- **aaron-bond.better-comments** → Comentários coloridos e destacados.
  
- **alefragnani.bookmarks** → Marcadores no código.
  
- **gruntfuggly.activitusbar** → Customizar a barra lateral de atividades.
  
- **sleistner.vscode-fileutils** → Manipulação rápida de arquivos.
  
- **roscop.activefileinstatusbar** → Mostra arquivo ativo na status bar.
  
- **nick-rudenko.back-n-forth** → Navegação rápida entre locais de edição.
  
- **sergeyegorov.folder-color** → Mudar cor de pastas no Explorer.
  
- **mgesbert.indent-nested-dictionary** → Melhor indentação para dicionários.
  
- **exodiusstudios.comment-anchors** → Âncoras visuais em comentários.
  
- **cliffordfajardo.highlight-line-vscode** → Destaca a linha atual.
  
- **ryu1kn.text-marker** → Marca palavras/trechos no editor.
  
- **stkb.rewrap** → Quebra de linha inteligente em comentários/texto.
  
- **shardulm94.trailing-spaces** → Destaca/remover espaços no fim da linha.
  
- **timonwong.shellcheck** → Integração com ShellCheck (lint bash).
  
- **tyriar.sort-lines** → Ordenar linhas rapidamente.
  
- **mechatroner.rainbow-csv** → Visualização de CSV colorida.
  
- **bierner.markdown-preview-github-styles** → Preview de Markdown estilo GitHub.
  
- **tomoki1207.pdf** → Visualizar PDFs no VS Code.
  
- **yutengjing.open-in-external-app** → Abrir arquivos em apps externos.
  
- **yutengjing.vscode-archive** → Abrir arquivos ZIP diretamente.
  
- **foxundermoon.shell-format** → Formatador de shell scripts.
  
- **sleistner.vscode-fileutils** → Operações rápidas com arquivos.
  
- **christian-kohler.path-intellisense** → Autocompleta nomes de arquivos e paths.
  
- **usernamehw.errorlens** → Mostra erros e warnings diretamente no código, sem precisar olhar o painel de problemas.
  

### Python

- **ms-python.python** → Suporte principal ao Python.
  
- **ms-python.debugpy** → Debug Python.
  
- **charliermarsh.ruff** → Linter/ferramenta rápida para Python.
  
- **franneck94.vscode-cpython-extension-pack** → Pacote de extensões para C/Python.
  
- **benjamin-simmonds.pythoncpp-debug** → Debug híbrido Python/C++.
  
- **ms-python.vscode-pylance**→ Melhor análise estática e IntelliSense para Python.
  
- **ms-toolsai.jupyter** → Suporte a Jupyter notebooks no VS Code.
  
- **ms-toolsai.jupyter-keymap** → Atalhos de teclado estilo Jupyter.
  
- **ms-toolsai.jupyter-renderers** → Melhor visualização de gráficos no notebook.
  

### Cython

- **guyskk.language-cython** → Sintaxe Cython.
  
- **ktnrg45.vscode-cython** → Outro suporte Cython.
  
- **tcwalther.cython** → Extensão Cython extra.
  

### C / C++ / Rust

- **ms-vscode.cpptools** → IntelliSense, debug, CMake etc.
  
- **llvm-vs-code-extensions.vscode-clangd** → Suporte a Clangd.
  
- **cmstead.js-codeformer** → Refatoração de JavaScript (útil em C++ também).
  

### Java

- **vscjava.vscode-java-pack** → Pacote completo para Java.
  
- **vscjava.vscode-java-dependency** → Gerenciamento de dependências Java.
  
- **vscjava.vscode-java-debug** → Debug Java.
  
- **vscjava.vscode-maven** → Suporte ao Maven.
  

### Bancos de dados e Ferramentas Web

- **qwtel.sqlite-viewer** → Visualizar bancos SQLite.
- **mtxr.sqltools** → Cliente SQL integrado com suporte a PostgreSQL, MySQL, SQLite etc.
- **ritwickdey.liveserver** → Servidor local para visualizar HTML/CSS/JS ao vivo.
- **esbenp.prettier-vscode** → Formatador para JS/TS/HTML/CSS.

### Terminais & Shell

- **adrianwilczynski.terminal-commands** → Gerenciar comandos no terminal integrado.
  
- **antiantisepticeye.vscode-color-picker** → Selecionar cores.
  
- **artdiniz.quitcontrol-vscode** → Controle rápido de fechamento.
  
- **foxundermoon.shell-format** → Formatador para shell scripts.
  
- **mads-hartmann.bash-ide-vscode** → IDE para Bash.
  

### AI e Automação

- **codeium.codeium** → Autocompletar com IA.

### Git, Diff e Colaboração

- **jinsihou.diff-tool** → Comparar arquivos.
  
- **l13rary.l13-diff** → Outra ferramenta de diff.
  
- **vsls-contrib.gistfs** → Abrir Gists do GitHub como FS.
  
- **github.codespaces** → Integração com GitHub Codespaces.
  
- **eamodio.gitlens** → Ferramenta avançada para Git (blame, histórico, comparação).
  

### Extras

- **tyriar.luna-paint** → Editor de pixel art.
  
- **tldraw-org.tldraw-vscode** → Integração com tldraw (diagramas).
  
- **tomoki1207.vscode-input-sequence** → Gera sequências de entrada automáticas.
  
- **guiextensions.tosingleline** → Junta várias linhas em uma só.
  
- **wscats.command-runner** → Executa comandos pré-configurados.
  
- **cmstead.js-codeformer** → Refatorador JS.
  

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
    "ruff.showNotifications": "always",
    "ruff.trace.server": "verbose",
    "ruff.ignoreStandardLibrary": false,
    "ruff.lint.run": "onType",
    "ruff.organizeImports": true,
    "ruff.lint.args": [
        "--select=ALL",
        "--extend-select=UP",
        "--fix"
    ],

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

Um script interativo que centraliza a gestão de ambientes Python no Linux. Ele permite criar novos ambientes tanto com **Conda** quanto com **uv**, listar os já existentes, e abrir qualquer um deles diretamente no **VS Code** com o interpretador configurado automaticamente.

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
err(){ printf "\n\033[1;31m[ERRO]\033[0m %s\n" "$*" >&2; }

# VS Code fora do script -> sempre Python global
set_vscode_global_python() {
  [ -n "${PY_GLOBAL}" ] || return 0
  ensure_dir "$(dirname "$CODE_USER_SETTINGS")"
  if have jq && [ -f "$CODE_USER_SETTINGS" ]; then
    tmp="$(mktemp)"; jq --arg p "$PY_GLOBAL" '. + {"python.defaultInterpreterPath": $p}' "$CODE_USER_SETTINGS" > "$tmp" || true; mv "$tmp" "$CODE_USER_SETTINGS"
  elif have jq; then
    jq -n --arg p "$PY_GLOBAL" '{ "python.defaultInterpreterPath": $p }' > "$CODE_USER_SETTINGS"
  else
    # fallback simples (não valida JSON existente)
    printf '{\n  "python.defaultInterpreterPath": "%s"\n}\n' "$PY_GLOBAL" > "$CODE_USER_SETTINGS"
  fi
}

# Cria runner pyexe DENTRO do env (como no seu script original), chamando IPython via VTM
make_env_pyexe() {
  local env_dir="$1" env_name="$2" pybin="$3"
  local logfile="${env_dir}/ipython.log"
  cat >"${env_dir}/pyexe" <<EOF
#!/bin/sh
# Runner IPython dentro do VTM (env: ${env_name})
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

# Garante ipython no ambiente
ensure_ipython() {
  local pybin="$1"
  "$pybin" - <<'PY' || "$pybin" -m pip install -q --upgrade ipython >/dev/null
import importlib, sys
sys.exit(0 if importlib.util.find_spec("IPython") else 1)
PY
}

# Abre VS Code no env com interpretador fixado no workspace
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

# ========== Fluxos ==========

# 0) Python GLOBAL no VTM (sem env)
open_global_ipython_vtm() {
  have vtm || { err "vtm não encontrado (instale com 'cargo install vtm' ou pacote da distro)"; exit 1; }
  [ -n "$PY_GLOBAL" ] || { err "python3 global não encontrado"; exit 1; }
  ensure_ipython "$PY_GLOBAL"
  msg "Abrindo IPython GLOBAL no VTM..."
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

# 1) Criar venv com uv (com pyexe e VS Code)
create_uv_env() {
  have uv || { err "uv não instalado. Ex.: curl -LsSf https://astral.sh/uv/install.sh | sh"; exit 1; }
  ensure_dir "$UV_ENVS_DIR"
  read -rp "Nome do env (uv): " name; [ -n "$name" ] || { err "Nome inválido."; exit 1; }
  read -rp "Versão do Python (ex.: 3.11) [vazio = padrão]: " pyver || true
  local env_dir="${UV_ENVS_DIR}/${name}"
  [ -d "$env_dir" ] && { err "Já existe: ${env_dir}"; exit 1; }
  msg "Criando env uv '${name}'..."
  if [ -n "${pyver:-}" ]; then uv venv --python "python${pyver}" "$env_dir"; else uv venv "$env_dir"; fi
  local pybin="${env_dir}/bin/python"
  "$pybin" -m ensurepip --upgrade >/dev/null 2>&1 || true
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "uv:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Use: ${env_dir}/pyexe  # para abrir IPython no VTM neste env"
}

# 2) Criar env com conda (com conda init/activate, pyexe e VS Code)
create_conda_env() {
  have conda || { err "conda não encontrado. Instale Miniconda/Anaconda e rode 'conda init'."; exit 1; }
  read -rp "Nome do env (conda): " name; [ -n "$name" ] || { err "Nome inválido."; exit 1; }
  read -rp "Versão do Python (ex.: 3.11): " pyver; [ -n "$pyver" ] || { err "Informe a versão do Python."; exit 1; }
  msg "Criando env conda '${name}'..."
  conda create -y -n "$name" "python=${pyver}" ipython >/dev/null
  local env_dir="${CONDA_ENVS_DIR}/${name}" pybin="${env_dir}/bin/python"
  # Garante conda init/activate como no seu fluxo original
  conda init >/dev/null 2>&1 || true
  # Criar pyexe e abrir Code
  make_env_pyexe "$env_dir" "conda:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Use: ${env_dir}/pyexe  # para abrir IPython no VTM neste env"
}

# Entrar em env Conda existente (com conda init/activate, pyexe e VS Code)
enter_conda_env() {
  local name="$1"
  local env_dir="${CONDA_ENVS_DIR}/${name}"
  local pybin="${env_dir}/bin/python"
  [ -x "$pybin" ] || { err "Python do env conda '${name}' não encontrado em ${pybin}"; exit 1; }
  # Preserva seu comportamento: conda init e conda activate no shell interativo do VTM, se quiser
  conda init >/dev/null 2>&1 || true
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "conda:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  # Além do Code, já oferecemos abrir IPython agora:
  msg "Abrindo IPython no VTM do env conda '${name}'..."
  "${env_dir}/pyexe"
}

# Entrar em env uv existente (com pyexe e VS Code)
enter_uv_env() {
  local name="$1"
  local env_dir="${UV_ENVS_DIR}/${name}"
  local pybin="${env_dir}/bin/python"
  [ -x "$pybin" ] || { err "Python do env uv '${name}' não encontrado em ${pybin}"; exit 1; }
  ensure_ipython "$pybin"
  make_env_pyexe "$env_dir" "uv:${name}" "$pybin"
  fix_workspace_interpreter_and_open_code "$env_dir" "$pybin"
  msg "Abrindo IPython no VTM do env uv '${name}'..."
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
  echo "0) Abrir Python GLOBAL (fora de envs) no IPython (via VTM)"
  echo "1) Criar venv com uv"
  echo "2) Criar env com conda"
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
  read -rp "Escolha: " choice
  [[ "$choice" =~ ^[0-9]+$ ]] || { err "Escolha inválida."; exit 1; }

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
      err "Opção fora do intervalo."
      exit 1
      ;;
  esac
}

main_menu
```

Acesso privilegiado

```bash
chmod +x ~/condalinux.sh
```
