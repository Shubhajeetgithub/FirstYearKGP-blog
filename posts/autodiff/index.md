---
title: Autodiff
description: A gentle introduction to differentiation, automatic differentiation, and computational graphs, exploring how modern machine learning frameworks efficiently compute exact gradients through forward and reverse mode autodiff
pubDate: 2026-11-28
author: kingshuk-patra
tags:
  - pytorch
cover: cover.png
coverAlt: PyTorch
draft: false
---

So, here we are going to talk about a concept that is used most frequently in everything. So what's the problem? Suppose we have a function $f(x)$ and we want to compute $\mathbb{\delta} f(x)/\mathbb{\delta} x$.

But why? Well suppose we are in a continuous domain. Then for a function $f$, we want to find $\inf_{x \in S} f(x)$, well thanks to the geometrical significance of gradient over the continuous space, we can do this by using the fact that, $\nabla f(x)$ at point $x$, points at direction of increase of $f$, if $f(x)\in \mathbb{R}$.

So if we just take a small step opposite to the gradient, $f$ goes down. And if we keep doing it,

$$x_{t+1} = x_t - \eta \nabla f(x_t)$$

where $\eta$ is some small step size. This is gradient descent. Very simple, but this one line is basically how every neural network gets trained. There $x$ is all the parameters of the model, which can be millions or even billions, and $f$ is the loss, which says how wrong the model is. So training a model is nothing but finding a point where the loss is small, and for that we need the gradient again and again, in every single step.

And it's not only deep learning. Physics simulations, robotics, fitting curves to data, finance where people want to know how the price changes if some input changes, even Newton's method for finding roots, all of them need derivatives somewhere.

Now think of what it asks from us. The function is not some nice formula written on paper, it's a whole program, with loops, ifs and thousands of operations. And we need its gradient with respect to a lot of inputs, exactly, and fast, because we need it millions of times. Keep these three things in mind, as we will see that each method below either fails or passes on these.

So, overall in a non-rigorous sense we can understand that finding gradient is important.

## So how to compute gradient?

Given how important gradients are, let's see what we can do.

### By definition?

Well, if we remember the definition of differentiation by Newton, then

$$
\frac{\mathbb{\delta}f(x)}{\mathbb{\delta}x} = \lim_{h \to 0} \frac{f(x+h)-f(x)}{h} 
$$

So by the definition itself, instead of zero limiting $h$, if we take a small $h$, such as say $10^{-4}$ or such, then we can have a numerical approximation for a value for any black box function $f$.

So implementing it in python is fairly simple.

```python
def differentiate(func:Callable, x, h=1e-4):
    return (func(x+h)-func(x))/h
```

But is it all? Our problem solved? Perhaps no.

Why? What is even the issues with this? Well technically we are approximating the value, not calculating the exact value. It works and gives a near approximation in most cases, but not exactly right solution.

If we try to do some examples.

$f(x)=x^2$ and we want $f'(0)$. If we take then the error is $O(h)$. In fact, if we try,suppose the function $f$ is analytic, then

$\forall h \in \mathbb{R} \exists c \in (x, x+h)  f(x+h)=f(x)+hf'(x)+\frac{h^2}{2}f''(x)+\frac{h^3}{3!}f'''(c)$

So, by that we have a error in our evaluation in our computation for a analytic function. And non-rigorously speaking, it's more uncertain in non-analytic cases.

### Can we do better?

Well it seems as yes, we can do much better. In fact we can find exact values. How?

Let's do little computation. Let's see how we compute a differentiation.

Let's suppose a function $f(x)=x+sin(x^2+x)$

Now if we go for computing the value for $f'(x)$, let's look into our computation. I say you take the thing in your copy yourself.

So it goes as, $f'(x) = 1 + (2x + 1)cos(x^2+x)$. Did you see what and how you did?

Well if we formally say, in $\mathbb{R}$, the operations are additions and multiplication. And what happens under additions and multiplications? If you remember your calculus course,

$$\frac{\delta (f_1(x)+f_2(x))}{\delta x} = \frac{\delta f_1(x)}{\delta x}+\frac{\delta f_2(x)}{\delta x}$$
$$\frac{\delta f(x)g(x)}{\delta x} = f'(x)g(x)+f(x)g'(x)$$
$$
\frac{\delta f(g(x))}{\delta x}=f'(g(x))g'(x)
$$

