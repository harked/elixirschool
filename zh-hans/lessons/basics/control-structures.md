---
version: 1.1.1
title: 控制语句
---

这篇教程，我们将学习 Elixir 语言中的控制语句。

{% include toc.html %}

## `if` 和 `unless`

你之前可能遇到过 `if/2`了，如果你使用过 Ruby，也会很熟悉 `unless/2`。它们在 Elixir 使用方式也一样，只不过它们在 Elixir 里是宏定义，不是语言本身的语句。你可以在 [Kernel 模块](https://hexdocs.pm/elixir/Kernel.html) 找到它们的实现。

需要注意的是，Elixir 中唯一为假的值是 `nil` 和 布尔值 `false`。

```elixir
iex> if String.valid?("Hello") do
...>   "Valid string!"
...> else
...>   "Invalid string."
...> end
"Valid string!"

iex> if "a string value" do
...>   "Truthy"
...> end
"Truthy"
```

`unless/2` 使用方法和 `if/2` 一样，不过只有当判断为否才会继续执行：

```elixir
iex> unless is_integer("hello") do
...>   "Not an Int"
...> end
"Not an Int"
```

## `case`

如果需要匹配多个模式，我们可使用 `case`：

```elixir
iex> case {:ok, "Hello World"} do
...>   {:ok, result} -> result
...>   {:error} -> "Uh oh!"
...>   _ -> "Catch all"
...> end
"Hello World"
```

`_` 变量是 `case/` 语句重要的一项，如果没有 `_`，所有模式都无法匹配的时候会抛出异常：

```elixir
iex> case :even do
...>   :odd -> "Odd"
...> end
** (CaseClauseError) no case clause matching: :even

iex> case :even do
...>   :odd -> "Odd"
...>   _ -> "Not Odd"
...> end
"Not Odd"
```

可以把 `_` 想象成最后的 `else`，会匹配任何东西。

因为 `case/2` 依赖模式匹配，所以之前所有的有关模式匹配的规则和限制在这里都适用。
如果你想匹配已经定义的变量，一定要使用 pin 操作符 `^/1`：

```elixir
iex> pie = 3.14
 3.14
iex> case "cherry pie" do
...>   ^pie -> "Not so tasty"
...>   pie -> "I bet #{pie} is tasty"
...> end
"I bet cherry pie is tasty"
```

`case/2` 还有一个很酷的特性：它支持卫兵表达式（Guard Clause）：

_这个例子直接取自 Elixir 官方指南的[上手教程](https://elixir-lang.org/getting-started/case-cond-and-if.html#case)_

```elixir
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Won't match"
...> end
"Will match"
```

参考官方的文档来看[卫兵支持的表达式](https://hexdocs.pm/elixir/guards.html#list-of-allowed-expressions)

## `cond`

当我们需要匹配条件而不是值的时候，可以使用 `cond/1`，这和其他语言的 `else if` 或者 `elsif` 相似：

_这个例子直接取自 Elixir 官方教程的 [上手指南](https://elixir-lang.org/getting-started/case-cond-and-if.html#cond)_

```elixir
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

和 `case/2` 一样，`cond/1` 语句如果没有任何匹配会抛出异常。我们可以定义一个 `true` 的条件放到最后，来处理这种意外：

```elixir
iex> cond do
...>   7 + 1 == 0 -> "Incorrect"
...>   true -> "Catch all"
...> end
"Catch all"
```

## `with`

当我们需要使用嵌套的`case/2`语句,或者无法完全地使用管道符号连接在一起时,这时`with/1`是非常有用的。`with/1`表达式由关键字(keyword)、生成器(generator)和最后一个表达式(expression)组成。

我们将会在[list comprehensions lesson](../comprehensions/)谈论更多关于生成器的知识,现在我们只需要知道它们使用[pattern matching](../pattern-matching/)将`<-`右侧与左侧进行比较。

我们将从一个简单的`with/1`例子开始,然后再探索更多的东西:

```elixir
iex> user = %{first: "Sean", last: "Callan"}
%{first: "Sean", last: "Callan"}
iex> with {:ok, first} <- Map.fetch(user, :first),
...>      {:ok, last} <- Map.fetch(user, :last),
...>      do: last <> ", " <> first
"Callan, Sean"
```

如果表达式匹配失败,将会返回这个匹配失败的值:

```elixir
iex> user = %{first: "doomspork"}
%{first: "doomspork"}
iex> with {:ok, first} <- Map.fetch(user, :first),
...>      {:ok, last} <- Map.fetch(user, :last),
...>      do: last <> ", " <> first
:error
```

现在我们来看一个不带`with/1`并且复杂例子,看看我们是如何重构它的:

```elixir
case Repo.insert(changeset) do
  {:ok, user} ->
    case Guardian.encode_and_sign(user, :token, claims) do
      {:ok, token, full_claims} ->
        important_stuff(token, full_claims)

      error ->
        error
    end

  error ->
    error
end
```

当我们引入`with/1`时,我们可以获得更加简洁并且容易理解的代码:

```elixir
with {:ok, user} <- Repo.insert(changeset),
     {:ok, token, full_claims} <- Guardian.encode_and_sign(user, :token, claims) do
  important_stuff(token, full_claims)
end
```

在Elixir 1.3中,`with/1`语句能够支持`else`:

```elixir
import Integer

m = %{a: 1, c: 3}

a =
  with {:ok, number} <- Map.fetch(m, :a),
    true <- is_even(number) do
      IO.puts "#{number} divided by 2 is #{div(number, 2)}"
      :even
  else
    :error ->
      IO.puts("We don't have this item in map")
      :error

    _ ->
      IO.puts("It is odd")
      :odd
  end
```

使用`case`这种模式匹配的方式来处理错误是非常有好处的,第一个未能够匹配表达式的值将会被传递。
