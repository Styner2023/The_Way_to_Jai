# Chapter 2 - Setting up a Jai Development Environment.

## 2.1 Opening up the Jai compiler toolkit

Until this moment Jai is still closed beta (Jan 2023).
The group members can download the compiler as a 'vanilla' zip file (size +- 220 Mb).
Unzipping this file shows the following contents:

![Jai folder structure](https://github.com/Ivo-Balbaert/The_Way_to_Jai/tree/main/images/jai_folder.png)

Here is what these folders contain:

- _bin_: this contains the Jai compiler executables (jai.exe for Windows, jai-linux for Linux, jai-macos for MacOs, see $ 4.1-4.4) and the LLVM linker lld (see section § 4.5).
- *how_to*: this contains some examples with explanatory comments.
- _examples_: this contains some more advanced example programs.
- _modules_: this contains Jai’s standard library, we'll discuss this in the rest of the book, in particular in § 6B, 19, 27, 31-34.  
We'll discuss the purpose of _editor_support_  (see § 20.5 for natvis) and _redist_ later (see 2.2.4).

## 2.2 Setting up the Jai compiler

### 2.2.1 Copying the compiler to its destination folder

You can use the extracted folder structure as-is, no installation is needed!
Unpack the zip file in a temporary folder using:

    - On Windows: RIGHT-CLICK the file and select Extract All... 
    - On Linux/MacOs (or WSL2 in Windows): use tar -xvf in a terminal

Now copy the _jai_ subfolder in its entirety to its destination (let's call this the _Jai root folder_), as follows:

    - On Windows: for example on the C-drive to c:\jai 
    - On Linux/MacOs (or WSL2 in Windows): for example to /usr/local/bin/jai or /opt/jai or $HOME/jai 
    (assuming $HOME is /home/your_username)
      Rename jai-linux or jai-macos to jai:  mv jai-linux jai (same for Jai-macos)
      You also have to make jai and the link program executable with the following commands 
      (perhaps sudo is needed):
          chmod +x jai
          chmod +x lld-linux (or chmod +x lld-macos)

Now open a terminal in your Jai root folder, and type the command:  
    - On Windows: `jai -version`
    - On Linux/MacOs (or WSL2 in Windows): `./jai -version`
it will show the following output (for your actual version):

_Version: beta 0.1.036, built on 17 August 2022._

But of course you'll want to be able to use Jai from any directory, let's see how this can be done. 

### 2.2.2 Making the jai command system-wide available

To achieve this, do the following:

    - On Windows: add the path to where jai.exe lives (for example c:\jai\bin) to the PATH system 
    environment variable; here is how to do that:
        • open Explorer, right-click This PC, Properties, then the System settings window appears
        • click Advanced system settings, then button Environment variables
        • choose System Variables, Path, click button New, add c:\jai
        • click OK on all open windows 

    - On Linux/MacOs (or WSL2 in Windows): make a symbolic link like ln -s /path/to/jai/bin/jai /usr/bin/jai
        (for example: ln -s $HOME/jai/bin/jai /usr/bin/jai)

        Or add this line to your /etc/profile or $HOME/.profile:
        export PATH="$HOME/jai/bin/:$PATH"

        After creating the link, you can run Jai on Linux/MacOs simply with: `jai`

Test this out by moving in a terminal to another folder than the Jai root folder and typing in the command: `jai -version`. You should see the same output as in the previous section.
    
### 2.2.3 Updating Jai and switching versions

Remove the current _jai_ folder (make sure that you have backed up any files or modules you have added or changed in there) or rename it to _jai_old_.
Then just drop the most recent _jai_ folder as the new Jai root folder, and you're done!
In the same way, you can have many jai-versions in 'parallel', just rename the newest version to *jai_new*, and rename the version you want to use to _jai_. When the work in the older version is finished: rename _jai_ to _jai_vnnn_ and *jai_new* to *jai*.

### 2.2.4 Prerequisite for Windows 

> (from the distribution README.txt) In order to compile on Windows, you need to have installed on your machine at least one Windows SDK (which is how we link to stuff like kernel32, user32, etc) and one Visual C++
> Re-distributable (which has various C-related libraries). If you
> compile C++ programs on your computer, you probably already have this.
> If you don't, run vs_BuildTools.exe, and select the option to install C++ build tools.
> (It is the box in the upper-left corner).

### 2.2.5 Windows as development platform

Windows is the primary development platform for Jai, because most games are written to run on Windows first. Jonathan Blow also uses Visual Studio as IDE for the compiler.

### 2.2.6 Solution for install problem on Linux distros
On many 64 bit Linux platforms (Mint, Ubuntu, ...) starting the Jai compiler gives the following error message:

```
In Workspace 1 ("First Workspace"):
/etc/jai/modules/POSIX/libc_bindings.jai:243,20: Error: /lib/i386-linux-gnu/libdl.so.2: Dynamic library load failed. Error code 2, message: No such file or directory

    // @header dlfcn.h
    dynamic_linker :: #system_library "libdl";

/etc/jai/modules/Basic/posix.jai:1,2: Info: This occurred inside a module that was imported here.

    #import "POSIX";

/etc/jai/modules/Default_Metaprogram.jai:435,2: Info: ... which was imported here.

    }
    #import "Basic";

/home/sl3dge/.build/.added_strings_w1.jai:2,2: Info: ... which was imported here.

    #import "Default_Metaprogram";

    dlerror says: /lib/i386-linux-gnu/libdl.so.2: wrong ELF class: ELFCLASS32
```

The main issue here is: **libdl.so.2: wrong ELF class: ELFCLASS32**  
Other similar errors can occur, like:    
**librt.so.1: wrong ELF class: ELFCLASS32**  
**libpthread.so.0: wrong ELF class: ELFCLASS32**

For some reason when you ask for `libdl` or `librt` or `libpthread`, the OS points you to the 32bit version instead of the 64 bit version.

As suggested on the Discord channel, all that is needed to solve these problems is to install **libc6-dev-amd64**.  
This is done by executing the following commands in a terminal:  
```
1) sudo apt-get update -y
2) sudo apt-get install -y libc6-dev-amd64
```

Check with `jai -version`:  
Version: beta 0.1.039, built on 17 September 2022.

> For Simp or other OpenGL modules you need to install libgl-dev.  

### 2.2.7 Working in WSL on Windows
WSL on Windows with Ubuntu doesn't have this problem on a 64 bit machine.  

Make sure that the Jai compiler is inside the WSL file system. If it is not, compiling can be 10x slower! 

## 2.3 Editor help for coding Jai
Writing a program's source code is easier when you have some support such as syntax highlighting in your code editor. Support exists for vim, Sublime Text 3 and VSCode, see: [Tooling Ecosystem](https://github.com/Jai-Community/Jai-Community-Library/wiki/References#tooling-ecosystem). 

### 2.3.1 Overview of different editors

1) **vim**: [Jai.vim](github.com/rluba/jai.vim): Vim syntax highlighting for the Jai programming language.  			
     
2) **Emacs**:  [Jai-mode.el](https://github.com/krig/jai-mode/blob/master/jai-mode.el)

3) **Notepad++**: [jai_npp](https://github.com/cookednick/jai_npp)
 Syntax highlighting  

4) **Sublime Text 3**:  
Download Sublime Text 3 from [here](https://www.sublimetext.com/) and install.

Installing the [JaiTools package](https://packagecontrol.io/packages/JaiTools) provides a simple build system.  
Activate the build system like this:
- Choose from the Menu:  Tools / Build System / New Build System. Copy the following in the build file, and save it as *jai.sublime-build*:  
```
{
	"cmd": ["jai", "$file"],
	"selector" : "source.jai",
	"file_regex" : "^(.*):(\\d+)\\,(\\d+)\\: Error\\: (.*)$"
}
```
Now you can build the current file with ctrl-B, and navigate to compilation errors via F4.

Also, add the Jai folder to your project via Menu / Projects/ Add Jai folder to project.

Here is the source code of this plugin: [RobinWragg/JaiTools](https://github.com/RobinWragg/JaiTools). It provides syntax highlighting, autocompletion, and Goto Symbol/Anything for the Jai language. 

5) **Visual Studio Code**: (see § 2.2.2)
Iain King: [The Language - Visual Studio Marketplace - v0.0.85](https://marketplace.visualstudio.com/items?itemName=onelivesleft.the-language) – 

6) **Language server**:	[Pyromuffin/jai-lsp](https://github.com/Pyromuffin/jai-lsp) or [Sl3dge78/jai-lsp](https://github.com/Sl3dge78/jai_lsp)

Works with Vim, Emacs, VSCode, and should work with other editors that implement an lsp client.
The jai_lsp version works together with [this](https://github.com/Sl3dge78/jai-lsp-vscode) VS Code extension, which has to be installed with the .vsix file.
Both GitHub folders contain instructions for setting up the client and the server.  
(I haven't been able to get the jai_lsp to work on Windows (11); it probably needs an update and testing.)

An old plugin for Visual Studio exists
7) **Visual Studio**: [Jai Revolution](https://inductive.no/jai/jai-revolution/) 
   plugin for Visual Studio 2013 / 2015 from Inductive AS, published 2015
	Syntax highlighting (cannot be installed in VS 2017).
Hopefully a new version will come when Jai is released.


At this time I recommend the VSCode plugin, because it probably has the most functionality. The Sublime-Text plugin is also nice.
### 2.3.2  Using the Visual Studio Code plugin

Visual Studio Code (VSCode) is one of the most popular programmer’s editors today and can be installed from: [VSCode](https://code.visualstudio.com/), it offers lots of basic functionality (code folding, brace pairing, numbering lines, and so on) and a myriad number of extensions.
There is a VS Code plugin for Jai named _The Language_, which provides basic IDE functionality: [The Language](https://marketplace.visualstudio.com/items?itemName=onelivesleft.the-language).

VSCode is very helpful for editing you source code. Compiling is usually done from the command-line (cmd on Windows or a terminal in Linux/MacOS), but you can do it also from within VSCode by opening up a New Terminal.

> Hint: Whenever you want to search for a procedure (say for example parse_int) in the Jai modules, open VS-code in that folder, select Edit / Find in Files (or the equivalent in your code editor) and fill in `parse_int ::` in the search field. This will only find the definition of parse_int, not all the cases where it is called. In a Linux/MacOS terminal you can also use `grep -rn 'parse_int ::'`.

### 2.3.3 How to edit, build and run a Jai program in VS-Code through CodeRunner

Install the [CodeRunner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)  extension.
Then in File, Preferences, Settings:
  Edit Jai settings by Searching for:  Jai
    If necessary, set the Path to Jai executable.
    Search for:  Run code configuration:
		Custom command:  
        `jai $fileName -exe a.exe & a`
	
This is the same as editing in settings.json:
		 `"code-runner.executorMap": { "jai": "jai $fileName -exe a.exe && a",` 
			...
	
Now you can RIGHT-click on the code window and select the 1st command: `Run Code`, and it will execute this command, which will compile and then run the generated executable a.exe.
We'll show an image of this in chapter 3.

### 2.4 The compiler command

Now open up a terminal and type `jai` and ENTER. You get the following message:

_You need to provide an argument telling the compiler what to compile!  Sorry._

`jai` is a compiler, so it needs at a minimum the path to some source code (file) to compile! 
In the next chapter; we're going to compile our first Jai program, and learn some more about the compiler.

If you're curious about which other command-line options exist for the compiler, see section 2B.

[2B - Compiler command-line options](https://github.com/Ivo-Balbaert/The_Way_to_Jai/blob/main/book/2B_Compiler_command_line_options.md)  