Now if we look at these cases, if we already have the values of $f$,$g$,$f'$,$g'$ at $x$, then finding those values are trivial right? That is the only information we need.

Now think of what it lefts us at? Suppose a complicated function $f(x)=\mathbb{e}^{x^2+sin(x)}$, even for that we only need those composition and addition rules and some basic forms like what happens for exponential and what happens for trigonometric function. Right?

Now think about the situation it leaves us at. How much computation and how much logic do we need to find gradients? It seems as it's much small.

Now let's see computation as a graph. If you have explored compilers and AST, or say database query processing or anything such, you can understand this is a very trivial concept. Even it is a great way to see the diagrams in the Deep Learning papers' architectures.

#### Let's see Computation as a Graph.

Now consider the function given before. $f(x)=\mathbb{e}^{x^2+sin(x)}$.

If I ask how do you define this function in python?

```python
def f(x):
    sin_output = math.sin(x)
    squared = x**2
    exp_in = sin_output + squared
    exp_out = math.exp(exp_in)
    return exp_out
```

Well actually, in any case I would have written that function in a one liner or perhaps a lambda. But here writing that elaborately has specific reasons.
Now see, each variable like `exp_in` to `exp_out`, that's a function composition. Also the addition in calculating `exp_in`, and finding `sin_output` or `squared`.


