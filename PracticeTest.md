# CS352 Practice Questions
## Week 1 - Parsing + Basic Compiler
### Question 1: In Scala, implement def getNum for a single char parser with 32-bit integers assumed. `in.next` pops the next character. To check if the next char is a digit call `in.hasNext(isDigit)`.

### Question 2: Given the following ast, and given that one is using a stack based compiler that emits Scala code, lease fill in what code is emitted by the compiler.

a) 
```
AST:
eval(
  Let("x", Lit(5),
    Prim("+",
       Ref("x"), Prim("*",
          Lit(2), Ref("x")
       )
    )
  )
) 

Emitted:
val memory = new Array[Int](MEM_SIZE)

???

memory(0)
```

b)
```
AST:
eval(
  Let("x", Prim("*",Lit(2), Lit(3)),
    Let("y", Prim("-", Ref("x"), Lit(1)),
      Prim("*",
        Ref("y"), Prim("+",
           Lit(3), Ref("x")
        )
      )
    )
  ) 
)

Emitted:
val memory = new Array[Int](MEM_SIZE)

???

memory(0)
```

## Week 2 - Syntax/Semantic Errors, Syntax Sugar, Stack Compiler, etc.

todo: Syntax Error Handling + Semantic Error Handling

### Question 3: Given the AST Similar to question 2, evaluate the emitted code for a stack compiler to Scala.

```
AST:
eval(Let("x", 1, 
  If(Cond(">", Ref("x"), 0),
     Prim("+", 2, Ref("x")),
     0)
  ), 0
)(Map())

Emitted:
val memory = new Array[Int](1000)
var flag = true

???

memory(0)
```
### Question 4: Given the following grammar, please explain which each is syntactically valid or not. If valid state the return value.
<img width="459" alt="Screenshot 2025-03-12 at 6 08 37 PM" src="https://github.com/user-attachments/assets/588c5baf-eb29-4dae-a3cb-221e34aa2704" />

a)
```
if (3 == 5) {
  2
} * 4 else 8
```
b)
```
if (3 == 2)
  val x = 3; x
else
  5
```
### Question 5: Given the syntax sugar expressions, return the desugared ASTs:
a)
```
x = x + 2;
y = y + 2 * x
```
b)
```
if (x > 0)
  x = x - 1;
val y = x * 5;
x
```
c)
```
if (y > x)
  y = x - 1
  x = y + x;
x
```
## Week 3 - Type Checking, Functions, and Arrays
todo: Study call and ret for functions and how they manipulate the stack to ensure no data leaks out of functions. Also study CST.

<img width="395" alt="Screenshot 2025-03-12 at 6 14 45 PM" src="https://github.com/user-attachments/assets/cf0e60c8-054e-436e-8854-d2d8b4cca504" />
<img width="229" alt="Screenshot 2025-03-12 at 6 14 56 PM" src="https://github.com/user-attachments/assets/19fcfa91-322f-44c3-a8cb-650bb4864361" />
<img width="480" alt="Screenshot 2025-03-12 at 6 15 04 PM" src="https://github.com/user-attachments/assets/6f2e7ed6-ef65-4437-8428-faf42271bdc6" />
<img width="416" alt="Screenshot 2025-03-12 at 6 15 15 PM" src="https://github.com/user-attachments/assets/8b40962c-c691-4b34-9da4-85fc20547255" />
<img width="500" alt="Screenshot 2025-03-12 at 6 18 47 PM" src="https://github.com/user-attachments/assets/34e09d34-c6fe-414a-93b9-7eb1f6617bd8" />




### Question 6: Prove that each expression/AST is its given type using the Axioms/Type Rules.
a)
```
Type: Boolean

val x: Int = 1 + 5; x > 2
```
b)
```
Type: Boolean

val x: UnknownType = -4; x < 2
```
c)
```
Type: Int

val x: Int = 10; If(x > 5, x + 1, x - 1)
```
d)
```
Type: Int

AST:

VarDec(
  "c",
  BooleanType,
  Lit(BoolLit(true)),
  While(
    Ref("c"),
    Let(
      "dummy",
      UnitType,
      VarAssign("c", Lit(BoolLit(false))),
      Lit(UnitLit())
    ),
    Lit(IntLit(42))
  )
)
```
e)
```
Type: Int

AST:

VarDec(
  "arr",
  ArrayType(IntType),
  PrimAlloc(ArrayType(IntType), List(Lit(IntLit(5)))),
  Let(
    "dummy",
    UnitType,
    Prim("block-set", List(Ref("arr"), Lit(IntLit(3)), Lit(IntLit(99)))),
    Prim("block-get", List(Ref("arr"), Lit(IntLit(3))))
  )
)
```
### Questions 7: Please write the AST of:
```
def f(x: Int) = { g(x) + 1 };
def g(x: Int): Int = 2 * x;
f(1 + 1)
```
## Week 4-5
## Week 6
## Week 7
