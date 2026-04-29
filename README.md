# CSCI 489 — ADVANCED TOPICS IN COMPUTER SCIENCE | INTERPRETER PROJECT #

**Year:** Fall, 2014

**Instructor:**  _Professor R. Holliday_

**Text:**  There is no required text for this course. A standard reference book is “_Compilers:  Principles, Techniques, and Tools_” by **Aho, Sethi**, and **Ulman** (known familiarly as the “_dragon book_”). 


The purpose of this course is to acquaint the student with the problems involved in constructing language translators and with the tools that have been developed to aid in the translator-writing process.


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


- - - -

## Lexical Analysis Summary ##

The compilation process can be divided into two major areas:  **Analysis** and **Syntheses**

The analysis phase consists of:

**Lexical Analysis (Scanner):** breaks the source program up into atomic units called tokens;often under control of the syntax analyzer

**Syntax Analysis (parser):** makes sure that the tokens occur in an order permitted by the language specification (the language’s grammar). 	Typically constructs a parse tree.

**Semantic Analysis:** transforms the parse tree into some intermediate form (reverse Polish, quadruples). Checks context  sensitivity (i.e, type checking)


The synthesis phase consists of:

**Code Optimization:** attempts to produce an intermediate version of the source which is faster and/or smaller.
	
**Code Generation:**converts the intermediate code into machine language instructions (an interpreter does not have this phase, but instead executes the intermediate code).



**Other “duties”:**

+ **Bookkeeping:** managing a symbol table; managing storage allocation; creating a listing.

+ **Error handling:** can occur in any phase; can be difficult.



Some of the phases of a compiler can be automated. That is, a specification of the language to be compiled and the machine on which it is to be implemented are fed to a large program, called a “compiler-generator”, which produces some part of the compiler.  The most successful areas for this are in parser generator, and scanner generators.


##### The Scanner #####


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


- - - -

## Syntactic Analysis ##


### Top-Down Parsing ###


In top-down parsing, we try to build the syntax tree by starting at the root and working down to the leaves.  Essentially, we try to replace nonterminals (left-hand sides of production rules) with an appropriate right-hand-side of a production rule.

In a naïve approach, we simply pick an alternative right-hand-side.  If that choice turns out to be incorrect, we backtrack and try another alternative.  We can see a “bad” backtracking example if we try to parse idr + idr from the grammar that follows:

	E ::= T + E | T
	T ::= F *  T | F
	F ::= (E) | idr


This backtracking is highly inefficient and is, in fact, of exponential complexity in terms of the length of the string being parsed.  A compile needs to be of linear complexity to be useful.  

Also note that a “backtracking” parser might be difficult to implement, particularly if the parser and the scanner are working together as they often are, with the scanner under the control of the parser.  

	Suppose we rewrite the above grammar in perhaps a more natural way (that forces left-to-right evaluation of addition and multiplication):

	E ::= E + T | T
	T ::= T * F | F
	F ::= (E) | idr

Now when we try to parse idr + idr, we have a different problem (if we always try the first right-hand-side alternative).  Namely, the left recursion sends us into an infinite loop.

Thus, if we want a practical top-down parser (that builds the tree by scanning from left-to-right), we need to eliminate backtracking and left recursion.     We consider two different ways to avoid these problems.  One technique is to use theorems from “grammar theory”.  Another way is to allow additional descriptive devices for our grammar rules.

 
For the first technique, let’s look at the following result:

Theorem:  Given a context-free grammar, there is an algorithm that constructs an equivalent grammar without left-recursion.


We consider this result in steps:

Lemma 1:  Given a grammar with -productions (where  represents the null string), we can algorithmically construct an equivalent grammar with no  -productions.


Lemma 2:  Given a grammar with unit productions (of the form AB), we can construct an equivalent grammar without unit productions.


Lemma 3:  Given a grammar with direct left recursion, we can construct an equivalent grammar without left recursion.


Lemma 4:  Given a grammar with indirect left recursion, we can construct an equivalent grammar without indirect left recursion.



To get rid of backtracking, we essentially need to always choose the correct alternative on the first try on the basis of the next token.  Many programming languages are designed with this feature build in.  For example, in most programming languages, each of the different kinds of statements begins in a unique way.   For languages that don’t have this feature, we can use the technique of “left-factoring”.  For example, if a language has the production rules:  A   |  , we can replace these rules with A   A’ and A’   | .



	To see how we can alter our “grammar-writing conventions to deal with this problem, we look at something called extended-BNF notation.  (Regular BNF notation is the production-writing rules allowed in the definitions of grammars.  B stands for Backus, one of the language designers of FORTRAN, and N stands for Naur, one of the designers of Algol.)

 
In extended BNF we allow the following symbols:

{ }  -- arbitrary repetition of the enclosed element

[ ] – optional occurrence of the enclosed element

( )  -- alternative choices (separated by the | symbol)


Examples:

E  E + T | T  becomes E  T { + T }

<cond>  if <cond> then <st> fi | if <cond> then <st> else <st> fi

becomes

<cond>  if <cond> then <st>  [ else <st>  ] fi


U  Vx 
V  Uy | z

becomes

U  Vx
V  z { xy }


