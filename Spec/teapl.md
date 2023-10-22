## Grammar of TeaPL

Each program is composed of variable declarations, function declarations, function definitions, and comments.

```
program := (varDeclStmt | structDef | fnDeclStmt | fnDef | comment | < ; >)*
```

### 基本标识符、值、表达式和赋值

Each identifier begins with an alphabat and contains only alphabats and digits, e.g., alice, a0.

```
id := [a-zA-Z][a-zA-Z0-9]*   
```

TeaPL allows integers, e.g., 123
```
num := [1-9][0-9]* | 0
```

**算术表达式**
An expression is a composd of identifiers, values,  and operators, e.g., 1+2, a*(b+c). For simplicity, we donot support unary operators, such as ++, +=.

```
arithExpr :=  arithExpr arithBiOp arithExpr | exprUnit
exprUnit :=  num | id | < ( > arithExpr < ) > | fnCall | id < [ > id | num < ] > | id < . > id | arithUOp exprUnit
arithBiOp := < + > | < - > | < * > | < / >
arithUOp := < - >
```

主要可能是优先级的问题

**条件表达式**

```
boolExpr := boolExpr boolBiOp boolUnit | boolUnit
boolUnit := < ( > exprUnit comOp exprUnit < ) > | < ( > boolExpr < ) > | boolUOp boolUnit // we restrict the operands of comparison operators to be exprUnit instead of rightVal to avoid confusing the precedence.
boolBiOp := < && > | < || >
boolUOp := < ! >
comOp := < > > | < < > | < >= > | < <= > | < == > | < != >
```

**赋值**
We restrict neither the left value nor right value can be assignments.

```
assignStmt := leftVal < = > rightVal < ; >  
leftVal := id | id < [ > id | num < ] > | id < . > id
rightVal := arithExpr | boolExpr
```

**函数调用**

```
fnCall := id < ( > rightVal (< , > rightVal)* | ϵ < ) >
```

### 变量声明

TeaPL allows declaring one variable each time, which can be either a primitive or array type. Developers can initializate the variable during declaration. For example, it supports the following variable declaration samples.

**基本数据类型**

```
let a:int; // declare a variable of type int; the type field can be ignored.
let b:int = 0; // declare a variable of int and init it with value 0.
```
**一维数组**

```
let c[10]:int; // declear a variable of integer array.
let d[10]:int = {0}; // declear a variable of integer array and initialize it with zero.
```

The grammar is defined as follows.
 ```
varDeclStmt := < let > (varDecl | varDef) < ; >   
varDecl := id < : > type |  id < [ > num < ] >< : > type
varDef :=  id < : > type < = > rightVal  //primitive type
         | id < [ > num < ] >< : > type < = > < { > rightVal (< , > rightVal)* | ϵ < } > //array
type := nativeType | structType | ϵ
nativeType := < int >
structType := id
 ```

### 定义新的结构

Developers can define new customized types with the preserved keyword struct, e.g., 
```
struct MyStruct { 
    node:int, 
    len:int  
}
```

The grammar is defined as follows.
 ```
structDef := < struct > id < { > (varDecl) (< , > varDecl)* < } >
 ```

### 函数声明和定义

Each function declaration starts with the keyword fn.
```
fn foo(a:int, b:int)->int;
fn foo();
```

The grammar is defined as follows.
```
fnDeclStmt := fnDecl < ; >
fnDecl := < fn > id < ( > paramDecl < ) > //without return value
        | < fn > id < ( > paramDecl < ) > < -> > type //with return value
paramDecl := varDecl (< , > varDecl)* | ϵ
```

**函数定义**
We can also define a function while declaring it.
```
fn foo(a:int, b:int)->int {
    return a + b;
} 
```

The grammar is specified as follows.
```
fnDef := fnDecl codeBlock  
codeBlock :=  < { > (varDeclStmt | assignStmt | callStmt | ifStmt | whileStmt | returnStmt | continueStmt | breakStmt | < ; > )* < } > 
returnStmt ：= < ret > rightVal < ; >
continueStmt := < continue > < ; >
breakStmt := < break > < ; >
```

We have already defined the grammar of varDeclStmt and assignStmt. The callStmt is simply a function call terminated with an colon.

```
callStmt := fnCall < ; >
```
Next, we define the grammar of each rest statement type.

### 控制流

**If-Else 语句**
The condition should be surrounded with a paired parenthesis, and we further restrict the  body should be within a paired bracket. The following shows an example.

```
if (x >0) {
    if (y >0) {
        x++;
    }
    else {
        x--;
    }
} else {

}

```

Besides, we restrict the condition expression to be explicit logical operations, e.g., x >0; we donot allow implicit expressions like x, which means.  We define the grammar as follows.
```
ifStmt := < if > < ( > boolExpr < ) > codeBlock ( < else > codeBlock | ϵ )
```

**While 语句**

Used for the representability of complicated loops.

Example:
```
while (x  > 0) {
    x--;
}
```

Definition:

```
whileStmt := < while > < ( > boolExpr < ) > codeBlock
```

### 代码注释

Similar to most programming languages, TeaPL allows line comments with "//" and scope comments with "/* ... */".
```
int a = 0; // this is a line comment.

/*
    Feature: this is a scope comment
*/  
fn foo(){
    ...
}
```

```
comment :=  < // > _* | < /* > _* < */ >  
```