# Лабораторная работа 7. Преобразование и анализ кода с использованием Clang и LLVM
Цель работы: Познакомиться с инструментами Clang и LLVM, научиться собирать AST и IR-промежуточное представление кода на C/C++, а также извлекать базовую информацию о программе (например, список функций).
Задачи:
1. Установить Clang и LLVM;
2. Скомпилировать простой C-файл с использованием clang и получить его: абстрактное синтаксическое дерево (AST), промежуточное представление LLVM IR;
3. Использовать opt для применения базовой комплексной оптимизации (например, О2);
4. Построить граф потока управления (CFG) для оптимизированной программы;
5. Проанализировать результат, сделать выводы и ответить на контрольные вопросы

## 1. Установка и подготовка среды
Работа выполнялась в среде Ubuntu 22.04. Установлены следующие инструменты:
● clang — компилятор языка C/C++;
● llvm — инструменты анализа и оптимизации кода;
● opt — инструмент для работы с LLVM IR и применения
оптимизаций;
● Graphviz — инструмент для визуализации кода.
Команда установки: sudo apt install clang llvm

![image](https://github.com/user-attachments/assets/580fddd1-ccbd-483f-add7-e09d6c6eab06)

## 2. Исходный код программы для демонстрации работы инструментов (сохранена в test.c):
```C++
#include <stdio.h>
int square(int x) {
 return x * x;
}

int main() {
 int a = 5;
 int b = square(a);
 printf("%d\n", b);
 return 0;
}
```

## 3. Получение AST

Команда: clang -Xclang -ast-dump -fsyntax-only test.c

```C++
|-FunctionDecl 0x1873fca0 <test.c:2:1, line:4:1> line:2:5 used square 'int (int)'
| |-ParmVarDecl 0x1873fc08 <col:12, col:16> col:16 used x 'int'
| `-CompoundStmt 0x1873fde8 <col:19, line:4:1>
|   `-ReturnStmt 0x1873fdd8 <line:3:2, col:13>
|     `-BinaryOperator 0x1873fdb8 <col:9, col:13> 'int' '*'
|       |-ImplicitCastExpr 0x1873fd88 <col:9> 'int' <LValueToRValue>
|       | `-DeclRefExpr 0x1873fd48 <col:9> 'int' lvalue ParmVar 0x1873fc08 'x' 'int'
|       `-ImplicitCastExpr 0x1873fda0 <col:13> 'int' <LValueToRValue>
|         `-DeclRefExpr 0x1873fd68 <col:13> 'int' lvalue ParmVar 0x1873fc08 'x' 'int'
`-FunctionDecl 0x1873fe50 <line:5:1, line:10:1> line:5:5 main 'int ()'
  `-CompoundStmt 0x187407b8 <col:12, line:10:1>
    |-DeclStmt 0x1873ff90 <line:6:2, col:11>
    | `-VarDecl 0x1873ff08 <col:2, col:10> col:6 used a 'int' cinit
    |   `-IntegerLiteral 0x1873ff70 <col:10> 'int' 5
    |-DeclStmt 0x187400f0 <line:7:2, col:19>
    | `-VarDecl 0x1873ffc0 <col:2, col:18> col:6 used b 'int' cinit
    |   `-CallExpr 0x187400b0 <col:10, col:18> 'int'
    |     |-ImplicitCastExpr 0x18740098 <col:10> 'int (*)(int)' <FunctionToPointerDecay>
    |     | `-DeclRefExpr 0x18740028 <col:10> 'int (int)' Function 0x1873fca0 'square' 'int (int)'
    |     `-ImplicitCastExpr 0x187400d8 <col:17> 'int' <LValueToRValue>
    |       `-DeclRefExpr 0x18740048 <col:17> 'int' lvalue Var 0x1873ff08 'a' 'int'
    |-CallExpr 0x18740710 <line:8:2, col:18> 'int'
    | |-ImplicitCastExpr 0x187406f8 <col:2> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
    | | `-DeclRefExpr 0x18740108 <col:2> 'int (const char *, ...)' Function 0x18724c78 'printf' 'int (const char *, ...)'
    | |-ImplicitCastExpr 0x18740758 <col:9> 'const char *' <NoOp>
    | | `-ImplicitCastExpr 0x18740740 <col:9> 'char *' <ArrayToPointerDecay>
    | |   `-StringLiteral 0x18740128 <col:9> 'char[4]' lvalue "%d\n"
    | `-ImplicitCastExpr 0x18740770 <col:17> 'int' <LValueToRValue>
    |   `-DeclRefExpr 0x18740148 <col:17> 'int' lvalue Var 0x1873ffc0 'b' 'int'
    `-ReturnStmt 0x187407a8 <line:9:2, col:9>
      `-IntegerLiteral 0x18740788 <col:9> 'int' 0
```
Видно, что функция square принята, содержит параметр x и возвращает x * x.

## 4. Генерация LLVM IR

Команда: clang -S -emit-llvm test.c -o test.ll

```C++
; ModuleID = 'test.c'
source_filename = "test.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"


@.str = private unnamed_addr constant [4 x i8] c"%d\0A\00", align 1


; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @square(i32 noundef %0) #0 {
  %2 = alloca i32, align 4
  store i32 %0, i32* %2, align 4
  %3 = load i32, i32* %2, align 4
  %4 = load i32, i32* %2, align 4
  %5 = mul nsw i32 %3, %4
  ret i32 %5
}


; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  %3 = alloca i32, align 4
  store i32 0, i32* %1, align 4
  store i32 5, i32* %2, align 4
  %4 = load i32, i32* %2, align 4
  %5 = call i32 @square(i32 noundef %4)
  store i32 %5, i32* %3, align 4
  %6 = load i32, i32* %3, align 4
  %7 = call i32 (i8*, ...) @printf(i8* noundef getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i64 0, i64 0), i32 noundef %6)
  ret i32 0
}


declare i32 @printf(i8* noundef, ...) #1


attributes #0 = { noinline nounwind optnone uwtable "frame-pointer"="all" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { "frame-pointer"="all" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }


!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{i32 7, !"frame-pointer", i32 2}
!5 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
```

## 5. Оптимизация IR

Команда: clang -O0 -S -emit-llvm test.c -o test_O0.ll<br>
Стоит отметить, что в файле с IR до оптимизации:<br>
Все переменные (a, b, x.addr) размещены в памяти через alloca;<br>
Множество операций load и store;<br>
square вызывается как отдельная функция<br>

```C++
; ModuleID = 'test.c'
source_filename = "test.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"


@.str = private unnamed_addr constant [4 x i8] c"%d\0A\00", align 1


; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @square(i32 noundef %0) #0 {
  %2 = alloca i32, align 4
  store i32 %0, i32* %2, align 4
  %3 = load i32, i32* %2, align 4
  %4 = load i32, i32* %2, align 4
  %5 = mul nsw i32 %3, %4
  ret i32 %5
}


; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  %3 = alloca i32, align 4
  store i32 0, i32* %1, align 4
  store i32 5, i32* %2, align 4
  %4 = load i32, i32* %2, align 4
  %5 = call i32 @square(i32 noundef %4)
  store i32 %5, i32* %3, align 4
  %6 = load i32, i32* %3, align 4
  %7 = call i32 (i8*, ...) @printf(i8* noundef getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i64 0, i64 0), i32 noundef %6)
  ret i32 0
}


declare i32 @printf(i8* noundef, ...) #1


attributes #0 = { noinline nounwind optnone uwtable "frame-pointer"="all" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { "frame-pointer"="all" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }


!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{i32 7, !"frame-pointer", i32 2}
!5 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
```

Команда: clang -O2 -S -emit-llvm test.c -o test_O2.ll<br>

Команда -O2 – комплексная оптимизация среднего уровня. Она применяет более 30 различных оптимизаций:<br>
● -inline – встраивание небольших функций (встраивает square в main, если она вызывается один раз);<br>
● -constprop – подставит значение square(5) → 25, если функция встроена и всё известно на этапе компиляции;<br>
● -mem2reg – перевод переменных из памяти в регистры (SSA);<br>
● -instcombine – объединение и упрощение инструкций (упростит арифметику, например x * x может быть преобразовано в shl при x = 2^n);<br>
● -simplifycfg – оптимизирует структуру блоков (Упростит граф управления, если после inlining останутся лишние блоки);<br>
● -reassociate, -gvn, -sroa, -dce и другие.<br>
В файле с IR после оптимизации:<br>
Вся функция square исчезла – она была встроена (-inline) и затем вычислена (оптимизация -constprop);<br>
Никаких переменных, alloca, store, load – всё удалено (оптимизации -mem2reg, -dce);<br>
Остался только вызов printf(25).<br>

```C++
; ModuleID = 'test.c'
source_filename = "test.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"


@.str = private unnamed_addr constant [4 x i8] c"%d\0A\00", align 1


; Function Attrs: mustprogress nofree norecurse nosync nounwind readnone uwtable willreturn
define dso_local i32 @square(i32 noundef %0) local_unnamed_addr #0 {
  %2 = mul nsw i32 %0, %0
  ret i32 %2
}


; Function Attrs: nofree nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #1 {
  %1 = tail call i32 (i8*, ...) @printf(i8* noundef nonnull dereferenceable(1) getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i64 0, i64 0), i32 noundef 25)
  ret i32 0
}



; Function Attrs: nofree nounwind
declare noundef i32 @printf(i8* nocapture noundef readonly, ...) local_unnamed_addr #2


attributes #0 = { mustprogress nofree norecurse nosync nounwind readnone uwtable willreturn "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { nofree nounwind uwtable "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #2 = { nofree nounwind "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }


!llvm.module.flags = !{!0, !1, !2, !3}
!llvm.ident = !{!4}


!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
```

Команда: diff test_O0.ll test_O2.ll
Сравнение двух файлов:

```
8,15c8,11
< ; Function Attrs: noinline nounwind optnone uwtable
< define dso_local i32 @square(i32 noundef %0) #0 {
<   %2 = alloca i32, align 4
<   store i32 %0, i32* %2, align 4
<   %3 = load i32, i32* %2, align 4
<   %4 = load i32, i32* %2, align 4
<   %5 = mul nsw i32 %3, %4
<   ret i32 %5
---
> ; Function Attrs: mustprogress nofree norecurse nosync nounwind readnone uwtable willreturn
> define dso_local i32 @square(i32 noundef %0) local_unnamed_addr #0 {
>   %2 = mul nsw i32 %0, %0
>   ret i32 %2
18,29c14,16
< ; Function Attrs: noinline nounwind optnone uwtable
< define dso_local i32 @main() #0 {
<   %1 = alloca i32, align 4
<   %2 = alloca i32, align 4
<   %3 = alloca i32, align 4
<   store i32 0, i32* %1, align 4
<   store i32 5, i32* %2, align 4
<   %4 = load i32, i32* %2, align 4
<   %5 = call i32 @square(i32 noundef %4)
<   store i32 %5, i32* %3, align 4
<   %6 = load i32, i32* %3, align 4
<   %7 = call i32 (i8*, ...) @printf(i8* noundef getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i64 0, i64 0), i32 noundef %6)
---
> ; Function Attrs: nofree nounwind uwtable
> define dso_local i32 @main() local_unnamed_addr #1 {
>   %1 = tail call i32 (i8*, ...) @printf(i8* noundef nonnull dereferenceable(1) getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i64 0, i64 0), i32 noundef 25)
33c20,21
< declare i32 @printf(i8* noundef, ...) #1
---
> ; Function Attrs: nofree nounwind
> declare noundef i32 @printf(i8* nocapture noundef readonly, ...) local_unnamed_addr #2
35,36c23,25
< attributes #0 = { noinline nounwind optnone uwtable "frame-pointer"="all" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
< attributes #1 = { "frame-pointer"="all" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
---
> attributes #0 = { mustprogress nofree norecurse nosync nounwind readnone uwtable willreturn "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
> attributes #1 = { nofree nounwind uwtable "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
> attributes #2 = { nofree nounwind "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
38,39c27,28
< !llvm.module.flags = !{!0, !1, !2, !3, !4}
< !llvm.ident = !{!5}
---
> !llvm.module.flags = !{!0, !1, !2, !3}
> !llvm.ident = !{!4}
45,46c34
< !4 = !{i32 7, !"frame-pointer", i32 2}
< !5 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
---
> !4 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
```

Стоит отметить, что после оптимизации произошли следующие изменения:<br>
● Переменные типа alloca были удалены;<br>
● Код переведён в SSA-форму;<br>
● Оптимизация улучшила читаемость и упростила поток управления<br>

## 6. Граф потока управления программы
Команда для генерации оптимизированного LLVM IR: clang -O2 -S -emit-llvm main.c -o test.ll<br>
Команда для генерации .dot-файлов CFG для функций: opt -dot-cfg -disable-output test.ll<br>

![image](https://github.com/user-attachments/assets/7356698c-7a15-43ad-84b7-0de4bfb289ae)

Эта команда создаст DOT-файлы: .main.dot – для функции main; .square.dot – для square, если она не была удалена оптимизацией. <br>

Команда для установки библиотеки Graphviz: sudo apt install graphviz<br>
Команды для преобразования файлов с расширением .dot в .png с помощью Graphviz:<br>
1. dot -Tpng .main.dot -o cfg_main.png<br>
2. dot -Tpng .square.dot -o cfg_square.png<br>
Команды для просмотра файлов с CGF:<br>
xdg-open cfg_main.png<br>
![image](https://github.com/user-attachments/assets/cf8d3194-ca3e-4f36-b782-00fc69625e17)<br>
xdg-open cfg_square.png<br>
![image](https://github.com/user-attachments/assets/84f057f9-bab3-401b-b90d-9accb7c8a4b9)<br>

Стоит отметить, что в LLVM каждый граф потока управления (CFG) строится на уровне функции, поскольку структура управления всегда локальна для тела функции. Для получения полного представления о программе, нужно построить CFG для всех функций и анализировать их совокупность. Автоматическое объединение всех CFG в один граф не предусмотрено в LLVM по умолчанию.

## Выводы:
● С помощью Clang можно получить полную структуру AST и IR, а также CGF;<br>
● LLVM предоставляет гибкие инструменты анализа и оптимизации;<br>
● Промежуточное представление кода удобно для написания компиляторных трансформаций.<br>

## Ответы на контрольные вопросы:
### 1. Что такое Clang, и какова его роль в процессе компиляции программ?
Clang — это компилятор для языков C, C++, Objective-C и др., основанный на инфраструктуре LLVM. Он выполняет разбор исходного кода, строит абстрактное синтаксическое дерево (AST), проверяет семантику и генерирует промежуточное представление (LLVM IR), передавая его дальше на этапы оптимизации и генерации машинного кода.

### 2. Что представляет собой LLVM и как он используется в современных компиляторах?
LLVM — это модульная компиляторная инфраструктура с набором библиотек для анализа, оптимизации и генерации кода. Она используется в качестве backend’а в компиляторах, таких как Clang, Rust и Swift, обеспечивая переносимость, мощные оптимизации и поддержку множества целевых платформ.

### 3. Чем отличается абстрактное синтаксическое дерево (AST) от промежуточного представления LLVM IR?
AST — это высокоуровневое представление структуры исходного кода, близкое к синтаксису языка. LLVM IR — это низкоуровневое, платформонезависимое представление, приближенное к машинному коду. AST используется на ранних этапах компиляции, а IR — на этапах анализа, оптимизации и генерации кода.

### 4. Для чего необходимо промежуточное представление (IR) в процессе компиляции?
IR упрощает переносимость, многоуровневую оптимизацию и упрощает поддержку новых архитектур. Оно служит единым форматом между frontend (анализ кода) и backend (генерация кода), позволяя многократно переиспользовать компоненты компилятора.

### 5. Что делает инструкция alloca в LLVM IR, и зачем она используется в функциях?
Инструкция alloca выделяет память на стеке внутри функции. Она используется для хранения временных переменных или массивов, аналогично локальным переменным в C. Выделенная память автоматически освобождается при выходе из функции.

### 6. Зачем нужна оптимизация кода в компиляторе, и какие основные цели она преследует?
Оптимизация улучшает производительность, уменьшает размер кода, снижает потребление памяти и увеличивает энергоэффективность. Цель — преобразовать код так, чтобы он выполнялся быстрее или занимал меньше ресурсов, не изменяя поведение программы.

### 7. Что такое SSA-форма и почему она важна при оптимизации программ?
SSA (Static Single Assignment) — это форма представления кода, где каждая переменная присваивается только один раз. Это упрощает анализ зависимостей, позволяет эффективнее проводить оптимизации, такие как устранение мёртвого кода и константная свёртка.

### 8. Что такое граф потока управления (CFG) и как он помогает анализировать поведение программы?
CFG (Control Flow Graph) — это граф, где вершины представляют базовые блоки (последовательности инструкций без ветвлений), а рёбра — возможные переходы управления. Он используется для анализа возможных путей выполнения, упрощения анализа зависимостей и построения оптимизаций.

### 9. Как устроено представление арифметических операций в LLVM IR (например, умножение, сложение)?
Арифметические операции в LLVM IR представлены инструкциями вроде add, mul, sub, fadd (для вещественных чисел). Например:
```C++
%result = add i32 %a, %b
```
Каждая операция явно указывает тип операндов и всегда возвращает результат, хранящийся в новой переменной.

### 10. Почему функции в LLVM IR обычно представляют собой отдельные единицы анализа и оптимизации?
Функции являются изолированными блоками кода с чёткими границами, что упрощает их анализ и оптимизацию. Это позволяет применять инлайнинг, удаление неиспользуемых функций и локальные оптимизации без влияния на остальной код.

### 11. Что происходит с функцией в LLVM IR, если она вызывается один раз и очень короткая?
Такая функция обычно подлежит инлайн-развёртке — её тело вставляется непосредственно в место вызова, чтобы устранить накладные расходы вызова и улучшить возможности для последующих оптимизаций.

### 12. Какие преимущества даёт использование IR и CFG для автоматических оптимизаций по сравнению с анализом исходного текста на C?
IR и CFG дают чёткую, формальную структуру программы, избавленную от синтаксических сложностей языка. Это облегчает анализ зависимостей, управление потоком, позволяет применять универсальные оптимизации и переносить их между языками, в отличие от анализа исходного кода, где такие операции были бы сложны и ненадёжны.

## Выполнение доп. задания
Вариант №1. Задание: Напишите программу, в которой создается объект типа std::complex<double> z(3.0, 4.0);, и выведите его модуль. Постройте AST и LLVM IR, проанализируйте, как компилятор обрабатывает вызов конструктора и вычисление модуля.

Реализация на C++:
```C++
#include <iostream>
#include <complex>
#include <cmath>

int main() {
    std::complex<double> z(3.0, 4.0);
    std::cout << std::abs(z) << std::endl;
    return 0;
}
```

### Абстрактное синтаксическое дерево (AST)
Команда: clang++ -Xclang -ast-dump -fsyntax-only dop.cpp<br>
Ключевой фрагмент AST:<br>
```C++
`-FunctionDecl 0x18460bd0 <dop.cpp:5:1, line:9:1> line:5:5 main 'int ()'
  `-CompoundStmt 0x1846f248 <col:12, line:9:1>
    |-DeclStmt 0x18460ea0 <line:6:5, col:37>
    | `-VarDecl 0x18460d68 <col:5, col:36> col:26 used z 'std::complex<double>':'std::complex<double>' callinit
    |   `-CXXConstructExpr 0x18460e68 <col:26, col:36> 'std::complex<double>':'std::complex<double>' 'void (double, double)'
    |     |-FloatingLiteral 0x18460dd0 <col:28> 'double' 3.000000e+00
    |     `-FloatingLiteral 0x18460df0 <col:33> 'double' 4.000000e+00
    |-CXXOperatorCallExpr 0x1846f1b0 <line:7:5, col:38> 'std::basic_ostream<char>::__ostream_type':'std::basic_ostream<char>' lvalue '<<'
    | |-ImplicitCastExpr 0x1846f198 <col:30> 'std::basic_ostream<char>::__ostream_type &(*)(std::basic_ostream<char>::__ostream_type &(*)(std::basic_ostream<char>::__ostream_type &))' <FunctionToPointerDecay>
    | | `-DeclRefExpr 0x1846f120 <col:30> 'std::basic_ostream<char>::__ostream_type &(std::basic_ostream<char>::__ostream_type &(*)(std::basic_ostream<char>::__ostream_type &))' lvalue CXXMethod 0x18055788 'operator<<' 'std::basic_ostream<char>::__ostream_type &(std::basic_ostream<char>::__ostream_type &(*)(std::basic_ostream<char>::__ostream_type &))'
    | |-CXXOperatorCallExpr 0x1846e550 <col:5, col:28> 'std::basic_ostream<char>::__ostream_type':'std::basic_ostream<char>' lvalue '<<'
    | | |-ImplicitCastExpr 0x1846e538 <col:15> 'std::basic_ostream<char>::__ostream_type &(*)(double)' <FunctionToPointerDecay>
    | | | `-DeclRefExpr 0x1846e4c0 <col:15> 'std::basic_ostream<char>::__ostream_type &(double)' lvalue CXXMethod 0x18056e58 'operator<<' 'std::basic_ostream<char>::__ostream_type &(double)'
    | | |-DeclRefExpr 0x18460f08 <col:5, col:10> 'std::ostream':'std::basic_ostream<char>' lvalue Var 0x180d1ba8 'cout' 'std::ostream':'std::basic_ostream<char>'
    | | `-CallExpr 0x1846a470 <col:18, col:28> 'double':'double'
    | |   |-ImplicitCastExpr 0x1846a458 <col:18, col:23> 'double (*)(const complex<double> &)' <FunctionToPointerDecay>
    | |   | `-DeclRefExpr 0x1846a3c0 <col:18, col:23> 'double (const complex<double> &)' lvalue Function 0x1846a2c0 'abs' 'double (const complex<double> &)' (FunctionTemplate 0x184099d8 'abs')
    | |   `-ImplicitCastExpr 0x1846a498 <col:27> 'const complex<double>':'const std::complex<double>' lvalue <NoOp>
    | |     `-DeclRefExpr 0x18460fe0 <col:27> 'std::complex<double>':'std::complex<double>' lvalue Var 0x18460d68 'z' 'std::complex<double>':'std::complex<double>'
    | `-ImplicitCastExpr 0x1846f108 <col:33, col:38> 'basic_ostream<char, std::char_traits<char>> &(*)(basic_ostream<char, std::char_traits<char>> &)' <FunctionToPointerDecay>
    |   `-DeclRefExpr 0x1846f0d0 <col:33, col:38> 'basic_ostream<char, std::char_traits<char>> &(basic_ostream<char, std::char_traits<char>> &)' lvalue Function 0x18059f08 'endl' 'basic_ostream<char, std::char_traits<char>> &(basic_ostream<char, std::char_traits<char>> &)' (FunctionTemplate 0x1803b488 'endl')
    `-ReturnStmt 0x1846f238 <line:8:5, col:12>
      `-IntegerLiteral 0x1846f218 <col:12> 'int' 0
```
Видим, что:
CXXConstructExpr — вызов конструктора std::complex<double>(3.0, 4.0).
CallExpr — вызов abs(z).
CXXOperatorCallExpr — перегруженный <<.

### LLVM IR (O1):
Команда: clang++ -O1 -S -emit-llvm dop.cpp -o dop.ll
Получаем следующее:
```C++
; ModuleID = 'dop.cpp'
source_filename = "dop.cpp"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

%"class.std::ios_base::Init" = type { i8 }
%"class.std::basic_ostream" = type { i32 (...)**, %"class.std::basic_ios" }
%"class.std::basic_ios" = type { %"class.std::ios_base", %"class.std::basic_ostream"*, i8, i8, %"class.std::basic_streambuf"*, %"class.std::ctype"*, %"class.std::num_put"*, %"class.std::num_get"* }
%"class.std::ios_base" = type { i32 (...)**, i64, i64, i32, i32, i32, %"struct.std::ios_base::_Callback_list"*, %"struct.std::ios_base::_Words", [8 x %"struct.std::ios_base::_Words"], i32, %"struct.std::ios_base::_Words"*, %"class.std::locale" }
%"struct.std::ios_base::_Callback_list" = type { %"struct.std::ios_base::_Callback_list"*, void (i32, %"class.std::ios_base"*, i32)*, i32, i32 }
%"struct.std::ios_base::_Words" = type { i8*, i64 }
%"class.std::locale" = type { %"class.std::locale::_Impl"* }
%"class.std::locale::_Impl" = type { i32, %"class.std::locale::facet"**, i64, %"class.std::locale::facet"**, i8** }
%"class.std::locale::facet" = type <{ i32 (...)**, i32, [4 x i8] }>
%"class.std::basic_streambuf" = type { i32 (...)**, i8*, i8*, i8*, i8*, i8*, i8*, %"class.std::locale" }
%"class.std::ctype" = type <{ %"class.std::locale::facet.base", [4 x i8], %struct.__locale_struct*, i8, [7 x i8], i32*, i32*, i16*, i8, [256 x i8], [256 x i8], i8, [6 x i8] }>
%"class.std::locale::facet.base" = type <{ i32 (...)**, i32 }>
%struct.__locale_struct = type { [13 x %struct.__locale_data*], i16*, i32*, i32*, [13 x i8*] }
%struct.__locale_data = type opaque
%"class.std::num_put" = type { %"class.std::locale::facet.base", [4 x i8] }
%"class.std::num_get" = type { %"class.std::locale::facet.base", [4 x i8] }

@_ZStL8__ioinit = internal global %"class.std::ios_base::Init" zeroinitializer, align 1
@__dso_handle = external hidden global i8
@_ZSt4cout = external global %"class.std::basic_ostream", align 8
@llvm.global_ctors = appending global [1 x { i32, void ()*, i8* }] [{ i32, void ()*, i8* } { i32 65535, void ()* @_GLOBAL__sub_I_dop.cpp, i8* null }]

declare void @_ZNSt8ios_base4InitC1Ev(%"class.std::ios_base::Init"* noundef nonnull align 1 dereferenceable(1)) unnamed_addr #0

; Function Attrs: nounwind
declare void @_ZNSt8ios_base4InitD1Ev(%"class.std::ios_base::Init"* noundef nonnull align 1 dereferenceable(1)) unnamed_addr #1

; Function Attrs: nofree nounwind
declare i32 @__cxa_atexit(void (i8*)*, i8*, i8*) local_unnamed_addr #2

; Function Attrs: norecurse uwtable
define dso_local noundef i32 @main() local_unnamed_addr #3 {
  %1 = call double @cabs(double noundef 3.000000e+00, double noundef 4.000000e+00) #7
  %2 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo9_M_insertIdEERSoT_(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) @_ZSt4cout, double noundef %1)
  %3 = bitcast %"class.std::basic_ostream"* %2 to i8**
  %4 = load i8*, i8** %3, align 8, !tbaa !5
  %5 = getelementptr i8, i8* %4, i64 -24
  %6 = bitcast i8* %5 to i64*
  %7 = load i64, i64* %6, align 8
  %8 = bitcast %"class.std::basic_ostream"* %2 to i8*
  %9 = getelementptr inbounds i8, i8* %8, i64 %7
  %10 = getelementptr inbounds i8, i8* %9, i64 240
  %11 = bitcast i8* %10 to %"class.std::ctype"**
  %12 = load %"class.std::ctype"*, %"class.std::ctype"** %11, align 8, !tbaa !8
  %13 = icmp eq %"class.std::ctype"* %12, null
  br i1 %13, label %14, label %15

14:                                               ; preds = %0
  call void @_ZSt16__throw_bad_castv() #8
  unreachable

15:                                               ; preds = %0
  %16 = getelementptr inbounds %"class.std::ctype", %"class.std::ctype"* %12, i64 0, i32 8
  %17 = load i8, i8* %16, align 8, !tbaa !13
  %18 = icmp eq i8 %17, 0
  br i1 %18, label %22, label %19

19:                                               ; preds = %15
  %20 = getelementptr inbounds %"class.std::ctype", %"class.std::ctype"* %12, i64 0, i32 9, i64 10
  %21 = load i8, i8* %20, align 1, !tbaa !15
  br label %28

22:                                               ; preds = %15
  call void @_ZNKSt5ctypeIcE13_M_widen_initEv(%"class.std::ctype"* noundef nonnull align 8 dereferenceable(570) %12)
  %23 = bitcast %"class.std::ctype"* %12 to i8 (%"class.std::ctype"*, i8)***
  %24 = load i8 (%"class.std::ctype"*, i8)**, i8 (%"class.std::ctype"*, i8)*** %23, align 8, !tbaa !5
  %25 = getelementptr inbounds i8 (%"class.std::ctype"*, i8)*, i8 (%"class.std::ctype"*, i8)** %24, i64 6
  %26 = load i8 (%"class.std::ctype"*, i8)*, i8 (%"class.std::ctype"*, i8)** %25, align 8
  %27 = call noundef signext i8 %26(%"class.std::ctype"* noundef nonnull align 8 dereferenceable(570) %12, i8 noundef signext 10)
  br label %28

28:                                               ; preds = %19, %22
  %29 = phi i8 [ %21, %19 ], [ %27, %22 ]
  %30 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo3putEc(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) %2, i8 noundef signext %29)
  %31 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo5flushEv(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) %30)
  ret i32 0
}

