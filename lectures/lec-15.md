
<!-- Модуль 2
	 (2::9) Лекция 15. Типобезопасность SLTC: сохранение -->

# Реконструкция типов #

## Мотивация ##

Ранее мы явным образом писали аннотации типов. 
А что, если аннотации хочется опускать? Тогда необходимо, чтобы
компилятор сам вычислял тип терма.

Например, что можно сказать о следующем терме (можно записать на ML)  
$\lambda f.\lambda g.\lambda x.
\mathtt{if}\ (f\ x)\ \mathtt{then}\ x\ \mathtt{else}\ (g\ x)$

А как вы это поняли? <Рассуждение> (с помощью выписывания ограничений).

На этой лекции рассмотрим алгоритм **реконструкции типов**.

## Типовые переменные и подстановки ##

Будем считать, что у нас в языке есть множество _неинтерпретируемых 
(uninterpreted)_ или _неизвестных (unknown)_ базовых типов, для которых 
не определены никакие базовые операции, нет способа создать термы этих 
типов. Такие типы рассматривать как _типовые переменные (type variables)_,
заглушки для настоящих типов.

Рассмотрим терм $\lambda f\,{:}\,\mathtt{X} \rightarrow \mathtt{Bool}.
\lambda g\,{:}\,\mathtt{X} \rightarrow \mathtt{X}.\lambda x\,{:}\,\mathtt{X}.
\mathtt{if}\ (f\ x)\ \mathtt{then}\ x\ \mathtt{else}\ (g\ x)$.  
Что можно сказать об этом терме, о его типизируемости?  
Можно заменить `X` на любой тип и мы получим правильно типизированный терм.
Вот это свойство как раз лежит в основе параметрического полиморфизма,
к которому мы позже и перейдём.

Типовые переменные можно _конкретизировать/инстанцировать (instantiate)_
другими типами с помощью _подстановки (substitution)_.

_Подстановка типов (type substitution)_ — это отображение $\sigma$ 
типовых переменных в типы. Например, $[\mathtt{X} \mapsto \mathtt{Nat}]$,
$[\mathtt{X} \mapsto \mathtt{Nat}, \mathtt{Y} \mapsto \mathtt{X} \rightarrow \mathtt{X}]$.
$dom(\sigma)$ — это область определения подстановки, то есть множество
типовых переменных, встречающихся слева.
А $range(\sigma)$ — множество типов, встречающихся справа.
Типовая переменная может встречаться и слева, и справа.

_Инстанцирование_ типа — это _применение_ подстановки к типу, 
записывается как $\sigma\mathtt{T}$.
Все компоненты подстановки применяются одновременно.

*	$[\mathtt{X} \mapsto \mathtt{Nat}] (\mathtt{Bool} \rightarrow \mathtt{X}) =
	\mathtt{Bool} \rightarrow \mathtt{Nat}$
*	$[\mathtt{X} \mapsto \mathtt{Nat}, \mathtt{Y} \mapsto \mathtt{X} \rightarrow \mathtt{X}]
	(\mathtt{Y} \rightarrow \mathtt{X}) = (\mathtt{X} \rightarrow \mathtt{X}) \rightarrow \mathtt{Nat} $

Определим **применение подстановки**:

$\begin{array}{lcl}
\sigma(\mathtt{X}) & = & \begin{cases}
		\mathtt{T}, \text{ if } (\mathtt{X} \mapsto \mathtt{T}) \in \sigma\\
		\mathtt{X}, \text{ if } \mathtt{X} \notin dom(\sigma)\\
\end{cases} \\
\sigma(\mathtt{Nat}) & = & \mathtt{Nat} \\
\sigma(\mathtt{Bool}) & = & \mathtt{Bool} \\
\sigma(\mathtt{T}_1 \rightarrow \mathtt{T}_2) & =  &\sigma\mathtt{T}_1 \rightarrow \sigma\mathtt{T}_2
\end{array}$

Нужно определить ещё применение подстановки к контексту типизации и терму.

Ясно, что к типу может применяться несколько подстановок.
Определим _композицию подстановок_ 
$(\sigma \circ \gamma)\mathtt{T} = \sigma(\gamma(\mathtt{T}))$.

$\sigma \circ \gamma = \left[ \begin{array}{ll}
\mathtt{X} \mapsto \sigma\mathtt{T} & \text{forall } (\mathtt{X} \mapsto \mathtt{T}) \in \gamma \\
\mathtt{X} \mapsto \sigma\mathtt{T} & \text{forall } (\mathtt{X} \mapsto \mathtt{T}) \in \sigma, 
	\mathtt{X} \notin dom(\gamma)
\end{array} \right]$

#### Сохранение типизации при подстановке ####

**Теорема.** Если $\Gamma \vdash t : \mathtt{T}$, то 
$\sigma\Gamma \vdash t : \sigma\mathtt{T}$.

_Доказательство._ Индукция по отношению типизации.

*	Правило T-App:  
	$\cfrac{\Gamma \vdash t_1 : \mathtt{T}_1 \rightarrow \mathtt{T}_2 \quad \Gamma \vdash t_2 : \mathtt{T}_1}
	{\Gamma \vdash t_1\ t_2 : \mathtt{T}_2}$

	По индуктивной гипотезе: $\Gamma \vdash t_1 : \sigma\mathtt{T}_1 \rightarrow \sigma\mathtt{T}_2$,
	$\Gamma \vdash t_2 : \sigma\mathtt{T}_1$, значит применимо правило T-App.

Теперь посмотрим на терм $\lambda f\,{:}\,\mathtt{Z}.
\lambda g\,{:}\,\mathtt{Y}.\lambda x\,{:}\,\mathtt{X}.
\mathtt{if}\ (f\ x)\ \mathtt{then}\ x\ \mathtt{else}\ (g\ x)$.

Очевидно, не любая конкретизация даст правильный терм.
Такая даст, и такая даст, а такая нет.

Подстановка $[\mathtt{Z} \mapsto \sigma\mathtt{X} \rightarrow \mathtt{Bool},
\mathtt{Y} \mapsto \sigma\mathtt{X} \rightarrow \mathtt{X}$
даёт _наиболее общую (most general)_ конкретизацию терма. Это подстановка,
которая при подставновке конкретных типов даёт правильно типизированные термы,
и при этом предъявляет наименьшие требования.

Поиск корректных конкретизаций и лежит в основе алгоритмов **реконструкции
типов (type reconstruction)** или **вывода типов (type inference)**.  
Такие алгоритмы как раз используются в компиялторах SML и Haskell
(ещё немного, и сможете писать компилятор).

**Решением** для пары $(\Gamma, t)$, контекста и терма, называется такая 
пара $(\sigma, \mathtt{T})$, что $\sigma\Gamma \vdash t : \sigma\mathtt{T}$.

Один пример: $\Gamma = \{x : Nat, f : \mathtt{Y}, g : \mathtt{Z}\}$,
`t = if iszero(x) then f x else g (f x)`.
Примеры решений: `Nat -> Nat / Y, Nat -> Nat / Z`, `Nat -> S / Y, S -> S / Z`.

## Типизация на основе ограничений ##

## Алгоритм унификации ##



## Задания ##

1.	Записать применение подстановки к контексту типизации.
2.	Записать применение подстановки к терму.
3.	Доказать сохранение типов при подстановке для T-IsZero, T-Abs.


## Литература ##

*	Бенджамин Пирс, глава 22 (реконструкция типов).
