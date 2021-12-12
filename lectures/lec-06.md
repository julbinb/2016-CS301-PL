
## Язык IMP^e^ — простой императивный (imperative) язык ##

Язык _IMP^e^_ содержит выражения с целыми числами и логические выражения,
переменные, условный оператор и циклы.

Программа на IMP^e^ состоит из последовательности операторов (statement), которая 
завершается арифметическим выражением. Результатом программы является это
последнее арифметическое выражение, оно вычисляется в состоянии, в которое
приходит программа после выполнения операторов.

### Синтаксис ###

<pre>	
Var 	::=		<обычные идентификаторы>

AExp	::=		Var 
			  | <константы целых чисел>
			  | AExp + AExp		| AExp - AExp
			  | AExp * AExp		| AExp / AExp
			  
BExp	::= 	true | false
			  | AExp <= AExp 	| AExp >= AExp
			  | AExp = AExp
			  | BExp and BExp	| BExp or BExp
			  | not BExp
			  
Stmt 	::= 	skip 
			  | Var := AExp 
			  | Stmt ; Stmt 
			  | if BExp then Stmt else Stmt 
			  | while BExp Stmt

Prgm	::= 	Stmt ; AExp
</pre>

Вся программа на языке IMP^e^ представляется нетерминалом `Prgm`.
_Термом_ будем называть программу или часть программы.

*	Метапеременными $x$, $y$, ... будем обозначать переменные.
*	Метапеременными $a$, $a_k$ — арифметические выражения.
*	Метапеременными $i_k$ — целые числа.
*	Метапеременными $l_k$ — константы `true`, `false` (логические значения).
*	Метапеременными $s$, $s'$, $s_k$ — операторы.


### Семантика ###

Вычисление выражений не имеет побочных эффектов.
Опраторы имеют побочные эффекты, то есть изменяют _состояние_, в котором
выполняется программа.

**Состоянием** будем называть отображение из переменных в целые числа:
$\mathrm{Var} \rightarrow \mathrm{Int}$.

Состояния будем обоназначать $\sigma, \sigma', \ldots$.
$\sigma(x)$ — это значение переменной $x$ в состоянии $\sigma$.  
Через $\sigma_0$ будем обозначать начальное состояние.
В начальном состоянии значения переменных не определены, то есть
$\sigma$ — частичная функция.

Операционная семантика, которую мы будем определять, задаёт абстрактную
машину, _состояниями_ которой являются конфигурации.  
**Конфигурацией** будем называть пару, содержащую терм и состояние.
Конфигурации будем записывать в угловых скобках, например: $\langle a, \sigma \rangle$.

Теперь необходимо определить функцию перехода, или **отношение вычисления**.
используем наше любимое индуктивное определение.

#### Вычисление арифметических выражений ####

<!-- 
\langle x, \sigma \rangle

\langle x, \sigma \rangle  \longrightarrow \langle y, \sigma \rangle
 -->

$$\cfrac{\sigma(x) \text{ is defined}}
	{\langle x, \sigma \rangle  \longrightarrow \langle \sigma(x), \sigma \rangle}\, (\text{E-Var})$$

Как процессор умеет выполнять арифметические выражения, так и наша абстрактная
машина скрывает в себе математические вычисления. 
Именно этим мы пользуемся в предпосылке E-Sum.

$$\cfrac{i_1 + i_2 = i_3 \text{ в } \mathbb{Z}}
	{\langle i_1 + i_2, \sigma \rangle  \longrightarrow \langle i_3, \sigma \rangle}\, (\text{E-Sum})$$
	
Так как мы уже люди опытные в смысле детерминированных вычислений,
нужно аккуратно написать правила для сложения.	
	
