---
layout: post
title: "The root of Lisp's Note"
description: ""
keywords: ""
category: 
tags: [lisp]
---
###1. Definitions:

 1. **Atom**: A sequence of letters(eg foo)-------------原子。也可以称之为符号（symbol）？
 2. **Expression**: an Atom, or a list of zero or more expressions, seperated by whitespace and enclosed by parentheses.
 3. If an Expression is a list. we call the first element the **Operator** and the remaining elements the **Arguments**.
 
###2. 7 Rules:
 1. `(quota x)` 也可以写成 `'x` return `x`
 2. `(atom x)` return the atom t if the value of `x` is an atom or the empty list, otherwise, it return ().
   * Now that we have an operator **atom** whose argument is evaluated. this operator **atom** can show what quote is for.So by quoting a list we protect it from evaluation; An unquoted list ( given as an argument to an operator atom) is treated as code`(atom (atom 'a))`code and data are made out of the same data structures, and the operator quota is the way we distinguish between them.
 3. `(equal a b)` return T if the a and b the same atom or both empty list. otherwise return empty list. Notes:
   * `(eq '(2 3) '(2 3))`      ====== {}
   * `(eq 'a 'a)`              =======T
 4. `(car x)` return first element (the operator) of list x.
 5. `(cdr x)` return remaining elements (the arguments) of list x.
 6. `(cons x y)` y is a list, return a list: x the first element, y the remains.
 7. `(cond (p1,e1)...(pn,en))` the p expressions are evaluated in order until one return T. when one is found, the value of the corresponding e expresssion is return as the value of whole cond expression.

####Important:

In five of our seven primitive operators, the arguments are always evaluated when an expression beginning with that operator is evaluated. we call an operator of that type a funtion.

这句话的意思是：如果一个表达式以其中的五个操作符开始，当它被求值的时候，它的参数总会被求值的。我们也称这种类型的操作符为函数。 这5个操作符不包括`(quota x)` 以及 `(cond (p1 e1)...(pn en))`
###3. Function:
相当与引进了一个新的控制符－－－lambda.

####Definition:

	A function is expressed as (lambda (p1...pn) e), where p1...pn are atoms and e is an expression.
	
	A function call is expressed as ((lambda (p1...pn) e) a1...an). each ai is evaluated and the e evaluated.

If an expression has its first element an atom f that is not one of the primitive operators like `(f a1...an)` and the value of f is a function `(lambda (p1...pn) e)` then the value of the expression is the value of `((lambda (p1...pn) e) a1...an)`
Exp:

`((lambda (f) (f '(b c))) '(lambda (x) (cons 'a x)))`
这个表达式的值是(a b c)

####关于递归

为了方便写递归。我们给`(lambda (p1...pn) e)`做一个简便的标记。（其实我们可以通过Y combinator来定义递归） `(defun f (p1...pn) e)` 等价于`f = (lambda (p1...pn) e)` 所有碰到f的地方，都可以用`(lambda (p1...pn) e)`来替换。
一个我实验的例子

(defun f (x y) (cons x y))

然后

 1. `(f 'a 'b)`的值为`(a . b)`
 2. `(f f 'b)` 的结果是f unbound-variable
 3. `(t 'f 'b)` 的结果是t undefined function call
 4. `(t f 'b)`的结果是f unbound-variable
 5. `(f 'f 'b)` 的值为 `(f . b)`
 6. `(let ((f 2)) (f f 'b))` 的值为`(2 . b)`

注意，这里面f的求值情况. 从上面的几个例子我们可以看出来。

 1. defun更像一个赋值操作，**只不过，它只在第一个元素未知的时候，采取替换操作**. 第二个例子中，显示f没有值就是最好的说明。
 2. 表达式的求值，**先对参数进行求值，然后对操作符进行查找替换**（如果不是原始的那几个操作符的话）。3和4的例子，也很清楚说明了，参数的求值先于操作符的求值.

####结论：
这样一来，对于任何型如`(f x y)`的求值规则就很明显了。先对参数求值，然后对操作符求值。
###4. 一些函数
有了函数的定义，并且知道了求助规则。我们就可以根据前面的七个操作符，定义出我们自己的函数。like `(null. x)` `(and. x y)` `(not. x)`
###5.The suprise:
我们可以写一个函数作为我们语言的解释器:此函数取任意Lisp表达式作自变量并返回它的值. 如下所示:
{% highlight lisp %}
(defun eval. (e a)
  (cond 
    ((atom e) (assoc. e a))
    ((atom (car e))
     (cond 
       ((eq (car e) 'quote) (cadr e))
       ((eq (car e) 'atom)  (atom   (eval. (cadr e) a)))
       ((eq (car e) 'eq)    (eq     (eval. (cadr e) a)
                                    (eval. (caddr e) a)))
       ((eq (car e) 'car)   (car    (eval. (cadr e) a)))
       ((eq (car e) 'cdr)   (cdr    (eval. (cadr e) a)))
       ((eq (car e) 'cons)  (cons   (eval. (cadr e) a)
                                    (eval. (caddr e) a)))
       ((eq (car e) 'cond)  (evcon. (cdr e) a))
       ('t (eval. (cons (assoc. (car e) a)
                        (cdr e))
                  a))))
    ((eq (caar e) 'label)
     (eval. (cons (caddar e) (cdr e))
            (cons (list (cadar e) (car e)) a)))
    ((eq (caar e) 'lambda)
     (eval. (caddar e)
            (append. (pair. (cadar e) (evlis. (cdr  e) a))
                     a)))))
(defun evcon. (c a)
  (cond ((eval. (caar c) a)
         (eval. (cadar c) a))
        ('t (evcon. (cdr c) a))))
(defun evlis. (m a)
  (cond ((null. m) '())
        ('t (cons (eval.  (car m) a)
                  (evlis. (cdr m) a)))))
{% endhighlight %}

具体的函数的解析可以参照比较好的中文翻译，主要逻辑在那中间的四个子句，我在这里就不具体详细谈论了。

Lisp的强大之处，就表现出来了。可以用自己的语言写出自己的解释器。

Lisp可以定义任何我们想要的额外的函数甚至语法。我们想想，为什么Java语言要加入一些新的特性和功能，需要升级Java1.1到现在的java1.8？ 是的，需要Java语言的开发者，去改写自己的java解释器，去支持一些新的语法和特性。而Lisp完全这种困扰，可以完全按照自己的思维方式去code，而不需要考虑语言支不支持的问题。