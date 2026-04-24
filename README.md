# CSCI 489 — ADVANCED TOPICS IN COMPUTER SCIENCE | INTERPRETER PROJECT #

**Year:** Fall, 2014

**Instructor:**  _Professor R. Holliday_

**Text:**  There is no required text for this course. A standard 
reference book is “Compilers:  Principles, Techniques, and Tools” by 
Aho, Sethi, and Ulman (known familiarly as the “dragon book”). 


The purpose of this course is to acquaint the student with the problems 
involved in constructing language translators and with the tools that 
have been developed to aid in the translator-writing process.


The course outline is:
+	Introduction
+	Lexical Analysis
+	Syntactic Analysis
    +	Top-down parsing and recursive descent
    +	Bottom-up parsing and the simple precedence technique
  
+	Intermediate representations and semantic routines
    +	Reverse Polish
    +	Quadruples
    +	Semantic routines with bottom-up parsing
  
+	Interpreters
+	Simple code generation

 
Additional topics as time permits from among:
+	Symbol tables
+	Runtime storage allocation and organization
+	Simple code optimization
+	Error handling
+	Translator writing systems

## Interpreters ##

Compilers produce a machine code equivalent of the original source code.  Interpreters simply carry out the instructions prescribed by the source code.

Interpreters (as opposed to pure compilers) are often used:

	in a debugging environment
	for largely dynamic languages
	for languages that are much different than machine language

Interpreters tend to be much slower (10-20 times?) than compilers.

In this handout we consider interpreters for postfix intermediate code.

The basic idea applied to arithmetic expressions is:

1.  Scan postfix string form left to right using an initially empty stack of values.
2.  When an operand is scanned, stack its value
3.  When a k-ary operator is scanned apply the operator to the k top stack values and replace these values by the result.

Example:     a + (-b + c*d)   or   a b @ c d * + +

Suppose a = 1, b = 2, c =3 , d = 4
1+ (-2+3*4) = 1 + (-2 + 12) = 1 + (10) = 11
Stack:  	1	1 2	1 -2	1 -2 3	1 -2 3 4	1 -2 12		1 10	11

Now we want to extend this idea to other constructs as well.

For an assignment operator, we must stack the address of the left operand because the interpreter must change the value at that address.  A RESULT IS USUALLY NOT LEFT ON THE STACK.

A branch operation leaves no result on the stack either, but simply causes the interpreter to resume scanning from another point in the postfix string.

To see the outline of an interpreter, let's assume we have a postfix string P, indexed by pc (for postfix counter) and a runstack RS, indexed with rsc (for runstack counter).   Both of these are implemented as one-dimensional arrays.  We assume that rsc is pointing at the actual top of the stack, and the ps is pointing at the next item to be processed.  With these assumptions, the interpreter method is essentially one large switch statement.
void interpreter():   {

	switch (P[pc])  {
	
		case const:  RS[++rsc] = P[pc++] ; 
RS [++rsc]=P[pc++]; 
Break;

		case idr:  //same as above

		case plus:  if RS[rsc-3]==idr
				left = symTable[RS[rsc-2]].value
			          else
				left = RS[rsc-2]
			         if RS[rsc-1]==idr
				left = symTable[RS[rsc]].value
			         else
				left = RS[rsc]
			         RS[rsc-2] = left + right
			         RS[rsc-3]= const
			          rsc -= 2
			          pc++
			          break

		case asgn:  if RS[rsc-1]==idr
				symTable[RS[rsc-2]].value = symTable[RS[rsc]].value
			        else
				symTable[RS[rsc-2]].value = RS[rsc]
			        rs -= 4
			        pc++
			        break

		case BR:     pc = RS[rsc--]; rsc-- ; break    //pop the const code

		case BMZ:  if RS[rsc-2] <= 0
				pc = RS[rsc]
			        else
				pc++
			        rsc -= 4
			        break
		....

		default:  halt("illegal postfix code")
	
	}
}

 
Type issues

	Even if a language has only one type, there still might be more than one "type" of information on the runstack.  For example, we might want to keep track of whether we are interested in an identifier's value or its address (or position in the symbol table).    There are typically two ways to handle this issue.