; Function Attrs: nofree nounwind
declare double @cabs(double noundef, double noundef) local_unnamed_addr #4

declare noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo9_M_insertIdEERSoT_(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8), double noundef) local_unnamed_addr #0

declare noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo3putEc(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8), i8 noundef signext) local_unnamed_addr #0

declare noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo5flushEv(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8)) local_unnamed_addr #0

; Function Attrs: noreturn
declare void @_ZSt16__throw_bad_castv() local_unnamed_addr #5
declare void @_ZNKSt5ctypeIcE13_M_widen_initEv(%"class.std::ctype"* noundef nonnull align 8 dereferenceable(570)) local_unnamed_addr #0

; Function Attrs: uwtable
define internal void @_GLOBAL__sub_I_dop.cpp() #6 section ".text.startup" {
  call void @_ZNSt8ios_base4InitC1Ev(%"class.std::ios_base::Init"* noundef nonnull align 1 dereferenceable(1) @_ZStL8__ioinit)
  %1 = call i32 @__cxa_atexit(void (i8*)* bitcast (void (%"class.std::ios_base::Init"*)* @_ZNSt8ios_base4InitD1Ev to void (i8*)*), i8* getelementptr inbounds (%"class.std::ios_base::Init", %"class.std::ios_base::Init"* @_ZStL8__ioinit, i64 0, i32 0), i8* nonnull @__dso_handle) #7
  ret void
}



