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
	- Diving deep into Instatiation and Realization
		First, Nix expressions are translated to derivations in a process called “instatiation”. 
	 	- image:
			It’s a rather massive JSON file. Let’s break it down. 
			- The outputs
				The outputs section defines the deterministic path of the output of this derivation. The process is simply called “building”.
				Nix builds the derivation and then store its outputs to the path defined here. 
			- The inputs
				* InputSrcs defines the path of the source code within the Nix store and a builder script which will be executed. 
				* InputDrvs defines the inputs or dependencies to build this derivation.
				  It mentions other components via their respective derivations (note the .drv prefix).
				  You might be wondering why some have out, dev, lib or even multiple of them.
				  That’s because derivations can have multiple outputs and the list with out, dev tells Nix which outputs of the derivations to use.
				  You can see how it compares by running nix show-derivation /nix/store/457yxakhv0lc3df8jlb9cz0q8fk44lns-qtbase-5.15.2.drv but I won’t be getting into it in here. 
		    - The enviroment variables
					Here we see system, builder, and args, what’s up with these?
					You might be able to guess on your own but here’s an explaination:
					system defines which exact architecture/operating system this derivation is for.
					In the early days of Nix, builder was originally used to defined the script used to build the derivation.
					This script was made along with the expression but this was changed in later versions to use a generic builder script which we’ll see later.
					Nowadays, it always points to the bash shell interpreter.
					args defines the command arguments to pass to the builder. Because of the change mentioned before, this simply tells the bash interpreter to execute the generic builder script.
					env defines all of the environment variable that will be setup for the builder.
					To ensure reproducibility, before running the build script, Nix prepares a clean slate which involves removing all environment variables.
			- Section from package script(post patch prefixup)
				Basically we did by ourselves for mpv what nix did for other packages.. that is specifying the exact location of packages..

* Flakes
	- Nixpkgs is a nix channel which is similar to the list of repositories that can be found in other package managers (ex. /etc/apt/sources.list in Debian/Ubuntu).
	  This is all well and good but there’s you will encounter the following issue: Each Nix user may have different versions nixpkgs present. 
	- Slow evaluation
		Nix uses a simple, interpreted, purely functional language to describe package dependency graphs and NixOS system configurations. 
		So to get any information about those things, Nix first needs to evaluate a substantial Nix program. This involves parsing potentially 
		thousands of .nix files and running a Turing-complete language.
		Evaluating individual packages or configurations can also be slow -> as shown in output of second command.
	- Creating your own flake
		Add flake.nix file to your package folder and done.
		What does that file contain, let's have a look into that
	- Flake output
		> Flakes can provide arbitrary Nix values, such as packages, 
		NixOS modules or library functions. These are called its outputs. 
		We can see the outputs of a flake as shown on the left image.
	- Flake metadata
		> flakes have to declare their dependencies explicitly, and these dependencies have to be locked to specific revisions.
		In order to do so, flake.nix file declares an explicit dependency on Nixpkgs, which is also a flake.
	- Flake.nix
		The description attribute is a one-line description shown by nix flake metadata.
		The inputs attribute specifies other flakes that this flake depends on. These are fetched by Nix and passed as arguments to the outputs function.
		The outputs attribute is the heart of the flake: it’s a function that produces an attribute set. The function arguments are the flakes specified in inputs.
		The self argument denotes this flake. Its primarily useful for referring to the source of the flake (as in src = self;) or to other outputs (e.g. self.defaultPackage.x86_64-linux).
		The attributes produced by outputs are arbitrary values, except that (as we saw above) there are some standard outputs such as defaultPackage.${system}.
		Every flake has some metadata, such as self.lastModifiedDate, which is used to generate a version string like hello-20191015.
	- Flake.lock
		You may have noticed that the dependency specification github:NixOS/nixpkgs/nixos-20.03 is imprecise: it says that we want to use the nixos-20.03 branch of 
		Nixpkgs, but doesn’t say which Git revision. This seems bad for reproducibility. However, when we ran nix build, Nix automatically generated a lock file that 
		precisely states which revision of nixpkgs to use
	- Hermatic evaluation
		
	- 
		Any subsequent build of this flake will use the version of nixpkgs recorded in the lock file. 
		If you add new inputs to flake.nix, when you run any command such as nix build, Nix will 
		automatically add corresponding locks to flake.lock. However, it won’t replace existing locks. 
		If you want to update a locked input to the latest version, you need to ask for it:
