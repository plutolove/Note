##### 语法定义
```
program    --->   declaration* EOF ;

declaration   ---> funDecl;



funDecl  --->  "fn" function ;

statement      → varStmt
			   | exprStmt
               | forStmt
               | ifStmt
               | printStmt
               | returnStmt
               | whileStmt
               | block ;


varStmt  --->  "var" IDENTIFIER ":" IDENTIFIER ( "=" expression )? ";" ;

exprStmt       → expression ";" ;

forStmt        → "for" "(" ( varDecl | exprStmt | ";" )
                           expression? ";"
                           expression? ")" statement ;

ifStmt         → "if" "(" expression ")" statement
                 ( "else" statement )? ;

printStmt      → "print" expression ";" ;

returnStmt     → "return" expression? ";" ;

whileStmt      → "while" "(" expression ")" statement ;

block          → "{" statement* "}" ;


expression     → assignment ;

assignment     → ( call "." )? IDENTIFIER "=" assignment
               | logic_or ;

logic_or       → logic_and ( "or" logic_and )* ;
logic_and      → equality ( "and" equality )* ;
equality       → comparison ( ( "!=" | "==" ) comparison )* ;
comparison     → term ( ( ">" | ">=" | "<" | "<=" ) term )* ;
term           → factor ( ( "-" | "+" ) factor )* ;
factor         → unary ( ( "/" | "*" ) unary )* ;

unary          → ( "!" | "-" ) unary | call ;

call           → primary ( "(" arguments? ")" | "." IDENTIFIER )* ;

primary        → "true" | "false" | "nil"
               | NUMBER | STRING | IDENTIFIER | "(" expression ")";

function       → IDENTIFIER "(" parameters? ")" block ;

parameters     → IDENTIFIER ":" IDENTIFIER 
					( "," IDENTIFIER ":" IDENTIFIER)* ;

arguments      → expression ( "," expression )* ;


NUMBER         → DIGIT+ ( "." DIGIT+ )? ;
STRING         → "\"" <any char except "\"">* "\"" ;
IDENTIFIER     → ALPHA ( ALPHA | DIGIT )* ;
ALPHA          → "a" ... "z" | "A" ... "Z" | "_" ;
DIGIT          → "0" ... "9" ;
```