E  E + T | E – T | T is replaced by E  E {  ( + | - ) T }

	

	If we can write a grammar in such a way that we can always predict the correct alternative based on the next token, then the grammar is said to be an LL(1) grammar.    (The first L indicates that the string is being scanned from left to right.  The second L means we are building a leftmost derivation, which means we are replacing the leftmost nonterminal in each intermediate string.  The 1 means that we need one token of “lookahead”.  

 
Remarks:


1.	In general, there are LL(k) grammars that require k tokens of lookahead.


2.	LL(1) grammars may require a program terminator to that the parser knows when the program is over.


3.	A left-recursive grammar is not (LL1). But non left-recursive grammars need not be LL(1) either.


4.	There are standard transformation techniques to convert many grammars into LL(1) grammars.



Here are some theoretical results concerning LL(1) grammars:

“Negative”



Theorem:  There are context free languages that have no LL(1) grammar.


Theorem:  There is no algorithm that can determine whether an arbitrary context free language has an LL(1) grammar that defines it.



“Positive”


Theorem:  There is an algorithm that can determine whether an arbitrary context-free grammar is an LL(1) grammar.




Theorem:  There is an algorithm that can determine whether two arbitrary LL(1) grammars generate the same languages.


The Recursive Descent method

	Probably the most natural implementation

	One recursive method is written for each nonterminal symbol

	Might be useful to have a “terminator” symbol

	Any such algorithm will typically need a stack and a representation of the grammar

	The stack is managed by the recursive calls

	The methods represent the grammar

Example:  Simple arithmetic with + and *

E  T | E + T
T  F | T * F
F  idr | (E)

First rewrite in Extended-BNF:

E  T { + T }
T  F { *F  }
F  idr | (E)

void parse()		void E()			void T()
    tok = scan()	    	    T()				    F()
     E()			     while tok == plus		    while tok == star
				Tok = scan()			tok = scan()
				T()				F()
void F()
    if tok == idr
        tok = scan()
    else if tok == lpar
	      tok = scan()
	      E()
	      if tok != rpar
		error(“Missing Right Parenthesis”)
	      else
		tok = scan()
	else
	      error(“Illegal factor”)
Example:  Parse x + y

Calling structure:

Parse				Scans x
	E
		T
			F	Recognizes x , scans + and returns to T
				
		T		Not scanning *, returns to E
				
	E	   		Scanning +, scans y and calls T			
		T 
			F	Recognizes  y, scans (eoln?) and returns to T

		T

	E
Parse--accept


Example: Parse x + y * z

Parse				Scans x
	E
		T
			F	Recognizes x, scans +  and returns to T
		T		Not scanning *, returns to E
	E			Scanning +; scans  y, calls T
		T	
			F	Recognizes y, scans *, returns to T
		T		Scanning *, scans z, calls F
			F	Recognizes z, scans (eoln?), returns to T
		T
	E
Parse--accept

 
Example:  Parse (x+)y

Parse				
	E
		T
			F		recognizes (, scans x, calls E
				E
					T
						F  	recognizes x, scans +, returns to T
					T		not scanning *; returns to E
				E			scanning +; scans ), calls T
					T
						F	issues error (neither idr nor lpar)




Example:  Parse x*(y+z)

Parse   scans x
    E
        T
           F   recognizes x; scans *; returns to T
        T       scanning *; scans ( and calls F
            F    recognizes (, scans y and calls E
                E
                    T
                        F  recognizes y, scans +, returns to T
                    T
                E          recognizes +, scans z, calls T
	      T    
                       F   recognizes z, scans } and returns to T
                    T
                E          not scanning +, so E returns to F
           F         recognizes ) , scans (eoln?) and returns to T
       T
   E
Parse--accept


### Top-Down Parsing using a Parsing Table ###

	As an alternative to recursive descent parsing, one can write a parser making use of a parsing table.  The steps in the process can be automated, leading to the development of “parser-writing” programs, also called parser-generators.

Consider the following simple “arithmetic” grammar:
E  E + T | T
T  T * F | F
F  idr | (E)

Step 1:  Remove left recursion (using our standard technique)
E  TE’
E’  +TE’ | 
T  FT’
T’  *FT’ | 
F  (E) | idr

Step 2:  Compute the sets “FIRST” and “FOLLOW”

FIRST—the terminals that can begin a string derivable from a grammar symbol

FIRST(E) = FIRST(T) = FIRST(F) = { idr, ) }

FIRST(E’) = { + ,  }		FIRST(T’) = { *,  }


FOLLOW—terminals that can immediately follow a nonterminal in a derivation

FOLLOW(E) = FOLLOW(E’) = {  ), $ }

FOLLOW(T) = FOLLOW(T’) = { +, ) , $ }

FOLLOW(F) = ( *, +, ) $ }

(Note:  The “FIRST” sets can, at least in this example, be determined by inspection.

To determine follow of A, look for production rules with A on the right-hand-side.

If a production rule looks like this:  B YAX, then FOLLOW(A) contains FIRST(X)

If a production rule looks like this:  BYA, then FOLLOW(A) contains FOLLOW(B)

If a production rule looks like this:  BYAX where  is in FIRST X, then FOLLOW(A) contains FOLLOW(B)
Step 3:  

Construct the parsing table.  The rows are indexed by the nonterminals and the columns are indexed by the terminals and the endmarker.

The entries in the table are productions.  Given a production of the form A, place this into the table in row A and column x where x is any terminal in FIRST().

If  is in First(), then also place the production into the table in row A and column x where x is any terminal in FOLLOW(A).  

Following these two rules, we obtain the following table:



	idr		+		*		(		)		$

E	ETE’						ETE’

E’			E’+TE’					E’		E’

T	TFT’						TFT’

T’			T’e		T’*FT’			T’ 		T’		

F	Fidr						F(E)


The Parsing Algorithm:

Initialize the stack to $ and the start symbol (in this example, E), with the start symbol on top.

Compare the top of the stack X to the current scanned symbol a.
	If X = a = $, then accept
	If X = a ≠ $, then pop X and scan next symbol
	If X is a nonterminal and X ≠ a, then reject
	If X is a nonterminal, look in the table at row X, column a.  
		If no entry, then reject string
		If we find Xw1w2….wn, pop x and push wn…w2w1 (w1 on top)

 
Example:  Parse x*(y+z)
Stack			Symbol		Production
$E			x			ETE’

$E’T			x			TFT’

$E’T’F			x			Fidr

$E’T’x			x

$E’T’			*			T’*FT’

$E’T’F*			*

$E’T’F			(			F(E)

$E’T’)E(		(

$E’T’)E			y			ETE’

$E’T’)E’T		y			TFT’

$E’T’)E’T’F		y			Fidr

$E’T’)E’T’y		y			

$E’T’)E’T’		+			T’

$E’T’)E’		+			E’+TE’

$E’T’)E’T+		+

$E’T’)E’T		z			TFT’

$E’T’)E’T’F		z			Fidr

$E’T’)E’T’z		z		

$E’T’)E’T’		)			T’

$E’T’)E’		)			E’

$E’T’)			)		

$E’T’			$			T’

$E’			$			E’

$			$			accept



- - - - 

### Bottom-up Parsing ###

In general, bottom-up parsing attempts to construct a parse tree from an input string by beginning at the leaves and working up the root.  Essentially, as we scan, we look for a right-hand-side of a production rule (called a handle) that we can replace by its left-hand-side nonterminal.  Such a replacement is called a reduction.  So bottom-up parsing is performed by finding and reducing handles.

Bottom-up parsing often reverses a right-most derivation, that is, a derivation that replaces the rightmost nonterminal at every stage.  Because we reverse a right-most derivation, there are no nonterminals to the right of the handle.  This allows us to scan and parse simultaneously.

Simple precedence parsing

We define three relations on grammar symbols based only in terms of the grammar rules.

R = S if there exists a rule of the form U ::= …..RS…..

R < S if there exists a rule of the form U ::= ….. RV such that some string of terminals and/or nonterminals beginning with S is derivable from V in one or more steps.  Note that this requires V to be a nonterminal.

R > S if S is a terminal and there exists a rule of the form U ::=    …..VW….. such that some string ending in R is derivable from V in 1 or more steps (so V is a nonterminal) and some string beginning with S is derivable from W in 0 or more steps (so W may be a terminal or a nonterminal).

Example:

Z ::= bMb
M ::= (L | A
L ::= Ma)

=
 b = M	M=b
 ( = L	
 M = a   a = )

