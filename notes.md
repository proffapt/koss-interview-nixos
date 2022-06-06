## This is where i will write the content of script.md in short

* Slide 1
	Introduce yourself and the topic
* Slide 2
	options includes things like services to be enabled
	What is nix.. a package-manager.. an os.. or just a lang?.. give a brief speech on this

* The Nix OS
	- Read out the stuff from wiki
	Will look in detail for these two points in section 4, till then let's discuss the point 1
	- Instant runtime enviroment
		- line 1: declaring which interpreter to use for the script
		- line 2: telling the nix-shell which interpreter to use further.. and what packages will be needed
		- line 3: pinning the dependency being used.. nixpkgs in this case.
		What will nix do??
			It will prepare an enviroment with specified dependencies for the scripts to run
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

