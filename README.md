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