<
b < (	b < a
( < M   ( < (   ( < a

 
>
L > b	A > b	) > b
L > a	A > a	) > a

Let’s store these relations in a table:

	Z	b	M	L	a	(	)

Z	

b			=		<	<

M		=			=

L		>			>

a		>			>

(			<	=	<	<

)		>			>


A grammar is a simple-precedence grammar or a (1,1) precedence grammar if at most 1 relation holds between every pair of symbols.  The (1,1) refers to the fact that we use one symbol on each side of a possible handle to help us decide if it is a handle or not.  

Theorem:  All simple precedence grammars are unambiguous.  The unique handle of any sentential form S1….Sn is the leftmost substring Sj..Si such that 

Sj-1 < Sj = Sj+1 = … = Si > Si+1

The parsing algorithm:

Repeatedly scan tokens pushing them onto a parse stack until the top symbol > token being scanned.  At this point, the handle is in the stack--the last symbol of the handle is the top of the stack.   Using the above theorem, determine the first symbol of the handle.

Look up the handle in the grammar and replace it on the stack with the appropriate nonterminal.

Reduce again (if necessary)

Introduce a delimiter symbol, say #, which precedes and follows the source string.  Set # < all other symbols and all other symbols > #.

The parser accepts an input string when the stack contains # at the bottom with only the start symbol of the grammar above it, and the symbol being scanned is the terminating #.

Parse example:  using the previous table, show the parse of:  #b(aa)b#

Stack			Relation		Symbol Scanned

#			<			b

#<b			<			(

#<b<(			<			a

#<b<(<a		>			a

#<b<(<M		=			a

#<b<(<M<a		=			)

#<b<(<M<a=)	>			b

#<b<(=L		>			b

#<b=M		=			b

#<b=M=b		>			#

#<Z						#
		accept


Operator Precedence

One can also build precedence tables based on the desired precedence of operators.  That is, the relations aren’t determined by the grammar rules but by the rules of precedence.  In these situations, we are dealing with “operator precedence” grammars which have the property that there are no two adjacent nonterminals.

Let’s consider the following simple grammar:

E ::= E + E | E – E | E * E | E / E | E ^ E | (E) | idr
 
Here are the rules for generating the precedence table:

If 1 has higher precedence than 2, then set 1 < 2 and also set 2 < 1.

Significance:  In an expression like E 2 E 1 E, the parser will recognize E 1 E as the handle (i.e., will parse as E 2 (E 1 E)).

If 1 and 2 have equal precedence, then if the operators lave left associativity, make 1 > 2 and 2 > 1.  If the operators are right associative, make 1 < 2 and 2 < 1.

Significance:  In an expression like x / y / z, the parser will parse as (x/y)/z.  And in an expression like x ^ y ^ z, the parser will parse as x ^ (y ^ z).

Add the following relations:

 < id 		 < (		) > 
id > 		( < 		 > )

( = )		) > )		id > )
( < (
( < id

For the delimiter $:

$ < 		 > $
$ < id		id > $
$ < (		) > $

These last several relations allow $ to serve as the delimiter and also force both id to and (E) to get reduced to E whenever found.

Handling Unary Operators

If we have a unary operator, called !, then we use the following rules:

Make  < ! for any operator , whether unary or binary.  This ensures that E  !E is always parsed as E  (!E).

Make ! >  if ! has higher precedence than .  This will ensure that ! E  E will parse as (!E)  E.

Make ! <  if  has higher precedence than !.  This will ensure that ! E  E will parse as ! (E  E).

Example from preceding arithmetic grammar:

	+	-	*	/	^	id	(	)	$

+	>	>	<	<	<	<	<	<	>

-	>	>	<	<	<	<	<	<	>

*	>	>	>	>	<	<	<	>	>

/	>	>	>	>	<	<	<	>	>

^	>	>	>	>	<	<	<	>	>

id	>	>	>	>	>			>	>

(	<	<	<	<	<	<	<	=

)	>	>	>	>	>			>	>

$	<	<	<	<	<	<	<

 
Parse:  $ id – id ^ id / id $

Stack			Relation	Token Scanned

$ 			<		id
$ id			> 		-
$ E - 			<		id
$ E – id		> 		^
$ E – E			<		^
$E – E ^ 		< 		id
$E – E ^ id		>		/
$E – E ^ E		>		/
$E – E 			< 		/
$E – E /		<		id
$E – E /id		> 		$
$ E – E / E		>		$
$ E – E			> 		$
$ E			>		$
	accept

Note how the parse respects the precedence of operations, exponentiating first, then dividing, and finally subtracting.

Some final remarks:

1.	When a unary operator is also a binary operator (like the minus symbol), one approach is to let the lexical analyzer determine the difference between the two operators.  This might require the scanner to occasionally “back up”.

2.	In the early days of compiler writing, memory was very expensive and the amount of space for the two-dimensional table was a major concern.  There is a very nice mathematical development that uses graph theory to create  a pair of precedence functions.  These functions replace the matrix, requiring 2n storage locations instead of n2 locations.  

3.	As is typical in computer science, there is a tradeoff for using memory more efficiently.  And that is that the precedence functions don’t have any “holes” in the table.  These holes are helpful because they indicate syntax errors.  Of course, a parser using precedence functions will still accept/reject a string correctly, but it might take the parser several more steps to recognize the error.  (We will examine precedence functions later, if time permits.)



## Intermediate representations and semantic routines ##

CS 489--Quadruples (Part 1)

	An alternative (to postfix) form of intermediate code is called quadruples.  The general form of a quadruple is:  

	<operator>  <operand1>  <operand2>  <result>

Because quadruples are "closer" to machine code than postfix, they are often used for compilers.  The structure of quadruples also aids in code optimization because of the ease with which the order of the quadruples can be altered.

Example:  a * b might be represented as:  *   a   b   T, where T is some temporary location which stores the result of the multiplication.  

A typical sequence of quadruples will contain temporaries for most results.  Temporary locations will also appear in the operand positions for many quadruples.  And we should observe that in the case of a unary operator, one of the operand fields will be blank.

Example:  - (a*b + c*d)

	*	a	b	T1

	*	c	d	T2

	+	T1	T2	T3

	-		T3	T4        (this is unary minus)


As with postfix notation, we would like to generalize quadruples to handle other programming constructs.

BR	i	branch to the ith quadruple

BZ	i	P	branch to the ith quadruple if P is zero

:=		P1	P2	store the value of P1 into P2


CVIR		a	T	convert value of a from an integer to a real, store in T


SUBS	A	5	T	compute value of address of A[5] and store in T

 
Consider the following example:

	if  i > j then
		
		k := k + m*6

	else
	
		i := i + 1
	
	fi

The quadruple sequence might look like this:

1.  	-	i	j	T1

2.	BMZ		?	T1    //? needs to be fixed up

3.  	*	m	6	T2

4.  	+	k	T2	T3

5.  	:=		T3	k

6.  	BR		?		//? needs to be fixed up

7.  	+	i	1	T4

8.  	:= 		T4	i


	After fixup, first ? is 7.  After fixup, second ? is 9


Note how much closer to machine language the quadruple approach is.  If we could efficiently manage temporary locations, we could "easily" generate object code.


Let us investigate how to generate quadruples for arithmetic expressions using a bottom up parser.  This approach will mirror the approach that we followed for generating postfix using a bottom up parser.  (See last two pages of the handout Intermediate Code, Part 2.)
 
Consider the following simple arithmetic grammar, with the production rules numbered as indicated:

	E  T 1 | E + T  2 | E – T  3 | -T   4

	T  F   5  | T * F   6  | T / F    7

	F  idr    8  | (E)   9

Let's examine the attempted parse of the string a*(b+c).

As we parse, we would like to construct the following two quadruples:

	+	b	c	T1

	*	a	T1	T2

Following the bottom-up shift-reducing parser, we might expect the following parse stack sequence:

a
F
T
T*
T*(
T*(b
T*(F
T*(T
T*(E
T*(E+
T*(E+c
T*(E+F
T*(E+T
T*(E      
	Now it is at this time, when we do the reduction based on production rule 2, when it would be logical to generate the addition quadruple.  But notice what has happened.  We have replaced the variable names in the parse stack so we don't know what sort of addition quadruple to generate.  

	What we must do is save the variable names as "semantic information" associated with the nonterminals E, T, and F.

 
Instead of constructing this tree:

				E
			            /  \
			          E     \
			           |       \
          	T		          T        T
	 |		           |         |
      	F		          F        F
	|          		           |        |
       	a	 *             (       b +   c      )


We want to construct this tree:

				ET1
			            /  \
			          Eb    \
			           |        \
          	Ta		          Tb       Tc
	 |		           |          |
      	Fa		          Fb       Fc
	|          		           |          |
       	a	 *             (       b +   c      )


To accomplish this, we will keep a parse stack and a name stack concurrently.  Whenever we put a name token onto the parse stack, we put the symbol table position on the name stack at the same stack position.


Parse Stack:  	T	*	(	E	+	T

Name Stack:  	a			b		c


Now let's look at some (pseudo) code for generating quadruples.  As with generating postifx, we'll assume that the shift-reduce parser will call this routine whenever it is about to make a reduction.  The parser will pass the semantic analyzer the number of the production rule involved in the reduction.  

The parser will place items on the parse stack and will also manage the stack pointer (sp).  The semantic routine will maintain the name stack, will manage the temporary locations, and will generate the quadruples.

 
We'll assume that locations larger than 1000 are for temporary values.  To do this, we'll use a counter t, initialized to 1000.  We'll also assume that the quadruples are stored in a 2-dimensional array.  The counter qc will point to the next quadruple to be generated.

The first semantic routine simply generates the quadruples:

void generate(int opr, opd1, opd2, res)  {

   	quad[qc,1] = opr;
	quad[qc,2] = opd1;
	quad[qc,3] = opd2;
	quad[qc,4] = res;
	++qc:
}

The next routine is the main semantic analyzer, consisting of one lengthy switch statement.

void quadBuilder(int ruleNum)  {

switch (ruleNum)   {

	case 1:  break;

	case 2:  generate(plus, NS[sp-2], NS[sp], ++t) ; NS[sp-2]=t; break;

	case 3:  generate(minus, NS[sp-2], NS[sp], ++t) ; NS[sp-2]=t; break;

	case 4:  generate(uminus, 0, NS[sp], ++t) ; NS[sp-1]=t; break;

	case 5:  break;

	case 6:  generate(star, NS[sp-2], NS[sp], ++t) ; NS[sp-2]=t; break;

	case 7:  generate(slash, NS[sp-2], NS[sp], ++t) ; NS[sp-2]=t; break;

	case 8:  break;

	case 9:  NS[sp-2] = NS[sp-1]; break

	default:  halt("Invalid production rule number.")

       }
} 
As always, let us trace through the recognition of a string.  PS denotes the contents of the parse stack and NS denotes the contents of the name stack.  The production rule numbers are displayed after the step numbers.

a * (b + c)

1. 	PS:					11. 	PS:   T  *  (  E  +
	NS:						NS:   a          b

2.	PS:  a					12.	PS    T  *  (  E  +  c
	NS:  a						NS:   a          b      c

3.    8 	PS:  F					13.  8	PS:  T  *  (  E  +  F
	NS:  a						NS:  a          b      c

4.    5	PS:  T					14.  5	PS:  T  *  (  E  +  T
	NS:  a						NS:  a          b       c

5.  	PS:  T *				15.  2	PS:  T  *   (   E		+   b   c   T1
	NS:  a						NS:  a            T1

6.  	PS:  T * (				16.	PS:  T   *   (   E   )
	NS:  a						NS:  a             T1

7.  	PS:  T * ( b				17:  9	PS:  T   *   F
	NS:  a       b					NS:  a        T1

8.   8  	PS:  T * ( F				18:  6	PS:  T 			*   a   T1   T2
	NS:  a       b					NS:  T2

9.   5	PS:  T * ( T				19:  1	PS:  E
	NS:  a       b					NS:  T2

10.  1 	PS:  T * ( E
	NS:  a       b


CS 489--Quadruples (Part 2)

	In this handout we consider how to generate quadruples for conditional and loop constructs.  The quadruples will again be stored in a 2-dimensional table, with a qc pointing to the next quadruple to be generated.  We will use 3 extra stack fields in addition to the parse stack.  One of these will keep track of names, as in the previous handout, while the other two will help us keep track of branching quadruples.

Quadruples for Conditional statements

Suppose we have the following pertinent grammar rules:

<st1>  <if clause> <st2> fi | <if clause> <st2> else <st3> fi

<if clause>  if <expr> then

(Note:  For convenience we will assume that expression is integer valued, with 0 meaning false and 1 meaning true.)

If there is no “else” part, we want to generate the following:

	Quadruples for <expr> with result in T
	BZ  	n2	T	0
 	Quadruples for <st2>
n2	…..

Question:  When is the logical time to generate the BZ quadruple?  
Answer:  When if <expr> then is about to be reduced to <if clause>.  Of course, at this point, we don’t know the destination n2.

Also note that there will be a second reduction for the “if with no else”, namely when <if clause> <st2> fi is reduced to <st1>.  At this point, the quadruples for <st2> will have been generated, so we can “fixup” the BZ quadruple.

Here is a picture of the stack during these two reductions:

PS:	if	<expr>	then		Before first reduction
NS:		T
JS:
				sp

PS:	<if clause>				After first reduction
NS:
JS:	pos of BZ
	    sp
PS:	<if clause>	<st2>	fi		Before second reduction
NS:		
JS:
				sp

PS:	<st1>					After second reduction
NS:
JS:


Here are the code segments corresponding to the appropriate grammar rules:

<if clause>  if <expr> then:

	JS[sp-2] = qc;
	Generate(BZ, 0, NS[sp-1], 0);    // the first 0 needs to be fixed up


<st1>  <if clause> <st2> fi	:

	quad[ JS[sp-2], 2] = qc;


What if an else part is present?  Then we would like to generate the following sequence:

Part 1:  	quadruples for <expr> result in T

Part 2:		BZ 	n1	T	

Part 3:		quadruples for <st2>

Part 4:		BR	n2

Part 5:      n1    quadruples for <st3>
	     n2   …….

If we look at the tree corresponding to the grammar, we see that, in addition to the fixup issues, which we can handle, we have a more serious problem:

		<st1>
              /			\          \           \         \
 <if clause>         		  \ 	 \	 \        \
 /        |           \                                 \          \            \        \
if  <expr>  then		<st2>	else	<st3>	fi

The logical time to generate quadruples is when a handle is being reduced to its left-hand-side nonterminal.  We can generate Parts 1 and 2 during the first reduction.  Since there is only one more reduction, we must generate all the quads at that time.  But unfortunately, before the second reduction, the quads for <st2> and <st3> have already been generated by their respective actions and we had no chance to insert Part 4 between Parts 3 and 5.  In essence, the syntax of the grammar is too coarse.
A solution:  Refine the syntax to fit the semantics.

<st1>    <if clause> <st2> fi  | <fullCond> <st3> fi  

<fullCond>  <if clause> <st2> else

<if clause>  if <expr> then

Note that, in the case of an else clause (a full conditional), we have an extra reduction, which gives us an extra opportunity to generate quadruples.

Here are snapshots of the stack for the “full conditional” after the <if clause> reduction:

PS:	<if clause>				After <if clause> reduction
NS:		
JS:         pos of BZ
	sp

PS:	<if clause>	<st2>	else		Before <fullCond> reduction
NS:
JS:	pos of BZ
	    		              sp

PS:	<fullCond>				After <fullCond> reduction
NS:		
JS:        	pos of BR
	sp

PS:	<fullCond>	<st3>	fi		Before  <st1> reduction
NS:
JS:	pos of BR
				sp

PS:	<st1>					After  <st1> reduction
NS:
JS:	pos of BR
	sp


Here are the code segments corresponding to the appropriate grammar rules:

<fullCond>  <if clause> <st2> else:

	generate(BR, 0, 0, 0);
       	quad[ JS[sp-2] , 2 ] = qc;
	JS[ sp-2] = qc-1


<st1>  <fullCond> <st3> fi:	:

	quad[ JS[sp-2], 2] = qc;




Quadruples for Loop Statements

Now let’s do a similar analysis for a loop statement.  Consider a loop of the following form:

<st1>  for <var> := <expr1> step <expr2> until <expr3> do <st2> od


Semantics:  This loops executes like a typical “for” loop with the following explicit semantics:  

	<expr2> > 0

	<expr2> and <expr2> are evaluated before each repetition of the loop

Here is an appropriate sequence of quadruples for this construct:

	Quadruples for <var> := <expr1>
	BR	n2
n1	quadruples for <var> := <var> + <expr2>
n2	quads for T := <expr3>
	BP	n3	<var> 	T
	Quadruples for <st2>
	BR	n1
n3

 
With only one grammar rule, it is clear that the syntax is too course to be able to insert the jump quadruples.  So we refine the grammar to:

<for1>  for <var> := <expr1>

<for2>  <for1> step <expr2>

<for3>  <for2> until <expr3>

<st1>  <for3> do <st2> od


Here are snapshots of the parse stack and the other fields.  Note that we use two extra fields to keep track of the jump quadruples.


PS:	for	<var>		:=	<expr1>	Before <for1> reduction
NS:		symTable		Temp
JS1: 	
JS2:

					sp

PS:	<for1>						After <for1> reduction
NS:	symTable
JS1:	pos of 1st BR
JS2:     	n1
sp

PS:	<for1>		step	<expr2>		Before <for2> reduction
NS:	symTable		T’	
JS1:     	pos of 1st BR	
JS2:       n1
				sp

PS:	<for2>						After  <for2> reduction
NS:	symTable		
JS1:	pos of BP		
JS2:	n2
	sp

PS:	<for2>		until	<expr3>		Before  <for3> reduction
NS:	symTable		T’’
JS1:	pos of BP
JS2:	n1
				sp

PS:	<for3>						After  <for3> reduction
NS:	symTable		
JS1:	pos of BP		
JS2:	n1
	sp

PS:	<for3>		do	<st2>		od	Before  <st1> reduction
NS:	symTable		
JS1:	pos of BP
JS2:	n1
						sp

PS:	<st1>						After <st1> reduction
NS:
JS1:
JS2:
	sp


Here are the code segments corresponding to the appropriate grammar rules:

<for1>  for <var> := <expr1>

	generate( asgn, NS[sp], 0, NS[sp-2] );
	NS[sp-3] = NS[sp-2];
	JS1[sp-3] = qc;
	generate(BR, 0, 0, 0)
	JS2[sp-3] = qc


<for2>  <for1> step <expr2>

	generate( plus, NS[sp-2], NS[sp], NS[sp-2] )
	quad[ JS1[sp-2], 2 ] = qc;

<for3>  <for2> until <expr3>

	JS1[sp-2] = qc;
	generate( BP, 0, NS[sp-2], NS[sp] )

<st1>  <for3> do <st2> od

	generate( BR, JS2[sp-3], 0, 0 );
	quad [ JS1[sp-3], 2 ] = qc
 
The above code should generate the following pattern:

Quadruples for <expr1>, result in T

:=	T		<var>

BR	?				// fixed up as n2

n1	quads for <expr2>, result in T’

	+ 	<var> 	T’	<var>

n2	quads for <expr3>, result in T’’

	BP	?	<var>	T		// fixed up as n3

	Quadruples for <st2>

	BR	n1

n3


	
- - - -

## CS 489—Intermediate Code Generation ##

Once the scanner has created a file of tokens for the parser, the next step in the compilartion process is to produce intermediate code—that is code that is “between” the source code and the object code.  Intermediate code enables the compiler writer to perform certain operations (type conversion, optimization) that would be more difficult at either the source code level or the machine code level.

Since our project involves interpretation rather than compilation, the type of intermediate code that we’ll consider is postfix.  This is notation you should have seen with regular arithmetic.  Our job is to extend the notion of postfix to other programming constructs.

Postfix for Arithmetic

	Properties of Postfix
			
		Operations occur in execution order

		No parentheses are needed

		Operands occur in normal (infix) order

		Operators appears immediately after their operands

Examples:

Infix			Postfix

a*b			ab*

a*b+c			ab*c+

a*b+c*d		ab*cd*+

a*(b+c/d)		abcd/+*


Extending postfix to other constructs

Assignment:  this is very natural    a := b  becomes a  b  :=

Output:  Write a,b,c  becomes a Write  b  Write  c  Write

If/then/else:  we could “invent” a ternary operator, let’s call it ? as follows:

If e then st1 else st2  

would be translated as

e  st1  st2  ?


There is a drawback to this technique.  We can understand this after looking at execution of Postfix expressions.


Evaluation (interpretation) of Postfix

	What makes postfix an appropriate intermediate code for interpreters is the ease with which postfix expressions are evaluated.  The basic algorithm is this:

Push operands.
When an operator is encountered, pop the operands, perform the operation, and then push the result.


Once we understand how to interpret postfix, we should be able to see a potential problem with the ternary ? operator  defined earlier.

Consider this example:

If a > b then x := x + 1 else x := x – 1

If we produced the following postfix:

a b > x x 1 + := x x 1 - := ? 

then as we scanned and interpreted the postfix string, we would do both the increment and decrement of x, even though we should only do one of those operations.

Here is a suggested solution:  as we build the postfix (in a one-dimensional array), we will also make use of various “jump” operators. These operators will allow us to branch to various locations of the postfix string, allowing us to examine only the portions of the postfix string that require evaluations.


 We will look at examples of this technique for the conditional statement, the loop statements, and simple arithmetic.
 
Let us suppose we have jump operators BR (unconditional branch), BP (branch on positive), BM, BZ, BPZ, BMZ, BNZ (with the obvious interpretations).  

BR is a unary operator:  n  BR  yields an unconditional branch to position n of the postfix array

The other conditional jumps are binary operators:  a  n  BP yields a jump to position n if a is positive


Example—the conditional statement

Consider the following grammar rule:

<condSt> ::= if <expr> <relop> <expr> then <stGroup>  [ else <stGroup> ]  fi


Here is a recursive descent parsing method to recognize a conditional statement:

Condst()					Part 1:
  Tok = scan()						if tok == GTR 
  Expr()						  code = BMZ
  If tok != GTE && tok!= GT &&…			else if tok == GT
      Halt()						  code = BM…
  //see inserted code Part 1
  Tok = scan()					Part2:
  Expr()						postfix[i++] = minus
  // see inserted code Part 2				postfix[i++] = const
  if tok != kwThen					save1 = i
      Halt()						i++
  Tok = scan()						postfix[i++] = code
  stGroup()
  if tok == kwelse				Part 3:
    //  see inserted code Part 3			postfix[i++]=const
    tok = scan()						save2 = i
    stGroup()						i++
    //  see inserted code Part 4			postfix[i++] = BR
  if tok!= kwendif					postfix[save1]=i
    Halt()						
  Tok = scan()					Part 4:													postfix[save2] = i
 
Let’s consider a specific example.  Suppose we are using the code 1 for an identifier and the code 2 for a numeric constant.  Also suppose we have 4 variables in our symbol table, i, j, k,  and m, occupying positions 1 through 4, respectively.


Consider the following statement:

if   i > j then

      k := k + m*6

else

      i := i+ 1

fi



Following the code on the previous page, where the postfix is built using the ‘inserted” code sections, we get the following equivalent postfix string for this statement:


1   1   1   2   -   2   ?   BMZ   //   ? to be fixed up later



1   3   1   3   1   4   2   6   *      asgn



2   ?   BR   1   1   1   1   2   1   +   asgn   // first ? assigned 23 and second ? assigned 31

 
Example:  The Loop Statement

Consider the following grammar rule:  

<loopSt> ::= for <idr> = <expr> to <expr> loop <stGroup> endloop


Here is a recursive descent parsing method to recognize the loop statement:

loopSt()					Part 1:
  tok = scan()						postfix[i++]=1
  if tok != idr						posfix[i++] = posn
    halt()						save1 = posn
  //see inserted code, Part 1				
  tok = scan()
  if tok != asgn					Part2:
    halt()						postfix[i++]= asgn
  tok = scan()						save2 = i
  expr()						postfix[i++] = idr
  //see inserted code, Part 2				postfix[i++] = save1 
  if tok != kwTo
    halt()					Part3:
  tok = scan()						postfix[i++]= subt
  expr()						save3 = i
  //see inserted code, Part 3				i++
  if tok != kwLoop					postfix[i++] = BM
    halt()
  tok = scan()					Part 4:
  stGroup()						postfix[i++] = idr
  // see inserted code, Part 4			postfix[i++] = save1		
if tok != kwEndloop					postfix[i++] = idr
  halt()							postfix[i++] = save1
tok = scan()						postfix[i++] = const
							postfix[i++] = 1										postfix[i++] = plus
							postfix[i++] = asgn
							postfix[i++] = save2
							postfix[i++] = BR
							postfix[save3] = i

 
Now let’s do a specific loop example.   Suppose we are using the code 1 for an identifier and the code 2 for a numeric constant.  Also suppose we have 5 variables in our symbol table, a, b, c, d,  and x, occupying positions 1 through 5, respectively.



Consider the following statement:


for x := a+b to c-d loop

	print x, x*x

endloop



Following the code on the previous page, where the postfix is built using the ‘inserted” code sections, we get the following equivalent postfix string for this statement:



x   a   b   +   :=   x   c   d   -   -   ?   BM     // ? to be fixed up later



x   print   x   x   *   print   newline



x   x   1   +   :=   ?   BR  //  first ? assigned 27; second ? assigned 6


CS 489—Intermediate Code Generation (Part 2)


In this handout, we consider generating postfix for arithmetic expressions.  An advantage of using recursive descent is that the recursion helps the parser generate the postfix code in the proper order.

Consider the following simple grammar:

	E  T | E + T | E – T | -T

	T  F | T * F | T / F

	F  idr | (E)

Rewriting in Extended BNF yields:

	E  [ - ] T { ( + | - ) T }

	T  F { ( *  | / )  F }

	F  idr | (E)


Here are the parsing recognition methods:

E()

    If tok = minus
      Tok = scan()
       T1()
        // see inserted code, part1			postfix[i++]= uminus

    else
        T2()

    While (tok == plus) || (tok == minus)
        //see inserted code, part 2			oper = tok  // oper is local var
        tok = scan()
        T3()
         //  see inserted code, part 3			postfix[i++]= oper

 
T()
  
    F1 ()

    While (tok == star) _|| (tok == dvd) 
      //see inserted code part 1			oper = tok // oper is local var
        tok = scan()
        F2()
        //  see inserted code, part 2			postfix[i++] = oper


F()

  If tok == idr

           // see inserted code part 1			postfix[i++] = tok
	Tok = scan()

         // see inserted code part 2			postfix[i++]= tok
	Tok = scan()

  Else

	If tok != lpar
	    Halt(“Missing left parenthesis”)

	Tok = scan()

	E()

	If tok != rpar
	    Halt(“Missing right parenthesis”)
	
	Tok = scan()

 
Let us consider that parse of the following expression:  a – b * ( - c / d )


E			oper  -

	T2
		F1
	T2

E

	T3			oper  *
		F1
	T3

		F1
			E
				
T1  		oper  /
					F1
				T1
					F2
				T1			(inserts / into postfix)

			E			(inserts @ into postfix)
		F2
	T3				(inserts * into postfix)

E			(inserts – into postfix)


Here is the postfix code that is produced:

a    b    c    d    /    @    *     -

Note that this string provides the correct evaluation:

	First divide c by d
	Negate the quotient
	Multiply result by b
	Subtract this result from a
 
Observe that unary minus does not have precedence over * or / .  We can tell this by noticing that unary minus in introduced into the grammar “sooner” than multiplication and division.  In the interaction of multiplication and division this typically won’t matter.  

But what about a string like  - x – y?  There is some inherent ambiguity.  Should this be parsed as – (x-y) or as (-x) – y.  

The original grammar makes it clear (try to build the parse tree) that we should apply the unary minus to x before performing the subtraction.  However, the revised (extended BNF) grammar doesn’t make this clear.  

However, the parsing routines do in fact work correctly.  If we trace the above string through the previous methods, we do in fact get the following postfix string:  

x  @  y  -


Suppose we wanted unary minue to have precedence over the other operations.  We could accomplish this by changing the grammar:


	E  [ - ] T { ( + | - ) T }

	T  F { ( *  | / )  F }

	F  idr | (E)

Becomes


	E   T { ( + | - ) T }

	T  F { ( *  | / )  F }

	F  idr | (E) | - F

We would also need to make a simple change to the recognition methods.  Specifically, the statement that inserts unary minus into the postfix string  would be removed from the E() method and placed in the F() method.  

With the first grammar above, the string – a * b would result in a postfix equivalent of:    a      b    *   @

With the second grammar above, the string – a * b would result in a postfix equivalent of:  a   @   b   *
Finally, we take a brief look at how intermediate code can be generated within a bottom-up parser.  Recall that a bottom-up parser makes use of a precedence table.  A typical algorithm finds the handle in the parse stack (a right-hand-side of a production rule) and then replaces the handle with left-hand-side nonterminal of the appropriate production rule that is found in the parsing table.

Example:  Let’s return to our original grammar and number the production rules:

	E  T 1 | E + T  2 | E – T  3 | -T   4

	T  F   5  | T * F   6  | T / F    7

	F  idr    8  | (E)   9

The postfix generator can now be encapsulated in a single method.  This method is called by the parser.  Before the parser makes a reduction, it looks up the number of the production rule involved and calls the postfix generator method with a single argument, namely the production rule.

The postfix generator method can be simply written as follows:

postfixGen(int ruleNum):

   switch (ruleNum) {

	case 1:  break;

	case 2:  postfix[i++]=plus;  break;

	case 3:  postfix[i++]=minus;  break;

	case 4:  postfix[i++]=uminus;  break;

	case 5:  break;

	case 6:  postfix[i++]=star;  break;

	case 7:  postfix[i++]=dvd;  break;

	case 8:  postfix[i++]=tok;  tok = scan; postfix[i++] = tok;  break;

	case 9:  break;

	default:  halt(“invalid production rule”)
}
 
Example:  Trace the pars of:   a  *  ( b + c )

Stack		Rule Number		Postfix String

a		8			a

F		5			a

T					a

T*					a

T*(					a

T*(b		8			a  b

T*(F		5			a  b

T*(T		1			a  b

T*(E					a  b

T*(E+					a  b

T*(E+c		8			a   b   c

T*(E+F	5			a   b   c

T*(E+T	2			a   b   c   +

T*(E					a   b   c   +

T*(E)		9			a   b   c   +

T*F		6			a   b   c   +   *

T		1			a   b   c   +   *

E		accept			a   b   c   +   *




## Finite-State Automata | A Design Aid for Scanners
##


  One possible focus of the Theory of Computation concerns abstract machines that recognize “languages”, that is, sets of strings that satisfy a certain property (e.g., contain a certain patter or obey the syntax rules of Java).  Since a compiler is also, at least on some level, a string recognizer, it’s not surprising that concepts from Theory of Computation carry over into Compiler Writing.

A finite state automaton is a “transition diagram” consisting of circles (representing states) and labeled arcs (representing scanned data).  If the data scanned leads to an accepting state (drawn as a double circle), then we say that the FSA accepts the string.  The start state is denoted usually with an S or with an arrow leading to it that isn’t coming from another state.  Examples of FSAs are provided on the accompanying PDF.  These examples demonstrate how a scanner can be developed from an FSA (which is why automated scanner-generators are possible).


Introduction to Grammars and Parsing

	Once the scanner has broken the source program into a sequence of tokens, it is up to the parser to perform syntactic analysis on the tokens to see that the sequence is legal.  That is, the tokens obey the syntax rules of the language.  In terms of grammars, this means that the terminal string can be derived from the grammar rules.  Parsing techniques fall into two broad categories—top-down and bottom-up.  Before we examine parsing techniques, we first need to understand grammars.

Def:  A regular grammar is a 4-tuple [N,T,P,S] where 
N is a finite set (of nonterminals), 
T is a finite set of terminals, where N and T are disjoint, 
P is a finite set of production rules of the form a b where a is a nonterminal and b is either a terminal or a terminal followed by a nonterminal
S is a nonterminal designated as the start symbol of the grammar.

Here is a grammar that generates strings of 0s and 1s with an odd number of 1s:

S0S|1A|1
A0A|1S

If one draws the FSA corresponding to this grammar, it is easy to see the connection between FSAs and regular grammars.  In fact, there is a theorem in Computability Theory which asserts that a language is recognized by a FSA if and only if it can be generated by a regular grammar.

Regular grammars are not powerful enough to generate the sort of language that is useful in computer science.  For example, regular grammars can’t generate the language that consists of a string of a’s followed by a string of an equal number of b’s.  Compilers need to be able to recognize such languages—this is easy to see when we consider that compilers must be able to match left parentheses with right parentheses.

There is a more powerful class of grammars that can generate (for the most part) the kind of languages that model programming languages.  

Def:  A context-free grammar is a 4-tuple [N,T,P,S] where 
N is a finite set (of nonterminals), 
T is a finite set of terminals, where N and T are disjoint, 
P is a finite set of production rules of the form a b where a is a nonterminal and b is either a finite string of terminals and/or nonterminals.
S is a nonterminal designated as the start symbol of the grammar.


Here is a simple CFC that generates anbn:

S-> aSb | ab

Here is a simple (and ambiguous) “arithmetic” grammar.

EE+E | E*E | (E) | x | y | x

This grammar is ambiguous because the string x+y*z can be derived in two different ways (it has two different “parse” trees).

But we know that the arithmetic language is not ambiguous.  Here is an unambiguous grammar that uses left-recursion (to force left-to-right evaluation) and “levels” (to impose a precedence of operations):

E  E + T | T
T  T + F | F
F ::= x | y | z | (E)



- - - - 

## Interpreters ##

We all know that compilers produce a machine code equivalent of the original source code.  Interpreters simply carry out the instructions prescribed by the source code. You can think of the traditional _Java Compiler_ versus the _Python Interpreter_. Ofcourse, it is now possible to also interpret Java code rather than compile it, making Java more like Python and Javascript.


Interpreters (as opposed to pure compilers) are often used:

  + in a debugging environment
  + for largely dynamic languages
  + for languages that are much different than machine language

Interpreters tend to be much slower (10-20 times?) than compilers. This is because compilers compile to assembly language which is machine native language and it is executed much faster.

In this course we considered interpreters for _postfix intermediate code_.

The basic idea applied to arithmetic expressions is:

1.  Scan postfix string from left to right using an initially empty stack of values.
2.  When an operand is scanned, stack its value.
3.  When a k-ary operator is scanned apply the operator to the k top stack values and replace these values by the result.

**Example:** 

    a + (-b + c*d)   or   a b @ c d * + +
    
_Suppose a = 1, b = 2, c =3 , d = 4_
    
	1+ (-2+3*4) = 1 + (-2 + 12) = 1 + (10) = 11
	
_Stack: this part shows what the stack during operation looks like._ 

    1    1 2    1 -2    1 -2 3    1 -2 3 4    1 -2 12    1 10    11

Now, we want to extend this idea to other constructs as well.

For an assignment operator, we must stack the address of the left operand because the interpreter must change the value at that address.  A RESULT IS USUALLY NOT LEFT ON THE STACK. A branch operation leaves no result on the stack either, but simply causes the interpreter to resume scanning from another point in the postfix string.

To see the outline of an interpreter, let's assume we have a postfix string P, indexed by postfix counter (pc) and a runstack RS, indexed with runstack counter (rsc). Both of these are implemented as one-dimensional arrays.  We assume that rsc is pointing at the actual top of the stack, and the pc is pointing at the next item to be processed.  With these assumptions, the interpreter method is essentially one large switch statement.

    void interpreter():   {
	    switch (P[pc])  {
		    case const:
			    RS[++rsc] = P[pc++];
				RS[++rsc] = P[pc++];
				break;
			case idr:  //same as above
			case plus:
			    if RS[rsc-3]==idr
				    left = symTable[RS[rsc-2]].value;
				else
				    left = RS[rsc-2];
			    
				if RS[rsc-1]==idr
				    left = symTable[RS[rsc]].value;
			    else
				    left = RS[rsc];
				
				RS[rsc-2] = left + right;
				RS[rsc-3]= const;
				rsc -= 2;
				pc++;
				break;
			case asgn:
			    if RS[rsc-1]==idr
				    symTable[RS[rsc-2]].value = symTable[RS[rsc]].value;
				else
				    symTable[RS[rsc-2]].value = RS[rsc];
			    rs -= 4;
			    pc++;
			    break;
			case BR:
			    pc = RS[rsc--];
				rsc-- ; 
				break;    //pop the const code
			case BMZ:
			    if RS[rsc-2] <= 0
				    pc = RS[rsc];
				else
				    pc++;
			        rsc -= 4;
			        break;
					
			....
			default:  halt("illegal postfix code")
        }
    }

 
##### Type issues #####

Even if a language has only one type, there still might be more than one "type" of information on the runstack.  For example, we might want to keep track of whether we are interested in an identifier's value or it's address (or position in the symbol table). There are typically two ways to handle this issue.

**Implicit Method** -- let the runstack have two fields. When performing an operation, operands must be checked and converted (if necessary) and the "kind" field of the result must be set. For example, the runstack for the assignment statement a:= b might look like this

    runstack   a    b 	 
    'kind'     addr    val	


**Explicit Method** -- in this method, the postfix generator inserts explicit conversion operations directly into the postfix string using unary operators like A (address), V (value), etc.  So the reverse polish string for an assignment statement a := b might look like this:

    a  A  b  V  asgn


##### Arrays #####

If the language supports arrays, then there needs to be some sort of "subscript" operator, _SUBS_, that returns an address.  Consider a reference like:

    A[3, i+j] := 17

Then the postfix might look like this:

    3	i	j	+	A	SUBS   	17 	ASGN

**Remark:**  Note the importance of _A_ right before _SUBS_ -- we can look in the symbol table to confirm how many subscripts we need to process the array reference.

And if we have a statement like this:

    B := A[ 3, i + j ] 

then we would also need an operator (let's call it _CAV_ for convert address to value) that would let us know that we need to fetch the value at the given address.



- - - -

CS 489—SCANNER-WRITING GUIDE
FALL, 2014
INTRODUCTION
The scanner is the portion of the compiler that reads language specific tokens in the source code. Once the scanner determines the token, it associates a corresponding integer code to that token. The scanner might return this to a file writer (or write directly to a file).  If the scanner is under the control of the parser, it would return this token code to the parser.  
Only the scanner deals with the characters of the source code.  
The rest of the compiler works with the integer codes. We should first enumerate all the tokens of the “discussion language”.  For clarity and readability, it is probably a good idea to use a mnemonic identifier for each token.  For example, if the token code for a semicolon is 12, once the scanner recognizes a semicolon, it would be better to write something like tok = SEMI rather than tok = 12. Where SEMI is a suitably defined constant:  
	final int SEMI = 12;
Token		Code	Mnemonic	Token	Code	Mnemonic	
identifier		1		IDR	:=			14	ASGN
constant	2	CONST	+	15	PLUS
read	3	KWRD	-	16	MINUS
write	4	KWWR	*	17	STAR
if	5	KWIF	/	18	DVD
then	6	KWTH	=	19	EQR
else	7	KWEL	>	20	GTR
fi	8	KWFI	<	21	LTR
to	9	KWTO	<=	22	LER
do	10	KWDO	>=	23	GER
endloop	11	KWENDL	#	24	NER
;	12	SEMI	(	25	LPAR
,	13	COMMA	)	26	RPAR
If we had a program that began:  
Read n;
To n do
    F := 1; I := 0 ….

The scanner should produce the following succession of “tok” values:
	3  1  12   9  1  10  1  14  2  12 …
Identifiers and constants (really a value) would become two-part codes. To design the scanner, we will employ the technique of the state diagram.
•	States represented by circles
•	S is the start state
•	Arcs are labeled with a single character
•	Moving along an arc corresponds to scanning a character along the arc
•	One unlabeled arc may appear; move along this arc if the scanned character appears on no other arc
•	If we are unable to leave a state, there is a lexical error in the program
We will employ a simple error handler, namely print an error message and continue. Suppose ch is a character variable that holds the current input character. Since a scanner is supposed to create a listing, we might as well print the listing while we are scanning.  That is, when we read a value into ch, we also print it.
Observe that to detect the end of a token, the first character of the following the current input character might need to be examined, as is the case in distinguishing between < and <=.  For consistency, we will always read this next character before returning from the scanner.  So when we enter the scanner each time, the first character of the next token will have been read.  One consequence of this—we should get the first character of the source code before our original call to the scanner. Assuming a couple of global variables, ch and tok, and a method getChar() which reads the next nonblank character, what follows is a very basic outline in pseudocode of a scanner for the discussion language:

main()
	ch = getChar()
	while (ch != eof)
		tok = scanner(ch)
                          outFile.write(tok)

getChar()

scanner(ch)
	if ch >= ‘a’ && ch<= ‘z’    // identifier or keyword
		str = ch
   		while (ch>=’a’ && ch<=’z’) || (ch>=’0’ && ch <= ‘9’)
			ch = getChar()
                                        str = str + ch
                           for (j ==0; j<9; ++j)         //checking for keyword
			if str = kwTable[j]
                                              tok = kwToken[j]
       			      kWord = true
 			      break
		if !(kWord)
 			tok = idr
	else if ch == ‘;’
		tok = semi
		ch = getChar()
	.
	.
	.
	else if ch == ‘(‘
		tok = lpar
		ch = getChar()

	else if ch == ‘<’
		ch = getChar()
		if  ch == ‘=’
		      ch = getChar()
		      tok = ler
		else
		      tok = ltr
	.
	.
	.
	else
		error(“Unrecognizable token”)

			


- - - ---

### GRADING INFORMATION ###
Lake Forest College requires each department to 
assess the learning of its seniors.  This is required 
by the College’s accrediting agency.  (Think of an 
accrediting agency as the College’s “auditors”).  
Every 10 years the College goes through a re-accreditation 
process.  As part of this process the accreditors 
examine the assessment reports from the various departments.