Overall, this behaves as a nested elementary functions. So if we build a graph where by each node we signify a specific value, and those interact only by those functions that we define as elementary? Let's form the graph.
<figure class="compute-graph">
<svg viewBox="0 0 720 220" role="img" aria-label="Compute graph of f(x) = exp(x^2 + sin x)" xmlns="http://www.w3.org/2000/svg">
<defs>
<marker id="cg-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
<path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
</marker>
</defs>
<style>
.compute-graph svg { width: 100%; height: auto; color: #444; }
.compute-graph .node { fill: #f4f6fa; stroke: #8a94a6; stroke-width: 1.5; }
.compute-graph .out { fill: #e8f1ff; stroke: #3b6fd4; }
.compute-graph text { fill: #1a1d23; font: 15px ui-monospace, Menlo, Consolas, monospace; text-anchor: middle; dominant-baseline: middle; }
.compute-graph .edge { stroke: currentColor; stroke-width: 1.6; fill: none; marker-end: url(#cg-arrow); }
@media (prefers-color-scheme: dark) {
.compute-graph svg { color: #aab2c0; }
.compute-graph .node { fill: #232833; stroke: #5b6578; }
.compute-graph .out { fill: #1d2c4d; stroke: #6c9bff; }
.compute-graph text { fill: #e6e9ef; }
}
</style>
<path class="edge" d="M110,115 L200,52"/>
<path class="edge" d="M110,115 L200,178"/>
<path class="edge" d="M310,52 L390,105"/>
<path class="edge" d="M310,178 L390,125"/>
<path class="edge" d="M500,115 L560,115"/>
<rect class="node" x="20" y="95" width="90" height="40" rx="8"/>
<text x="65" y="115">v₀ = x</text>
<rect class="node" x="200" y="32" width="110" height="40" rx="8"/>
<text x="255" y="52">v₁ = v₀²</text>
<rect class="node" x="200" y="158" width="110" height="40" rx="8"/>
<text x="255" y="178">v₂ = sin v₀</text>
<rect class="node" x="390" y="95" width="110" height="40" rx="8"/>
<text x="445" y="115">v₃ = v₁ + v₂</text>
<rect class="node out" x="560" y="95" width="140" height="40" rx="8"/>
<text x="630" y="115">v₄ = exp v₃ = f</text>
</svg>
<figcaption>Compute graph for f(x) = e<sup>x² + sin x</sup>. Note that x fans out to two branches that rejoin at v₃.</figcaption>
</figure>


So now, I hope you understand the concept. Try to define some more cases of yourself.

Now look back at the $3$ rules for differentiation. I am not going to rewrite that. So if we see those and also the graph then we can observe a phenomenon. While being in a node, the differentiator operator only interacts with the adjacent nodes, and not any other nodes in the graph. This is kind of greedy property that is beneficial to our computation as we only need to write specific code.


Well, so now we understand a behaviour of derivatives. And also this properties follow through the multivariate calculus for partial derivatives.

Now I think we shall see what we gain from all here.

### Symbolic differentiation

Well with libraries like `sympy` you can compute exact symbolic expression for a function. Also actually it's much simple and not specific to sympy. Though it's relied already on what I told earlier so far, I am not going any more into that.
If you want you can also see perhaps that it can be implemented simply, though `sympy`'s one is more robust. [link](https://github.com/kingshuk7995/differentiator/blob/main/src/main.cpp). Also there are other options other than `sympy`, like `symengine`.

### Autodiff

Now I think we shall move forward to what happens when we actually need to compute gradient for a complicated expression. If we first think of the earlier `pytorch 1.x`'s autodiff, that was a eager mode tape based. Now you may ask what is eager mode.

Eager mode and lazy mode simply differs by when do you do computation. In eager mode, when you write `c = a + b` the addition is done right then and there, and you have the value of `c`. In lazy mode, it doesn't compute anything at that line, it just keeps a note that `c` is `a+b` and builds the graph, and only when you ask for the value it computes. If you have used `tensorflow 1.x` with the sessions, you perhaps understand. Also `jax.jit` or `torch.compile` in `pytorch 2.x` is somewhat like that side.

So why we care about it? Well, in eager mode we don't have the whole graph beforehand. We just see operations one after another as they happen. So what pytorch did is, while computing it also writes down each operation in a list, like what operation it was, what were the inputs and what was the output. That list is what we call tape. So when the forward computation is over, the tape is basically the compute graph we drew before, just written in the order we computed it. And think about that order, you can't use a value before you computed it, right? So the order is already a topological order of the graph. It's a nice thing as we will see later.

Now we have the graph. Then which way do we go?

#### Forward mode

The first thing that comes to mind is, go along with the computation itself. For each node $v_i$, along with the value we also keep $\dot{v_i} = \frac{\delta v_i}{\delta x}$. And because of that greedy property we talked about, $\dot{v_i}$ only needs the values from its parent nodes.

So for our $f(x)=\mathbb{e}^{x^2+sin(x)}$, let's take the graph and go one by one.

$\dot v_0 = 1$ as $v_0 = x$

$\dot v_1 = 2v_0 \dot v_0$

$\dot v_2 = cos(v_0)\dot v_0$

$\dot v_3 = \dot v_1 + \dot v_2$

$\dot v_4 = \mathbb{e}^{v_3}\dot v_3$

and $\dot v_4$ is our $f'(x)$. Now see, there is no $h$ here. So no approximation, it's the exact value.

Now how to implement it? One nice way is by dual numbers. Suppose a number of form $a + b\epsilon$ where $\epsilon \neq 0$ but $\epsilon^2 = 0$. Seems weird, but just think of it like $i$ where $i^2=-1$. Now go back to that Taylor expansion we did earlier, and put $h = b\epsilon$. All the terms with $h^2$ or more just becomes zero. So we get,

$$f(a+b\epsilon) = f(a) + f'(a)b\epsilon$$

So if we keep the $\epsilon$ part along with the value, we get the derivative also with the value. In python we just need to overload the operators.

```python
import math

class Dual:
    def __init__(self, val, der=0.0):
        self.val = val
        self.der = der

    def __add__(self, other):
        other = other if isinstance(other, Dual) else Dual(other)
        return Dual(self.val + other.val, self.der + other.der)
    __radd__ = __add__

    def __mul__(self, other):
        other = other if isinstance(other, Dual) else Dual(other)
        return Dual(self.val * other.val, self.der * other.val + self.val * other.der)
    __rmul__ = __mul__

def sin(x):
    return Dual(math.sin(x.val), math.cos(x.val) * x.der)

def exp(x):
    return Dual(math.exp(x.val), math.exp(x.val) * x.der)

def f(x):
    return exp(x * x + sin(x))

out = f(Dual(1.0, 1.0))
print(out.val, out.der)
```

If you look at `__add__` and `__mul__`, it's nothing but the addition rule and the product rule we saw earlier.

So are we done? Well, not really. Here in one pass we are getting derivative with respect to one input only. So if $f:\mathbb{R}^n \to \mathbb{R}$, then to get the full gradient we need to do $n$ passes. Now think of neural networks. There $n$ is the number of parameters which is in millions or even billions, and the output is a single scalar which is the loss. So for one step of gradient descent we need to do billions of passes? That's not practical at all.

#### Reverse mode

So let's think the other way round. Instead of finding how each node changes with $x$, let's find how $f$ changes with each node. So we define $\bar{v_i} = \frac{\delta f}{\delta v_i}$, people call it adjoint. Now $\bar v_4 = 1$ obviously, as $v_4$ is $f$ itself. Then we go backward in the graph.

$\bar v_4 = 1$

$\bar v_3 = \bar v_4 \mathbb{e}^{v_3}$

$\bar v_1 = \bar v_3$ and $\bar v_2 = \bar v_3$

$\bar v_0 = \bar v_1 2v_0 + \bar v_2 cos(v_0)$

Now look at $\bar v_0$ carefully. $x$ went to two branches, so contribution from both the branches gets added up. That's the multivariate chain rule, and that is why I said about the branches in the graph earlier.

So what we gain? In one forward pass and one backward pass we get derivative with respect to all the inputs. Whether it's one input or billion, doesn't matter. And yes, if you heard about backpropagation, this is that only. Backprop is nothing but reverse mode autodiff on a neural network.

But there is a cost. To go backward, we need the values of the intermediate nodes, like $v_3$ or $v_0$, so we need to store them while doing the forward pass. And where do we store them? In the tape itself. That is also why training needs much more memory than inference.

Now let's implement it. It's not much code actually.

```python
import math

tape = []

class Var:
    def __init__(self, val, parents=()):
        self.val = val
        self.grad = 0.0
        self.parents = parents
        tape.append(self)

    def __add__(self, other):
        return Var(self.val + other.val, [(self, 1.0), (other, 1.0)])

    def __mul__(self, other):
        return Var(self.val * other.val, [(self, other.val), (other, self.val)])

def sin(x):
    return Var(math.sin(x.val), [(x, math.cos(x.val))])

def exp(x):
    e = math.exp(x.val)
    return Var(e, [(x, e)])

def backward(out):
    out.grad = 1.0
    for v in reversed(tape):
        for p, local in v.parents:
            p.grad += v.grad * local

def f(x):
    return exp(x * x + sin(x))

x = Var(1.0)
y = f(x)
backward(y)
print(y.val, x.grad)
```

Here `parents` keeps the parent node and the local derivative with respect to that parent. If you run both the codes you will get same derivative. And see the `backward` function, there is no graph traversal or sorting or anything. It just goes through the tape in reverse order, as I said earlier the tape is already topologically ordered. And the `+=` there is for that adding up from the branches.

So this is more or less what happens when you call `loss.backward()` in pytorch. Of course the real one works with tensors and not scalars, and there are a lot of engineering on top of it, but the core idea is this only.

Now if you think that some magical things might happen when you are dealing with multivariate situation, well sadly not so. You can check out, [link](https://github.com/kingshuk7995/autodiff)

### What about the lazy side? JAX

Now remember we talked about lazy mode and I said `jax.jit` is somewhat like that side. So far everything was eager and tape based, so let's see what happens on the other side, and I think it's actually the more interesting one.

#### Functional way

In `jax`, functions are just functions. Like in maths. You give input, you get output, and nothing else happens. No hidden state, nothing stored inside the tensors, parameters are passed explicitly as arguments.

And then `jax.grad` is also just a function. It takes a function and gives you another function, which is the derivative. Just like the $\frac{\delta}{\delta x}$ operator in maths takes $f$ and gives $f'$.

```python
import jax
import jax.numpy as jnp

def f(x):
    return jnp.exp(x**2 + jnp.sin(x))

df = jax.grad(f)
print(df(1.0))
```

Now compare it with pytorch. There the gradient is not a return value, it's a side effect. You call `.backward()` and the gradients silently gets written into `.grad` of every tensor that needs it, and they even keep adding up if you don't call `zero_grad()`. If you ever forgot `zero_grad()` and wondered why the training went crazy, you know what I mean.

Since `grad` gives a function back, you can do whatever you do with functions. `jax.grad(jax.grad(f))` is the second derivative. `jax.jit(jax.grad(f))` is the compiled derivative. And `jax.jvp` and `jax.vjp` are exactly the forward mode and reverse mode we talked about, `grad` is basically `vjp` on a scalar output.

#### Tracing

Now how does `jax` get the graph without a tape? By tracing. When you `jit` a function, `jax` calls it once, but not with real numbers. It passes some placeholder values, which only know the shape and dtype, and every operation done on them gets recorded. So at the end it has the whole compute graph of the function, which `jax` calls jaxpr. You can see it yourself,

```python
print(jax.make_jaxpr(f)(1.0))
```

and you will see basically the same graph we drew earlier, just in text form.

And here is the point why the functional thing matters. Since the function is pure, the trace is the function. Whatever happened in that one trace will happen every time for the same shapes. So `jax` can trust that graph completely and hand it to the compiler (XLA), which can fuse operations, plan the memory and all.

But it's not free either. As the trace sees placeholders and not values, you can't do python `if` on the value of a traced array inside `jit`, you need things like `jax.lax.cond`. And a `print` inside runs only once while tracing, not on every call. So you have to write in that pure style, otherwise it bites.

#### Autodiff on the traced graph

Now combine the two. Since we have the whole graph as data, differentiation is just another transformation on the graph. `grad` takes the jaxpr of $f$ and gives a new jaxpr for $f'$. Nothing is recorded at runtime, there is no tape at all.

So look at a full training step.

```python
lr = 1e-3

@jax.jit
def step(params, x, y):
    grads = jax.grad(loss_fn)(params, x, y)
    return jax.tree.map(lambda p, g: p - lr * g, params, grads)
```

Here the forward pass, the backward pass and the update all are inside one function, and the whole thing gets traced and compiled as a single program. The compiler sees everything from the input to the updated parameters. So it can fuse the backward with the forward, decide which intermediate values to keep and which to recompute, and all.

Now in pytorch, the same step looks like,

```python
loss = loss_fn(model(x), y)
loss.backward()
opt.step()
opt.zero_grad()
```

And the issue is, `.backward()` is just a method call that can happen anywhere. After the forward, you can do any python thing, call `.backward()` once or twice, with `retain_graph`, or on some other tensor, or not at all. So the framework can never be so sure, while running the forward, where and when the backward will be called. `pytorch 2.x` tried to fix this with `torch.compile`, and to be fair, it does trace the backward ahead of time for the compiled parts. But the moment there is a graph break you get pieces, and the backward is still started from outside by your `.backward()` call, and the eager autograd engine still stitches those pieces together. So the compiler only sees parts of the step, never the whole step like `jax` does.

So overall, in `jax` the graph is something you have before running, and in pytorch the graph is something that comes out after running. That's the main difference, and most of the advantages of `jax` comes from that only.

> **Note:** Now don't take it as `jax` is just better than pytorch in every way. It's not that simple. `torch.compile` has mainly two parts. First is TorchDynamo, which hooks into python itself and reads the bytecode of your function to pull out a graph from it. Then AOTAutograd takes that captured forward graph, and traces the backward from it ahead of time, so both forward and backward gets compiled. So for the parts it captures, pytorch also gets that compiled backward.
>
> The difference is in how they get the graph. `jax` just runs your function with tracers, and it can do that because it makes you promise that the function is pure. Pytorch can't ask for that promise, because pytorch code can be any python. So Dynamo has to read the bytecode, put guards on whatever it assumed (types, shapes, some values) and check those guards on every call, and whenever it finds something it can't handle, it breaks the graph and falls back to normal python. That's why it's not as clean as `jax`.
>
> But that's also the reason pytorch is so easy to use. You write normal python, you debug it with normal `print` and debugger, and `torch.compile` is just something you add on top later. In `jax` you have to write in that pure style from the beginning. So it's a trade-off, not a winner and loser thing.

So hereby you see how autograd works, and that's mostly all of the theory part. There are some parts of memory optimization in many places of this chain. But I think this is the end of this.

Also though I couldn't sadly show an implementation of my own for the JAX style autograd. But, I was working with such one but in C++ with types. That is a different story. Also the autograd is nowhere near finished. There is a lot more stories regarding how the computation is optimized. But those are mostly engineering choices, the fundamental mathematical concepts are presented.