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
### Question 7: Please write the AST of:
```
def f(x: Int) = { g(x) + 1 };
def g(x: Int): Int = 2 * x;
f(1 + 1)
```
## Week 4 - CMScala to CPS
todo: Tail and Cond Opt + SSA, phi functions and the 2 alternative IRs

### Question 8: Please rewrite the following in CPS Scala Code:
a)
```
def gcd(x: Int, y: Int): Unit =
  if (y == 0)
    printInt(x)
  else
    gcd(y, x % y);
gcd(2016, 714)
```
b)
```
def isEven(n: Int): Boolean =
  if (n == 0) true else isOdd(n - 1)
def isOdd(n: Int): Boolean =
  if (n == 0) false else isEven(n - 1)
isEven(5)
```

<img width="456" alt="Screenshot 2025-03-12 at 6 37 57 PM" src="https://github.com/user-attachments/assets/3bc9099c-ba4c-4df4-863e-d92c9857cf25" />
<img width="459" alt="Screenshot 2025-03-12 at 6 38 12 PM" src="https://github.com/user-attachments/assets/85fb566f-3651-4318-bed2-c8f74dbe2656" />

### Question 9: Given the following CMS to CPS translation rules, translate the CMS ASTs to CPS ASTs

a)
```
AST (Simplified):

LetRec(
  List(
    FunDef(
      "max",
      List(
        Arg("x"),
        Arg("y")
      ),
      If(
        Prim(">", List(Ref("x"), Ref("y"))),
        Ref("x"),
        Ref("y")
      )
    )
  ),
  App(
    Ref("max"),
    List(
      Lit(IntLit(2016)),
      Lit(IntLit(714))
    )
  )
)
```
b)
```
AST:

VarDec(
  "c",
  BooleanType,
  Lit(BoolLit(true)),
  While(
    Ref("c"),
    VarAssign("c", Lit(BoolLit(false))),
    Lit(IntLit(42))
  )
)

```
c)
```
AST:

Let(
  "arr",
  ArrayType(IntType),
  Prim("block-alloc-242", List(Lit(IntLit(5)))),
  Let(
    "dummy",
    UnitType,
    Prim("block-set", List(Ref("arr"), Lit(IntLit(3)), Lit(IntLit(99)))),
    Prim("block-get", List(Ref("arr"), Lit(IntLit(3))))
  )
)

```

## Week 5 - Value Representation
todo: tagged integer arithmetic (AST or just for value), tagged block + I/O ops, type check functions (via tag)

## Week 6 - Closure Conversion
todo: study free variables as shown in image
<img width="398" alt="Screenshot 2025-03-12 at 6 49 23 PM" src="https://github.com/user-attachments/assets/2dfa6523-ef25-48a3-92e8-f8832fdd9a22" />

### Non-Optimized
<img width="460" alt="Screenshot 2025-03-12 at 6 49 51 PM" src="https://github.com/user-attachments/assets/c34a92af-dbd1-4658-9ddb-0736022d1b79" />
<img width="228" alt="Screenshot 2025-03-12 at 6 50 20 PM" src="https://github.com/user-attachments/assets/760876f1-8e18-4288-a0b2-d7dc00a107c7" />

### Optimized
<img width="486" alt="Screenshot 2025-03-12 at 6 50 56 PM" src="https://github.com/user-attachments/assets/bc175efc-01bc-4678-b824-ea22166e25de" />

### Question 10: As detailed as possible, write Pseudo/Scala code for the bodies and locations of the worker and wrapper functions.
a)

```
def processList(lst: List[Int]): List[Int] = {
  val offset = 5
  def processEven(l: List[Int]): List[Int] = l match {
    case Nil => Nil
    case h :: t =>
      if (h % 2 == 0) (h + offset) :: processOdd(t)
      else processOdd(t)
  }
  def processOdd(l: List[Int]): List[Int] = l match {
    case Nil => Nil
    case h :: t =>
      if (h % 2 != 0) (h - offset) :: processEven(t)
      else processEven(t)
  }
  processEven(lst)
}
```

b)

```
def makeAccumulator(initial: Int): Int => Int = {
  val base = initial + 10
  def accumulator(x: Int): Int = {
    def loop(n: Int, acc: Int): Int =
      if (n <= 0) acc else loop(n - 1, acc * x + base)
    loop(x, 1)
  }
  accumulator
}
makeAccumulator(20)
```
## Week 7 - Optimization

todo: Make some optimization examples for the techniques discussed in class and when we should or shouldn't optimize, what to check for, and other special cases.
