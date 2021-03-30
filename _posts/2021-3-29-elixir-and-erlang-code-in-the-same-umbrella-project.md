---
layout: post
title: "Elixir and Erlang code in the same umbrella project"
thumb_image: https://i.imgur.com/M93AK4d.jpg
tags: [elixir, erlang]
---

There's a simple way to write the Elixir and Erlang codes under the same umbrella project and you can use them together.

# 1. Create a new umbrella project
For this example, I'll create an umbrella project.

{% highlight sh %}
$> mix new ping_beam --umbrella
$> cd ping_beam
$> tree
.
├── apps
├── config
│  └── config.exs
├── mix.exs
└── README.md
{% endhighlight %}

# 2. Create an Elixir project in umbrella
I'll start off by creating an Elixir project inside the `apps` folder.

{% highlight sh %}
$> mix new ping_elixir
{% endhighlight %}

Now, I'll add a `ping` function inside the `PingElixir` module whose only purpose is to print out a message.

{% highlight elixir %}
# apps/ping_elixir/lib/ping_elixir.ex
defmodule PingElixir do
  @moduledoc false

  def ping do
    IO.puts("Ping from Elixir")
  end
end
{% endhighlight %}

# 3. Create an Erlang project in umbrella
I'll create an Erlang project using `rebar3` also inside the `apps` folder.

{% highlight sh %}
$> rebar3 new lib ping_erlang
{% endhighlight %}

As for the Elixir project, I'll also create a module and add a `ping` function there.

{% highlight erlang %}
% apps/ping_erlang/src/ping_erlang.erl
-module(ping_erlang).

-export([ping/0]).

ping() -> <<"Ping from Erlang">>.
{% endhighlight %}

# 4. Reference and use the projects above
For this step, I'll create a new project named `ping` where I'll create a `ping` function and call the modules above.

{% highlight sh %}
$> mix new ping
{% endhighlight %}

Now, I'll change the `mix.exs` adding the `PingElixir` and `ping_erlang` modules as umbrella dependencies.

{% highlight elixir %}
# apps/ping/mix.exs
defmodule Ping.MixProject do
  # project config...

  defp deps do
    [
      {:ping_elixir, in_umbrella: true},
      # As the Erlang project, it's really important to add the manager as rebar3
      {:ping_erlang, in_umbrella: true, manager: :rebar3}
    ]
  end
end
{% endhighlight %}

After that, I'll modify the ping module to call those modules.

{% highlight elixir %}
# apps/ping/lib/ping.ex
defmodule Ping do
  @moduledoc false

  def ping do
    [
      PingElixir.ping(),
      :ping_erlang.ping()
    ]
  end
end
{% endhighlight %}

# 5. Running
And as for the testing of the code, it's pretty simple.

{% highlight sh %}
$> iex -S mix

iex> PingElixir.ping()
"Ping from Elixir"

iex> :ping_erlang.ping()
"Ping from Erlang"

iex> Ping.ping()
["Ping from Elixir", "Ping from Erlang"]
{% endhighlight %}

# 6. Conclusion
It's a really simple process! Just put the Elixir and Erlang codes together. And with a few lines of code, you'll get it done.

Check the [repo](https://github.com/wesleimp/ping_beam) for more.

Thanks!