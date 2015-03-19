[![Rulp](https://img.shields.io/badge/Rulp-Ruby%20Linear%20Programming-blue.svg)](Rulp)
[![Gem Version](https://badge.fury.io/rb/rulp.svg)](http://badge.fury.io/rb/rulp)
[![Downloads](https://img.shields.io/gem/dt/rulp/stable.svg)](https://img.shields.io/gem/dt/rulp)
[![Inline docs](http://inch-ci.org/github/wouterken/rulp.svg?branch=master)](http://inch-ci.org/github/wouterken/rulp)
[![Codeship Status for wouterken/rulp](https://codeship.com/projects/f97c2f00-a4d2-0132-7283-026d769eacf9/status?branch=master)](https://codeship.com/projects/66508)


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Rulp](#rulp)
  - [Sample Code](#sample-code)
  - [Installation](#installation)
      - [Scip:](#scip)
      - [Coin Cbc:](#coin-cbc)
      - [GLPK:](#glpk)
  - [Usage](#usage)
      - [Variables](#variables)
    - [Variable Constraints](#variable-constraints)
    - [Problem constraints](#problem-constraints)
    - [Solving or saving 'lp' files](#solving-or-saving-lp-files)
      - [Saving LP files.](#saving-lp-files)
    - [Examples.](#examples)
    - [Rulp Executable](#rulp-executable)
    - [A larger example](#a-larger-example)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Rulp
Rulp is an easy to use Ruby DSL for generating linear programming and mixed integer programming problem descriptions in the LP file format.

The *[.lp]* file format can be read and executed by most LP solvers including coin-Cbc, Scip, GLPK, CPLEX and Gurobi.

Rulp will execute and parse the results generated by **Cbc**, **Scip** and **GLPK**.

Rulp is inspired by the ruby wrapper for the GLPK toolkit and the python LP library "Pulp".

## Sample Code

```ruby
	# maximize
	#   z = 10 * x + 6 * y + 4 * z
	#
	# subject to
	#   p:      x +     y +     z <= 100
	#   q: 10 * x + 4 * y + 5 * z <= 600
	#   r:  2 * x + 2 * y + 6 * z <= 300
	#
	# where all variables are non-negative integers
	#   x >= 0, y >= 0, z >= 0
	#


	given[

	X_i >= 0,
	Y_i >= 0,
	Z_i >= 0

	]

	result = Rulp::Max( 10 * X_i + 6 * Y_i + 4 * Z_i ) [
	                X_i +     Y_i +     Z_i <= 100,
	           10 * X_i + 4 * Y_i + 5 * Z_i <= 600,
	           2 *  X_i + 2 * Y_i + 6 * Z_i <= 300
	].solve

	##
	# 'result' is the result of the objective function.
	# You can retrieve the values of variables by using the 'value' method
	# E.g
	#   X_i.value == 32
	#   Y_i.value == 67
	#   Z_i.value == 0
	##
```



## Installation
To use Rulp as a complete solver toolkit you will need one of either Glpsol(GLPK), Scip or Coin-Cbc installed. Here are some sample instructions of how you install each of these solvers.
Be sure to read the license terms for each of these solvers before using them.

####Scip:

Go to install directory

	cd /usr/local

Download Scip source (Be sure to visit the SCIP website and check the license terms first.)

	curl -O http://scip.zib.de/download/release/scipoptsuite-3.1.1.tgz

Extract Scip source

	gunzip -c scipoptsuite-3.1.1.tgz  | tar xvf -

Scip relies on GMP to be installed. You can either install this or read the README for how
to compile scip without.
Example of installing GMP on mac:

	brew install gmp

Build Scip

	cd scipoptsuite-3.1.1
	make

Add scip bin directory to your path
E.g

	echo '\nexport PATH="$PATH:/usr/local/scipoptsuite-3.1.1/scip-3.1.1/bin"' >> ~/.bash_profile

Or

	echo '\nexport PATH="$PATH:/usr/local/scipoptsuite-3.1.1/scip-3.1.1/bin"' >> ~/.zshrc

You should now have scip installed.

####Coin Cbc:

Navigate to install location

	cd /usr/local

Follow CBC installation instructions

	svn co https://projects.coin-or.org/svn/Cbc/stable/2.8 coin-Cbc
	cd coin-Cbc
	./configure -C
	make
	make install

Add Coin-cbc to path

E.g

	echo '\nexport PATH="$PATH:/usr/local/coin-Cbc/bin"' >> ~/.bash_profile

Or

	echo '\nexport PATH="$PATH:/usr/local/coin-Cbc/bin"' >> ~/.zshrc

You should now have coin-Cbc installed.

####GLPK:
Download the latest version of GLPK from http://www.gnu.org/software/glpk/#downloading

From the download directory

	tar -xzf glpk-4.55.tar.gz
	cd glpk-4.55
	 ./configure --prefix=/usr/local
	make
	sudo make install

At this point, you should have GLPK installed. Verify it:

	which glpsol
	=> /usr/local/bin/glpsol

## Usage

#### Variables

```ruby
	# Rulp variables are initialized as soon as they are needed so there is no
	# need to initialize them.
	# They follow a naming convention that defines their type.
	# A variable is declared as a constant with one of three different suffixes.
	# 'f' or '_f' indicates a general variable (No constraints)
	# 'i' or '_i' indicates a integer variable
	# 'b' or '_b' indicates a binary/boolean variable


	An_Integer_i
	=> #<IV:0x007ffc4b651b80 @name="An_Integer">

	Generalf
	=> #<LV:0x007ffc4b651b80 @name="General">

	Bool_Val_b
	=> #<BV:0x007ffc4b67b6b0 @name="Bool_Val">

	# In some cases it is implausible to generate a unique name for every possible variable
	# as an LP problem description may contain many hundreds of variables.
	# To handle these scenarios variable definitions can
	# accept index parameters to create large ranges of unique variables.
	# Examples of how indexed variables can be declared are as follows:

	Item_i(4,5)
	#<IV:0x007ffc4b3ea518 @name="Item4_5">

	Item_i("store_3", "table_2")
	#<IV:0x007ffc4b3a3cd0 @name="Itemstore_3_table_2">

	[*0..10].map(&Unit_f)
	=> [#<LV:0x007ffc4cc25768 @name="Unit0">,
	 #<LV:0x007ffc4cc24cf0 @name="Unit1">,
	 #<LV:0x007ffc4cc0fc88 @name="Unit2">,
	 #<LV:0x007ffc4cc0f260 @name="Unit3">,
	 #<LV:0x007ffc4cc0ecc0 @name="Unit4">,
	 #<LV:0x007ffc4cc0e748 @name="Unit5">,
	 #<LV:0x007ffc4cc0df50 @name="Unit6">,
	 #<LV:0x007ffc4cc0d9d8 @name="Unit7">,
	 #<LV:0x007ffc4cc0d460 @name="Unit8">,
	 #<LV:0x007ffc4cc0cee8 @name="Unit9">,
	 #<LV:0x007ffc4cc0c970 @name="Unit10">]
```

### Variable Constraints

Add variable constraints to a variable using the <,>,<=,>=,== operators.
Be careful to use '==' and not '=' when expressing equality.
Constraints on a variable can only use numeric literals and not other variables.
Inter-variable constraints should be expressed as problem constrants. (Explained below.)

```ruby
	X_i < 5
	X_i.bounds
	=> "X <= 5"

	3 <= X_i < 15
	X_i.bounds
	=> "3 <= X <= 15"

	Y_f == 10
	Y_f.bounds
	=> "y = 10"
```

### Problem constraints

Constraints are added to a problem using the :[] syntax.

```ruby
	problem = Rulp::Max( 10 * X_i + 6 * Y_i + 4 * Z_i )

	problem[
		X_i +     Y_i +     Z_i <= 100
	]

	problem[
		10 * X_i + 4 * Y_i + 5 * Z_i <= 600
	]
	...
	problem.solve
```

You can add multiple constraints at once by comma separating them as seen in the earlier examples:

```ruby
	Rulp::Max( 10 * X_i + 6 * Y_i + 4 * Z_i ) [
		                X_i +     Y_i +     Z_i <= 100,
		           10 * X_i + 4 * Y_i + 5 * Z_i <= 600,
		           2 *  X_i + 2 * Y_i + 6 * Z_i <= 300
		          ]
```

### Solving or saving 'lp' files

There are multiple ways to solve the problem or output the problem to an 'lp' file.
Currently the Rulp library supports calling installed executables for Scip, Cbc and Glpk.
For each of these solvers it requires the solver command line binaries to be installed
such that the command `which [exec_name]` returns a path. (I.e they must be on your PATH. See [Installation](#installation)).

Given a problem there are multiple ways to initiate a solver.

```ruby
	@problem = Rulp::Max( 10 * X_i + 6 * Y_i + 4 * Z_i ) [
	                 X_i +     Y_i +     Z_i <= 100,
	           	10 * X_i + 4 * Y_i + 5 * Z_i <= 600,
	           	2 *  X_i + 2 * Y_i + 6 * Z_i <= 300
						]
```

Default solver:

```ruby
	@problem.solve
	# this will use the solver specified in the environment variable 'SOLVER' by default.
	# This can be 'scip', 'cbc', or 'glpk'. If no variable is given it uses scip as a default.
```

If you had a linear equation in a file named 'problem.rb' from the command line you could specify an alternate solver by executing:

```ruby
	SOLVER=cbc ruby problem.rb
```

Explicit solver:

```ruby
	@problem.scip
	# 	Or
	@problem.cbc
	#   Or
	@problem.glpk
```

Or

```ruby
	Rulp::Scip(@problem)
	Rulp::Glpk(@problem)
	Rulp::Cbc(@problem)
```

For debugging purposes you may wish to see the input and output files generated and consumed by Rulp.
To do this you can use the following extended syntax:

```ruby
	def solve_with(type, open_definition=false, open_solution=false)...
```

The optional booleans will optionally call the 'open' utility to open the problem definition or the solution. (This utility is installed by default on a mac and will not work if the utility is not on your PATH)

```ruby
	@problem.solve_with(SCIP, true, true)
```

#### Saving LP files.

You may not wish to use one of the RULP compatible but another solver that is able to read .lp files. (E.g CPLEX or Gurobi) but still want to use Rulp to generate your LP file. In this case you should use Rulp to output your lp problem description to a file of your choice. To do this simply use the following call

```ruby
	@problem.save("/Users/johndoe/Desktop/myproblem.lp")
```

OR
```ruby
	@problem.output("/Users/johndoe/Desktop/myproblem.lp")
```

You should also be able to call
```ruby
	@problem.save
```
Without parameters to be prompted for a save location.

### Examples.
Take a look at some basic examples in the ./examples directory in the source code.

### Rulp Executable
Rulp comes bundled with a 'rulp' executable which by default loads the rulp environment and either the Pry or Irb REPL.
(Preference for Pry). Once installed you should be able to simply execute 'rulp' to launch this rulp enabled REPL.
You can then play with and attempt LP and MIP problems straight from the command line.

```ruby

	[1] pry(main)> 13 <= X_i <= 45 # Declare integer variable
	=> X(i)[undefined]

	[2] pry(main)> -15 <= Y_f <= 15 # Declare float variable
	=> Y(f)[undefined]

	[3] pry(main)> @problem = Rulp::Min(X_i - Y_f) # Create min problem
	[info] Creating minimization problem
	=>
	Minimize
	 obj: X -Y
	Subject to
	0 X = 0
	Bounds
	 13 <= X <= 45
	 -15 <= Y <= 15
	General
	 X
	End

	[4] @problem[ X_i - 2 * Y_f < 40] #Adding a problem constraint
	=>
	Minimize
	 obj: X -Y
	Subject to
	  c0: X -2 Y <= 40
	Bounds
	 13 <= X <= 45
	 -15 <= Y <= 15
	General
	 X
	End


	[5] pry(main)> @problem.solve # Solve
	...
	[info] Solver took 0.12337
	[info] Parsing result
	=> -2.0 #(Minimal result)

	[6] pry(main)> Y_f # See value calculated for Y now that solver has run
	=> Y(f)[15.0]

	[8] pry(main)> X_i
	=> X(i)[13.0]

	# The result of the objective function (-2.0) was returned by the call to .solve
	# Now the solver has run and calculated values for our variables we can also test the
	# objective function (or any function) by calling evaluate on it.
	# E.g

	[9] (X_i - Y_f).evaluate
	=> -2.0

	[10] pry(main)> (2 * X_i + 15 * Y_f).evaluate
	=> 251.0
```

### A larger example
Here is a basic example of how Rulp can help you model problems with a large number of variables.
Suppose we are playing an IOS app which contains in-app purchases. Each of these in-app purchases costs
a variable amount and gives us a certain number of in-game points. Suppose our mother gave us $55 to spend.
We want to find the maximal number of in-game points we can buy using this money. Here is a simple example
of how we could use Rulp to formulate this problem.

We decide to model each of these possible purchases as a binary variable (as we either purchase them or
we don't. We can't partially purchase one.)

```ruby
# Generate the data randomly for this example.
costs, points = [*0..1000].map do |i|
	[Purchase_b(i) * Random.rand(1.0..3.0), Purchase_b(i) * Random.rand(5.0..10.0)]
end.transpose.map(&:sum) #We sum the array of points and array of costs to create a Rulp expression

# And this is where the magic happens!. We ask rulp to maximise the number of points given
# the constraint that costs must be less than $55

Rulp::Max(points)[
	costs < 55
].solve
=> 538.2125623353652 (# You will get a different value as data was generated randomly)

# Now how do we check which purchases were selected?
selected_purchases = [*0..1000].select do |i|
	Purchase_b(i).value
end.map(&Purchase_b)
=> [Purchase27(b)[true],
 Purchase86(b)[true],
 Purchase120(b)[true],
 Purchase141(b)[true],
 Purchase154(b)[true],
 ...
```

