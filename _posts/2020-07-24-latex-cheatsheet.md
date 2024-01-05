---
layout: post
title: LaTex备忘用例
date: 2020-07-24
tags: [latex, cheatsheet]
comments: true
---

[更多细节查询此网站](http://www.uinio.com/Math/LaTex)。

<!-- more -->

公式块上下使用`$$`标明，内联公式则用`$`。

不要让公式出现在文章摘要里。（Jekyll的默认文章首页摘要是第一段。手动摘要分割线是`<!-- more -->`，并且有摘要的字符数限制。）


Block math test

```
$$
\begin{align*}
y = y(x,t) &= A e^{i\theta} \\
&= A (\cos \theta + i \sin \theta) \\
&= A (\cos(kx - \omega t) + i \sin(kx - \omega t)) \\
&= A\cos(kx - \omega t) + i A\sin(kx - \omega t)  \\
&= A\cos \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big) + i A\sin \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big)  \\
&= A\cos \frac{2\pi}{\lambda} (x - v t) + i A\sin \frac{2\pi}{\lambda} (x - v t)
\end{align*}
$$

```

$$
\begin{align*}
y = y(x,t) &= A e^{i\theta} \\
&= A (\cos \theta + i \sin \theta) \\
&= A (\cos(kx - \omega t) + i \sin(kx - \omega t)) \\
&= A\cos(kx - \omega t) + i A\sin(kx - \omega t)  \\
&= A\cos \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big) + i A\sin \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big)  \\
&= A\cos \frac{2\pi}{\lambda} (x - v t) + i A\sin \frac{2\pi}{\lambda} (x - v t)
\end{align*}
$$

Inline math test `$\lim_{x \to \infty} \exp(-x) = 0$`, 
$\lim_{x \to \infty} \exp(-x) = 0$

