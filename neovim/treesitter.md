# Treesitter


## Prerequisites

Install the tree-sitter CLI tool globally:

```bash
npm -g install tree-sitter-cli
```

## Step 1: Clone the Parser Repository

Clone the specific tree-sitter parser repository for your target language:

```bash
git clone https://github.com/tree-sitter/tree-sitter-rust.git
```

Replace `tree-sitter-rust` with the appropriate parser for your language (e.g., `tree-sitter-python`, `tree-sitter-javascript`).

## Step 2: Build the Parser

Navigate to the parser directory and build it:

```bash
cd tree-sitter-rust
tree-sitter generate
make
```

Check that the shared library file was created:

```bash
ls -la libtree-sitter-rust.dylib  # On macOS
ls -la libtree-sitter-rust.so     # On Linux
ls -la libtree-sitter-rust.dll    # On Windows
```

## Step 3: Locate Neovim Runtime Paths

Check your Neovim runtime paths to determine where to place the parser:

```bash
nvim --headless -c 'echo &runtimepath' -c 'q!' 2>&1 | tr ',' '\n'
```

## Step 4: Install the Parser

Create the parser directory and copy the library file:

### For Linux/macOS:
```bash
mkdir -p $HOME/.local/share/nvim/site/parser
cp libtree-sitter-rust.dylib $HOME/.local/share/nvim/site/parser/rust.so
```

### Alternative using $NVIM_APPNAME variable:
```bash
export NVIM_APPNAME="nvim-test" # for example

mkdir -p $HOME/.local/share/$NVIM_APPNAME/site/parser
cp libtree-sitter-rust.dylib $HOME/.local/share/$NVIM_APPNAME/site/parser/rust.so
```

**Note:** The file extension may need to be changed to match your system's expected format:
- On macOS: copy as `rust.so` or `rust.dylib`
- On Linux: copy as `rust.so`
- On Windows: copy as `rust.dll`

## Verification

Restart Neovim and open a file of the target language to verify that tree-sitter syntax highlighting is working. You can also run `:checkhealth treesitter` if you have the treesitter plugin configured, or use Neovim's built-in treesitter functionality to verify the parser is loaded.