$$\cfrac{\langle a_1, \sigma \rangle  \longrightarrow \langle a_1', \sigma \rangle}
	{\langle a_1 + a_2, \sigma \rangle  \longrightarrow \langle a_1' + a_2, \sigma \rangle}\, (\text{E-SumLeft})
\qquad
\cfrac{\langle a_2, \sigma \rangle  \longrightarrow \langle a_2', \sigma \rangle}
	{\langle i_1 + a_2, \sigma \rangle  \longrightarrow \langle i_1 + a_2', \sigma \rangle}\, (\text{E-SumRight})$$

Аналогично вычитание и умножение.  
Теперь целочисленное деление:

$$\cfrac{i_2 \neq 0 \quad i_1 / i_2 = i_3 \text{ в } \mathbb{Z}}
	{\langle i_1 / i_2, \sigma \rangle  \longrightarrow \langle i_3, \sigma \rangle}\, (\text{E-Div})$$
	
$$\cfrac{\langle a_1, \sigma \rangle  \longrightarrow \langle a_1', \sigma \rangle}
	{\langle a_1 / a_2, \sigma \rangle  \longrightarrow \langle a_1' / a_2, \sigma \rangle}\, (\text{E-SumLeft})
\qquad
\cfrac{\langle a_2, \sigma \rangle  \longrightarrow \langle a_2', \sigma \rangle}
	{\langle i_1 / a_2, \sigma \rangle  \longrightarrow \langle i_1 / a_2', \sigma \rangle}\, (\text{E-SumRight})$$

#### Вычисление логических выражений ####

$$\cfrac{i_1 \leq i_2 \text{ в } \mathbb{Z}}
	{\langle i_1 \leq i_2, \sigma \rangle  \longrightarrow \langle \mathtt{true}, \sigma \rangle}\, (\text{E-LeqTrue})
\qquad
\cfrac{i_1 > i_2 \text{ в } \mathbb{Z}}
	{\langle i_1 \leq i_2, \sigma \rangle  \longrightarrow \langle \mathtt{false}, \sigma \rangle}\, (\text{E-LeqFalse})$$

$$\cfrac{\langle a_1, \sigma \rangle  \longrightarrow \langle a_1', \sigma \rangle}
	{\langle a_1 \leq a_2, \sigma \rangle  \longrightarrow \langle a_1' \leq a_2, \sigma \rangle}\, (\text{E-LeqLeft})
\qquad
\cfrac{\langle a_2, \sigma \rangle  \longrightarrow \langle a_2', \sigma \rangle}
	{\langle i_1 \leq a_2, \sigma \rangle  \longrightarrow \langle i_1 \leq a_2', \sigma \rangle}\, (\text{E-LeqRight})$$

Аналогично операторы $\geq$ и $=$.

$$\cfrac{l_1 \wedge l_2 = true \text{ в } \mathbb{B}}
	{\langle l_1\ \mathtt{and}\ l_2, \sigma \rangle  \longrightarrow \langle \mathtt{true}, \sigma \rangle}\, (\text{E-AndTrue})
\qquad
\cfrac{l_1 \wedge l_2 = false \text{ в } \mathbb{B}}
	{\langle l_1\ \mathtt{and}\ l_2, \sigma \rangle  \longrightarrow \langle \mathtt{false}, \sigma \rangle}\, (\text{E-AndFalse})$$
	
$$\cfrac{\langle b_1, \sigma \rangle  \longrightarrow \langle b_1', \sigma \rangle}
	{\langle b_1\ \mathtt{and}\ b_2, \sigma \rangle  \longrightarrow \langle b_1'\ \mathtt{and}\ b_2, \sigma \rangle}\, (\text{E-AndLeft})
\qquad
\cfrac{\langle b_2, \sigma \rangle  \longrightarrow \langle b_2', \sigma \rangle}
	{\langle l_1\ \mathtt{and}\ b_2, \sigma \rangle  \longrightarrow \langle l_1\ \mathtt{and}\ b_2', \sigma \rangle}\, (\text{E-AndRight})$$

#### Выполнение операторов ####

**Пустой оператор** выполнять не нужно.

**Оператор присваивания** предполагает изменение состояния.  
Состояние, полученное из $\sigma$ присваиванием переменной $x$ значения $i$,
будем записывать $\sigma[i/x]$. Оно определяется так:

$$\sigma[i/x](y) = \begin{cases}
	i, & \text{если } y = x\\
	\sigma(y), & \text{если } y \neq x\\
\end{cases}$$

Теперь правила вычисления:

$$\cfrac{\langle a, \sigma \rangle  \longrightarrow \langle a', \sigma \rangle}
	{\langle x := a, \sigma \rangle  \longrightarrow \langle x := a', \sigma \rangle}\, (\text{E-Assign})$$
	
А если значение уже вычислено:

$$\cfrac{}{\langle x := i, \sigma \rangle  \longrightarrow \langle \mathtt{skip}, \sigma[i/x] \rangle}\, (\text{E-Var})$$

Выполнили оператор присваивания и в новой конфигурации заменили его на пустой оператор.

**Последовательность операторов.**

$$\cfrac{\langle s_1, \sigma \rangle  \longrightarrow \langle s_1', \sigma' \rangle}
	{\langle s_1\ ; s_2, \sigma \rangle  \longrightarrow \langle s_1'\ ; s_2, \sigma' \rangle}\, (\text{E-SeqFirst})$$

$$\cfrac{}{\langle \mathtt{skip}\ ; s_2, \sigma \rangle  \longrightarrow \langle s_2, \sigma \rangle}\, (\text{E-Seq})$$

**Условный оператор.** 

$$\cfrac{}{\langle \mathtt{if}\ \mathtt{true}\ \mathtt{then}\ s_1\ \mathtt{else}\ s_2, \sigma \rangle  
	\longrightarrow \langle s_1, \sigma \rangle}\, (\text{E-IfTrue})
\qquad
\cfrac{}{\langle \mathtt{if}\ \mathtt{false}\ \mathtt{then}\ s_1\ \mathtt{else}\ s_2, \sigma \rangle  
	\longrightarrow \langle s_2, \sigma \rangle}\, (\text{E-IfFalse})$$

$$\cfrac{\langle b, \sigma \rangle  \longrightarrow \langle b', \sigma \rangle}
	{\langle \mathtt{if}\ b\ \mathtt{then}\ s_1\ \mathtt{else}\ s_2, \sigma \rangle  \longrightarrow \langle \mathtt{if}\ b'\ \mathtt{then}\ s_1\ \mathtt{else}\ s_2, \sigma \rangle}\, 
(\text{E-If})$$
	
**Оператор цикла.**

Значение условия нужно перевычислять на каждой итерации цикла.

$$\cfrac{}{\langle \mathtt{while}\ b\ s, \sigma \rangle  
	\longrightarrow \langle \mathtt{if}\ b\ \mathtt{then}\ (s\ ; \mathtt{while}\ b\ s)\ \mathtt{else}\ \mathtt{skip}, \sigma \rangle}\, 
(\text{E-While})$$
	
#### Выполнение программы ####	
	
Начальной конфигурацией машины для программы $s\, ; a$ является
$\langle s\, ; a, \sigma_0 \rangle$	
	
$$\cfrac{\langle s, \sigma \rangle  \longrightarrow \langle s', \sigma' \rangle}
	{\langle s\, ; a, \sigma \rangle  \longrightarrow \langle s'\, ; a, \sigma' \rangle}\, 
(\text{E-PrgmStmt})$$

Любой оператор в конце концов вычисляется в пустой оператор.
Далее переходим к вычислению выражения.

$$\cfrac{}{\langle \mathtt{skip}\, ; a, \sigma \rangle  \longrightarrow \langle a, \sigma \rangle}\,
(\text{E-Prgm})$$

#### Завершимость ####

Программа завершается корректно, если существует _финальная_ конфигурация
$\langle i, \sigma \rangle$, в которую приходит машина.

Программа завершается с ошибкой, если машина останавливается,
но не приходит в финальную конфигурацию.

Язык Тьюринг-полный, поэтому машина может никогда не остановиться.

## Задания ##

1.	Привести пример незавершающейся программы.
1.	Программы с делением на ноль.
1.	Программы с использованием плохой переменной.
1.	Хорошей программы.

1.	Привести вывод некоторого отношения вычисления.


## Литература ##

* 	Бенджамин Пирс. Типы в языках программирования [ref](http://www.ozon.ru/context/detail/id/7410082/).
	3.5.
	
*	Glynn Winskelю The Formal Semantics of Programming Languages: An Introduction.
	Глава 2. Разделы 2.1, 2.6.
	
*	Конспект лекций «CS522. Programming Language Design. Operational Semantics», Grigore Rosu.
	Department of Computer Science University of Illinois at Urbana-Champaign.
	