Implicit Method--let the runstack have two fields.  When performing an operation, operands must be checked and converted (if necessary) and the "kind" field of the result must be set.   For example, the runstack for the assignment statement a:= b might look like this

runstack   a  		b 	 
'kind'         addr	val	


Explicit Method--in this method, the postfix generator inserts explicit conversion operations directly into the postfix string using unary operators like A (address), V (value), etc.  So the reverse polish string for an assignment statement a := b might look like this:

a  A  b  V  asgn


Arrays

	If the language supports arrays, then there needs to be some sort of "subscript" operator, SUBS, that returns an address.  Consider a reference like:

A[3, i+j] := 17

Then the postfix might look like this:

3	i	j	+	A	SUBS   	17 	ASGN

Remark:  Note the importance of A right before SUBS--we can look in the symbol table to confirm how many subscripts we need to process the array reference.

And if we have a statement like this:  B := A[ 3, i + j ] then we would also need an operator (let's call is CAV for convert address to value) that would let us know that we need to fetch the value at the given address.


## Lexical Analysis Summary ##

The compilation process can be divided into two major areas:  Analysis and Syntheses

The analysis phase consists of:

Lexical Analysis (Scanner):
breaks the source program up into atomic units called tokens;
often under control of the syntax analyzer

Syntax Analysis (parser):
	Makes sure that the tokens occur in an order permitted by the language specification (the language’s grammar)
	Typically constructs a parse tree

Semantic Analysis:
	Transforms the parse tree into some intermediate form (reverse Polish, quadruples)
	Checks context  sensitivity (i.e, type checking)

The synthesis phase consists of:

Code Optimization:  
	Attempts to produce an intermediate version of the source which is faster and/or smaller
	
Code Generation:
	Converts the intermediate code into machine language instructions.
	(An interpreter does not have this phase, but instead executes the intermediate code.)


Other “duties”:

Bookkeeping:  managing a symbol table; managing storage allocation; creating a listing

Error handling:  can occur in any phase; can be difficult


Some of the phases of a compiler can be automated. That is, a specification of the language to be compiled and the machine on which it is to be implemented are fed to a large program, called a “compiler-generator”, which produces some part of the compiler.  The most successful areas for this are in parser generator, and scanner generators.
The Scanner

The scanner is that part of the compiler that reads in the original source program characters and constructs the source token—identifiers, reserved words, constants, and delimiters, e.g., *, =, (, etc.

The scanner is usually written as a separate phase because of the need for efficiency.  That is, many compilers spend a great deal of time in the lexical analysis phase, to perhaps the scanner part could be written in machine code.


Scanner Functions:

Recognition of tokens

Construction of a symbol table/constant table
Reading and printing a listing of the source program
Skipping blanks (if blanks are insignificant)
Conversion of constants
Reporting lexical errors, i.e., unrecognizable keyword, illegal token
Recovering from lexical errors
Skipping over comments


One approach

Associate with each token an integer code.  The scanner then creates a list of integers which are the token codes corresponding to the source program.  Note that some tokens, like identifiers and constants, need at two-part code, where the second integer carries information about which identifier is being referenced or the value of a constant.


“Look-ahead”

Scanners may need to read beyond a token to recognize which token is being processed.  For example, if a language has tokens < and <=, then after reading the < symbol, the scanner must then read the next symbol to determine which of the two tokens is actually being scanned.  

Transition Diagrams

A design aid for writing scanners is the transition diagram for finite-state automata.  These diagrams are described briefly in the “Guide to Writing a Scanner” handout.
The use of these diagrams is essentially the basis for “scanner-generator” programs.


- - - ---

### GRADING INFORMATION ###
Lake Forest College requires each department to 
assess the learning of its seniors.  This is required 
by the College’s accrediting agency.  (Think of an 
accrediting agency as the College’s “auditors”).  
Every 10 years the College goes through a re-accreditation 
process.  As part of this process the accreditors 
examine the assessment reports from the various departments.
