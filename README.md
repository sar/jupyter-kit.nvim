# vim-jukit

This plugin aims to provide an alternative for users who frequently work with python in jupyter-notebook and are searching for a way to work with jupyter-notebook files in vim. The goal here is not to replicate the features of jupyter-notebook in vim, but merely to provide a convenient way to convert and edit jupyter-notebook files in vim. 

It uses the graphical capabilities of the `kitty` terminal emulator (https://github.com/kovidgoyal/kitty) and incorporates the functionality of the packages`ipynb_py_convert` (https://github.com/kiwi0fruit/ipynb-py-convert) as well as `matplotlib-backend-kitty` (https://github.com/jktr/matplotlib-backend-kitty) and makes it possible to:
* easily send code to another split-window in the kitty-terminal 
* run individual lines, visually selected code, or whole cells like in jupyter-notebook
* display matplotlib plots in the terminal using `matplotlib-backend-kitty` 
* map keys for converting jupyter-notebook files to simple python-files and back using `ipynb_py_convert`

## Examples

Converting a .ipynb file to a .py file
![converting_to_py](https://user-images.githubusercontent.com/57172028/113363801-26a6d080-9352-11eb-9a1e-b32bbb59b707.gif)


Opening python shell in split window and sending code to window
![sending_code](https://user-images.githubusercontent.com/57172028/113363862-535ae800-9352-11eb-8061-12bb53513c9f.gif)


Converting .py file back to .ipynb, then convert to .pdf and open it
![convert_to_pdf](https://user-images.githubusercontent.com/57172028/113363912-7e453c00-9352-11eb-87dd-23dfe3590e02.gif)


## Requirements:

* kitty terminal emulator (https://github.com/kiwi0fruit/ipynb-py-convert)
* vim with python3 support
* vim with '+clipboard' to access the system clipboard (check with 'echo has("clipboard")' in vim)

## Installation

With your plugin manager of choice, e.g. using vim-plug:

```
Plug 'luk400/vim-jukit' 
```

## Usage

#### User defined variables

###### Default values
```
let g:jukit_highlight_markers = 1
let g:jukit_hl_settings = 'ctermbg=22 ctermfg=22'
let g:jukit_use_tcomment = 0
let g:jukit_comment_mark_default = '#'
let g:jukit_python_cmd = 'python'
let g:jukit_inline_plotting_default = 1
let g:jukit_register = 'x'
let g:jukit_html_viewer = 'firefox'
let g:jukit_pdf_viewer = 'zathura'
let g:jukit_mappings = 1
```

###### Explanation
`g:jukit_highlight_markers`: Specify if cell markers should be highlighter (1) or not (0)

`g:jukit_hl_settings`: Specify arguments for highlighting cell markers (see `:h highlight-args`)

`g:jukit_use_tcomment`: Specify if tcomment (https://github.com/tomtom/tcomment_vim) plugin should be used to comment out cell markers - recommended if you work with many different languages for which you intend to make use cell markers, otherwise you will have to manually set `b:jukit_comment_mark` every time you work on a file where `b:comment_mark` != `g:jukit_comment_mark_default`. Requires the tcomment plugin to already be installed.

`g:jukit_comment_mark_default`: Every time a buffer is entered the variable `b:comment_mark` is set according to `g:jukit_comment_mark_default` to prepend the cell markers in the file. Only required if `g:jukit_use_tcomment=0`.

`g:jukit_python_cmd`: Specifies the terminal command to use to start the interactive python shell ('python' or 'ipython')

`g:jukit_inline_plotting_default`: Every time a buffer is entered the variable `b:inline_plotting` is set according to this default value, which either enables (1) or disables (0) directly plotting into terminal using when matplotlib.

`g:jukit_jukit_register`: This is the register to which jukit will yank code when sending to the kitty-terminal window.

`g:jukit_html_viewer`: Specifies the html-viewer to use when using the `jukit#SaveNBToFile()` function (see next section)

`g:jukit_mappings`: If set to 0, no jukit function mappings will be set by default and you will have to define each desired mapping manually in your .vimrc.

### Functions and Mappings

###### Default mappings
```
nnoremap <leader>py :call jukit#PythonSplit()<cr>
nnoremap <leader>sp :call jukit#ReplSplit()<cr>
nnoremap <cr> :call jukit#SendLine()<cr>
vnoremap <cr> :<C-U>call jukit#SendSelection()<cr>
nnoremap <leader><space> :call jukit#SendSection()<cr>
nnoremap <leader>cc :call jukit#SendUntilCurrentSection()<cr>
nnoremap <leader>all :call jukit#SendAll()<cr>
nnoremap <leader>mm :call jukit#NewMarker()<cr>
nnoremap <leader>np :call jukit#NotebookConvert(1)<cr>
nnoremap <leader>pn :call jukit#NotebookConvert(0)<cr>
nnoremap <leader>ht :call jukit#SaveNBToFile(0,1,'html')<cr>
nnoremap <leader>rht :call jukit#SaveNBToFile(1,1,'html')<cr>
nnoremap <leader>ht :call jukit#SaveNBToFile(0,1,'pdf')<cr>
nnoremap <leader>rht :call jukit#SaveNBToFile(1,1,'pdf')<cr>
```

###### Explanation
`jukit#PythonSplit()`: Creates a new kitty-terminal-window and opens the python shell using the matplotlib backend for inline plotting (if `b:inline_plotting == 1`)
`jukit#ReplSplit()`: Only creates a new kitty-terminal-window
`jukit#SendLine()`: Sends the line at the current cursor position to the other window
`jukit#SendSelection()`: Sends the visually selected text
`jukit#SendSection()`: Sends the whole current section (i.e. the code inbetween cell-markers)
`jukit#SendUntilCurrentSection()`: Sends from the beginning of the file until (and including) the current section
`jukit#SendAll()`: Send content of whole file
`jukit#NewMarker()`: Create a new cell-marker
`jukit#NotebookConvert(1)`: Convert from .ipynb to .py
`jukit#NotebookConvert(0)`: Convert from .py to .ipynb
`jukit#SaveNBToFile(0,1,'html')`: Convert the existing .ipynb to .html and open it
`jukit#SaveNBToFile(1,1,'html')`: Convert the existing .ipynb to .html and open it, but run the code and include output
`jukit#SaveNBToFile(1,1,'pdf')` and `jukit#SaveNBToFile(0,1,'pdf')`: same as above, but with .pdf instead of .html

In general, the arguments of the functions `jukit#NotebookConvert()` and `jukit#SaveNBToFile()` are as follows:

`jukit#NotebookConvert(from_notebook)`: `from_notebook` is either 1 indicating that the current file is .ipynb and should be converted to .py, or 0 indicating the converse.

`jukit#SaveNBToFile(run, open, to)`: Must have Jupyter installed to work, since it uses the command `jupyter nbconvert` in terminal. Here, `run` is 1 or 0 and specifies if the code should be run to include output when converting, `open` is 1 or 0 indicating if file should be opened afterwards, and `to` specifies the output format. Generally, `to` can take any argument accepted by the `--to` flag of `jupyter nbconvert` (see `jupyter nbconvert -h`), note however that you need to specify a viewer to open the file (i.e. if you're using `jukit#SaveNBToFile(0, 1, 'pdf')` you also need to specify a variable `g:jukit_pdf_viewer`, e.g. `let g:jukit_pdf_viewer = 'zathura'` in you vimrc.


### Commands

When working with virtual environments, you can activate it before starting the python shell using the `KittyPy` command, e.g.:


```
:JukitPy conda activate myvenv
```

This will open a new kitty terminal window, activate the virtual environment using the given command, and then open the python shell with the matplotlib backend for inline plotting (if `b:inline_plotting == 1`).

## Other notes to be aware of

* When using ipython, be aware that the code is copied to the system clipboard and then pasted into the ipython shell using '%paste', thus modifying the contents of your system clipboard. 
* Converting ipynb-files using the `jukit#SaveNBToFile()` function has only been tested with pdf and html output thus far, and there are cases where converting to pdf may fail (e.g. when an image in the notebook should by displayed using a hyperlink).
* Everytime you open the python shell using `jukit#PythonSplit()` with `b:inline_plotting=1`, matplotlib is automatically imported at the beginning (to specify the backend matplotlib should use).

