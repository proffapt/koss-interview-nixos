## This is where i will write the content of script.md in short

* Slide 1
	Introduce yourself and the topic

* Slide 2
	options includes things like services to be enabled
	What is nix.. a package-manager.. an os.. or just a lang?.. give a brief speech on this

* The Nix OS
	- Reproducability, declrative, reliable
		* Reproducability: How often do you find yourself in a situation where something builds and works on your machine, but doesn’t build on CI 
		or fails catastrophically in production?
		In many cases, those problems are a symptom of something most developers face: lack of reproducibility. 
		Your code depends on an environment variable at compile time, or significantly changes behavior with different versions of a dependency.
	- Read out the stuff from wiki
	NixOS is both a Linux distro and a package manager, but unlike other disros (Ubuntu, Fedora, Arch Linux, etc.) it is a declarative OS/package manager. 
	It means, the packages and dependencies are always mentioned before-hand (in a configuration file), which avoids dependency hell.
	- Instant runtime enviroment
		- line 1: declaring which interpreter to use for the script
		- line 2: telling the nix-shell which interpreter to use further.. and what packages will be needed
		- line 3: pinning the dependency being used.. nixpkgs in this case.
		What will nix do??
			It will prepare an enviroment with specified dependencies for the scripts to run
	- Instant development enviroment
	  	- With only Nix, you’re able to launch a shell environment with not only Python and additional libraries but also jupyter, ffmpeg, youtube-dl, curl, wget, httpie.
	   	  It didn’t ask you to even run with sudo nor nag you about confusing missing library errors, it just works and all you need is Nix and a shell.nix file.
		- On how and what did it do, we shall look in the section 4.
	- Problems with current software deployment methods
		(So much of elaborative data to enter from the article https://notes.yukiisbo.red/posts/2021/09/Spice_up_with_Nix_Traditional_Software_Deployment.html)
		So these were elaborative problems.. how does nix solve all of these?.. shall look into this when we will be discussing the working of nix

* The Nix Language
	- Gibberish from wikipedia
		- Elaborate the three terms
			* Purity
				- Operations in the language don't have side-effects (for instance, there is no variable assignment).
			* Laziness
				- Arguments to functions are evaluated only when they are needed.
			* Functional
				- It belongs the the functional paradigm of programming langauges, in which you approach the solution to
				a problem via divide and conquer method.. dividing a probelem into sub-problems and the soloving them 
				individually.. later compiling them together into a single solution.
	- variables
		* immutable
		* strings
	- Sets
		* key value pair
		* like dict and hash tables in other programming langs
	- Lists
		* no commas
	- Functions
		* lambda functions
		* default arguments
		* unexpected argument
	- Derivations
		* better alternatives - will look into the section explaining the dark secrets of nixos
	- spcials
		* with
		* import
		* inherit
		* if 
		* let
		* rec
	- About repl

* Creating own package using nix
	- nix-build command
		* It’s built and the resulting component is stored inside /nix/store/y1aimywh5ff57pv2azg705hlkcciy2dn-youtube-launcher-0.1 as indicated by the last line of nix-build.

* What does it do under the hood
	- Declarative system configuration model
		The entire operating system — the kernel, applications, system packages, configuration files, and so on — is built by the Nix package manager from a description 
		in a purely functional build language. The fact that it’s purely functional essentially means that building a new configuration cannot overwrite previous configurations. 
		Most of the other features follow from this. 
	- We have already seen the creation of nix packages in the previous section, let's look into the process a bit more deeper
	- Elaborate on the hashing method and reason, which is to have multiple uncompaitible version of same thing in once place /nix/store
	  Do you recall the issues with the simple Python script we made?
		The problem with different systems having software installed in different places.
		The problem with different versions or variants of the same software.
	  Nix solves the issues mentioned above by storing components in a deterministic path which are prefixed with a cryptographic hash.
	  The hash itself is generated based on the inputs (i.e source code, build flags, dependencies) used to build the component.
	  If any of the inputs changed (ex. one of its dependencies was updated), the hash will be different which will result in a different path. 
	- What does shell.nix do?
		 - You’re essentially giving Nix a shopping list of what you need in your shell environment. 
	1) Reading it, you might think this is how you define imports but actually, this is how you define lambda functions in Nix.
	   Your shell.nix file actually define a single function which takes pkgs as an argument but also provide a default value. 
	2) We define a variable called pythonEnv which takes the result of the pkgs.python38.withPackages function. This function returns 
	   an altered Python package with the Python libraries we’ve defined.
	   Function calls in Nix doesn’t use parentheses similar to Haskell and Ruby. You simply pass the arguments to it separated by whitespaces.
	   The argument we pass is actually another function which takes a single argument called ps and returns a list of Python libraries/packages.
	   You might be confused why it needs to be a function and that’s because this allows multiple Python versions since not all Python versions 
	   share the same collection of packages. 
	- What does nix as a package manager do?
	   Beneath the seemingly complex language, you’ll see that the very foundation of Nix are built with basic fundamental building blocks
	   In the end: 
	   As you can see, unlike curl | sh installers, the Nix expression language has a very limited scope of what it can actually do.
	- Looking at the package we built
		- image 1: 
			* Everything in here uses the deterministic path generated by Nix which has the characteristic cryptographic hash prefix.
			  As you can see, it defines various environment variables required to run the application such as PATH which includes the exact version of Python required along 
			  with a bunch of other variables to make Qt applications work.
			  That last exec line executes our Python script, right? 
		- image 2: 
			*  The shebang is now deterministic to the exact version of Python used for this program.
			   The line with import sys; import site; defines the entire Python site path.
			   If you are unfamiliar, in Python, the site path defines a list of paths where it can find Python libraries, similar to the PATH environment variable.
			   With that in mind, the wrapper adds a bunch of path determistic paths to the runtime dependencies. 
			   In the end, it imported our main function and execute it.
			   So, where is our script?
		- image 3: 
			   The line which executes mpv was changed to use the deterministic path for the mpv binary. 
		- image 4: 
			    However, Nix isn’t the one doing the paths replacement and wrapping on its own.
				Instead, it is our Nix file which defines the build process that does all of the legwork to make this happen.
				>> The Python wrapper is done by buildPythonApplication function from the Nix package collection (nixpkgs) which we used to build our package.
				>> The Qt 5 wrapper is done by wrapQtAppsHook component from nixpkgs which we included as a dependency.
		- image 5: 
	- https://notes.yukiisbo.red/posts/2022/01/Spice_up_with_Nix_Functional_Software_Deployment.html --> conceptualisation
	- https://serokell.io/blog/what-is-nix

* Flakes
	- https://notes.yukiisbo.red/posts/2021/08/Spice_up_with_Nix_Shells.html --> idea from niv
	- https://www.tweag.io/blog/2020-05-25-flakes/
