# OpenCV-Python Neovim Project Template
1. You need to install `poetry` as that's the dependency manager used for this template.
2. You can install poetry by following instructions mentioned in the [poetry official site](https://python-poetry.org/)
  

# Working With OpenCV Project Using Neovim
I should mention that configuring Neovim for opencv-python projects is slightly different from it's C++
counterpart. Firstly, `opencv` is not a Python module, therefore, Python only provides bindings to the C++ library
calls. Unless your autocompletion engine is really `overpowered` (like Kite), Python LSP that provides completion
via _type-checking_ won't do much for `opencv-python`, since it DOES NOT SHIP with it's `stub`. There have been
many requests to do so, and hopefully it will be realized soon. 

So what do we do? We generate the `stub file` ourselves. This will provide better _type-hinting_ for our
_type-checking_-based LSP (I use `pyright`).

1. First we need to create the project. This will also create new directory with the same name as the project.
    ```bash
    $ poetry new <project_name>
    ```  

2. Enter the project directory, and configure the project dependencies.  

3. Create the `stub` for `opencv` using `stubgen` from the `mypy` package.

4. Now the LSP won't complain anymore.  **Happy vimming**!



# Configuring dependencies
There are 2 kinds of dependencies:
- Project Dependencies
- Development Dependencies

Project dependencies will always be installed whenever you deploy the application, but development
dependencies are for development purposes only. For example, I want to do unit testing on the application,
we can use `pytest`. On the development side, we often will use, the `pytest` package to test things.
But when the app is deployed, does the client need to run `testing`? And that's what Development dependencies
are for.

And make sure, when you are configuring the dependencies, you are in the virtual environment SHELL of your project.
* Generate virtual environment (if you haven't done so)
    ```bash
    $ poetry shell
    ```
* Enter the SHELL of existing environment
    ```bash
    # Check for existing environment
    $ poetry env info
    
    # activate existing
    $ source $(poetry env info --path)
    ```
#### ADDING
* Manually from the shell
  ```bash
  # if you dont specify the version package the latest one will be used
  # for project dependencies
  $ poetry add <package>
  
  # for development dependencies, use --dev flag
  $ poetry add --dev <package>
  ```
* By editing the `pyproject.toml` (to specify any version use `*` wildcard)
  ```toml
  # COMMENTS starts with #
  
  # Configure your project dependencies here
  # for packages that have special characters in their name, wrap them in quotes
  # ex: 
  # "discord.py" = "*"
  [tool.poetry.dependencies]
  python = "3.9.5"
  opencv-python = "^4.5.3"
  opencv-contrib-python = "^4.5.3"
  
  # List your development dependencies here
  [tool.poetry.dev-dependencies]
  neovim = "^0.3.1"
  mypy = "^0.910"
  pytest = "*"
  ```
  then run in the shell:
  ```bash
  $ poetry install
  ```
  If you do not want to install the development dependencies:
  ```bash
  $ poetry install --no-dev
  ```
  
#### REMOVING
* Directly from the shell
    ```bash
    $ poetry remove <package_name>
    ```
* By editing the `pyproject.toml`, removing the unneeded packages and the run this in the shell:
    ```bash
    $ poetry install --remove-untracked
    ```
    

#### LIST DEPENDENCIES
If you want to list current project dependencies:
```bash
$ poetry show
```

## Generating OpenCV stub file for the LSP
1. Firstly, after configuring the dependencies we need to make sure the `mypy` package is present
    ```bash
    $ poetry show | grep mypy
    
    # EXPECTED OUTPUT: 
    #   mypy                  0.910    Optional static typing for Python
    #   mypy-extensions       0.4.3    Experimental type system extensions for prog...
    ```  
    
2. After that, verify where your virtual environment directory is. That way we can locate the `site-packages`  
directory.
    ```bash
    poetry env info --path
    
    # Mine is:
    # /home/yeyee2901/.cache/pypoetry/virtualenvs/test-stub-bNKZ_6BG-py3.9
    
    # Which in general is
    # /home/yeyee2901/.cache/pypoetry/virtualenvs/<project_name>-<project_hash>
    ```
    
3. From there, there sould be directory with a structure like this:
    ```
    /home/yeyee2901/.cache/pypoetry/virtualenvs/<project_name>-<project_hash>
    ├── include
    │   └── site
    ├── lib
    │   └── python3.9
    └── pyvenv.cfg
    ```
4. Alright, the `site-package` is actually located in `lib - python3.9 - site-packages`, This may vary  
depending on the Python version that you use.

5. Finally we need to generate the `stub` file using `stubgen` and place it in:  
   `lib/python<version>/site-packages/cv/`  

   You can specify the path manually like these:
   ```bash
   stubgen -m cv2 -o <path_to_your_cv2_site_packages>
   ```  
   
   But I suggest using a little `bash` scripting to make our life a bit easier:
   ```bash
   stubgen -m cv2 -o $(poetry env info --path)/lib/**/site-packages/cv2/
   ```