attributes #0 = { "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { nounwind "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #2 = { nofree nounwind }
attributes #3 = { norecurse uwtable "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #4 = { nofree nounwind "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #5 = { noreturn "frame-pointer"="none" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #6 = { uwtable "frame-pointer"="none" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #7 = { nounwind }
attributes #8 = { noreturn }

!llvm.module.flags = !{!0, !1, !2, !3}
!llvm.ident = !{!4}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
!5 = !{!6, !6, i64 0}
!6 = !{!"vtable pointer", !7, i64 0}
!7 = !{!"Simple C++ TBAA"}
!8 = !{!9, !10, i64 240}
!9 = !{!"_ZTSSt9basic_iosIcSt11char_traitsIcEE", !10, i64 216, !11, i64 224, !12, i64 225, !10, i64 232, !10, i64 240, !10, i64 248, !10, i64 256}
!10 = !{!"any pointer", !11, i64 0}
!11 = !{!"omnipotent char", !7, i64 0}
!12 = !{!"bool", !11, i64 0}
!13 = !{!14, !11, i64 56}
!14 = !{!"_ZTSSt5ctypeIcE", !10, i64 16, !12, i64 24, !10, i64 32, !10, i64 40, !10, i64 48, !11, i64 56, !11, i64 57, !11, i64 313, !11, i64 569}
!15 = !{!11, !11, i64 0}
```

Видно, что по сути все действия по инициализации std::complex<double> и вычислению модуля компилятор свёл к двум вызовам:
В исходном коде я использовал 
```C++
std::complex<double> z(3.0, 4.0);
```
Но этот конструктор у std::complex тривиален (просто сохраняет два double в поля). Clang/LLVM при оптимизациях видит, что аргументы константны и структура простая, и не генерирует для этого никакой отдельной функции-конструктора. Вместо этого он просто записывает литералы 3.0 и 4.0 прямо в код, а поля z.real() и z.imag() остаются в регистрах или на стеке. Таким образом, в IR нет никакого call к функции-конструктору std::complex<double>::complex(double,double) — он сводится к «нулевому» по стоимости действию.
Сама же функция std::abs(complex<double>) реализована в стандартной библиотеке как обёртка вокруг функции cabs(double, double) из libm. В IR это отражено строкой:
```C++
%1 = call double @cabs(double noundef 3.000000e+00, double noundef 4.000000e+00)
```
Видно, что компилятор генерирует прямой вызов к cabs(3.0, 4.0).
Получив double—результат от cabs, LLVM продолжает генерировать код оператора вывода:
```C++
%2 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo9_M_insertIdEERSoT_(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) @_ZSt4cout, double noundef %1)
```
Это низкоуровневый вызов метода _M_insert<double>(cout, value), который превращает число в строку с учётом локали, выводит его символ за символом (put), а затем вызывает flush:
```C++
%30 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo3putEc(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) %2, i8 noundef signext %29)
%31 = call noundef nonnull align 8 dereferenceable(8) %"class.std::basic_ostream"* @_ZNSo5flushEv(%"class.std::basic_ostream"* noundef nonnull align 8 dereferenceable(8) %30)
```
