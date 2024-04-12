---
theme: envek
highlighter: shiki
lineNumbers: false
title: Threads, callbacks, and execution context in Ruby
info: |
  # Threads, callbacks, and execution context in Ruby

  When you provide a block to a function in Ruby, do you know when and where that block will be executed? What is safe to do inside the block, and what is dangerous? Let’s take a look at various code examples and understand what dragons are hidden in Ruby dungeons.
drawings:
  persist: false
download: true
mdc: true
talkDurationMinutes: 28
progressBarStartSlide: 2
---

# Threads, callbacks, and <small class="text-90%">execution context in Ruby</small>

<div class="absolute bottom-0 left-0 w-full px-10 py-8 grid grid-cols-2 justify-items-stretch items-end gap-4">
  <div class="text-left">
    Andrey Novikov, Evil Martians<br />
    <small><a href="https://2024.rubyconf.au/">RubyConf AU 2024</a></small><br />
    <small><time datetime="2024-04-12">12 April 2024</time></small>
  </div>

  <div class="w-28 h-28 object-contain justify-self-end">
    <a href="https://evilmartians.com/"><img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="block dark:hidden" /><img alt="Evil Martians" src="/images/02_Evil-Martians_Logo_v2.1_RGB_for-Dark-BG.svg" class="hidden dark:block" /></a>
  </div>
</div>

<div class="absolute top-0 left-0 w-full h-36 p-4 text-center">
<a href="https://2024.rubyconf.au/"><img alt="RubyConf AU 2024" src="/images/RubyConf-Coat-of-Arms-2024.webp" class="mx-auto max-h-36 object-contain drop-shadow-sm" /></a>
</div>


<style>
  a {
    border-bottom: none !important;
  }
</style>

<!--
Hi everyone! Nice to meet you here in Sydney.
-->

---
layout: image-right
image: /images/5222215177628405026_121.jpg
class: annotated-list
---

# About me

Hi, I'm Andrey

- Back-end engineer at Evil Martians

- Writing Ruby, Go, and whatever

  SQL, Dockerfiles, TypeScript, bash…

- Love open-source software

  Created a few little Ruby gems

- Living in Japan for 1 year already

- Driving a moped

<div class="absolute bottom-0 w-80% max-w-80% max-h-24 text-center">
<img src="/images/01_Evil-Martians_Logo_Lurkers_v2.0_on-Transparent.png" class="max-h-24 object-contain  mx-auto" />
</div>

<!--
My name is Andrey. I am a software developer, using Ruby for around 14 years now, and also all technologies around, I'm fan of Postgres, can deploy apps to Kubernetes, and do other scary things.
 
Initially I am from Russia, but recently, in 2022, I moved to Japan, partly because of _reasons_, the horrible things Russian authorities do, and partly because I like Japan so much. I am happily residing in Japan since then with my family, enjoying Japanese order and safety, and calm road traffic while riding on a moped around my neighborhood.

So if you're thinking about moving to Japan, let's chat about it after the talk, I've got some experience to share.
-->

---
layout: image
image: /images/worldmap.png
---


<!--
And I'm very happy to finally get to Australia. And when I saw RubyConf Australia Call for speakers I looked at this Mercator-distorted world map and thought: “Wow, Japan and Australia seems to be not so far from each other… ”
-->

---
layout: image
image: /images/distance-sydney-osaka.png
---

<!--
And, well, relatively, yes, they are. But Pacific ocean is so huge, and Australia itself is so massive. It is almost 8 thousand kilometers from Sydney to Osaka!

This kind of world view distortion caused by the fact that we are so used to maps based on Mercator projection, which makes everything that is closer to Earth poles to seem much bigger than it is. It is bugging me a lot.
-->

---

<a href="https://evilmartians.com/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks">
<img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="block dark:hidden object-contain text-center m-auto max-h-100" />
<img alt="Evil Martians" src="/images/02_Evil-Martians_Logo_v2.1_RGB_for-Dark-BG.svg" class="hidden dark:block object-contain text-center m-auto max-h-100" />
</a>

<p class="text-2xl text-center"><a href="https://evilmartians.com">evilmartians.com</a></p>

<!--
Also, I have to confess, actually I'm an alien. An Evil Martian agent here on Earth.

Evil Martians is distributed web development agency helping companies all sizes to live and prosper, we help small startups grow effectively, big enterprises to speed-up their monoliths and test hypotheses. And we are focusing on developer tools, open source, and effective software development practices.

Australia is still is a terra incognita for us and we are looking forward to help Australian companies to build better software. So, if you need help with performance, scaling, architecture, implementing best practices, or just need to strengthen your team with experienced engineers, feel free to reach out to us, we now have a few engineers in Japan, me included, in almost the same timezone as yours.
-->

---

# Martian Open Source

<div class="grid grid-cols-4 grid-rows-2 gap-4">
  <a href="https://ruby-next.github.io/">
    <figure>
      <img alt="Ruby Next" src="/images/martian-oss/ruby-next.png" class="object-contain h-32 mx-auto" />
      <figcaption>Ruby Next makes modern Ruby code run in older versions and alternative implementations</figcaption>
    </figure>
  </a>
  <a href="https://github.com/yabeda-rb/yabeda">
    <figure>
      <img alt="Yabeda" src="/images/martian-oss/yabeda.svg" class="object-contain h-32 mx-auto" />
      <figcaption>Yabeda: Ruby application instrumentation framework</figcaption>
    </figure>
  </a>
  <a href="https://github.com/evilmartians/lefthook">
    <figure>
      <img alt="LeftHook" src="/images/martian-oss/lefthook.svg" class="object-contain h-32 mx-auto" />
      <figcaption>Lefthook: git hooks manager</figcaption>
    </figure>
  </a>
  <a href="https://anycable.io/">
    <figure>
      <img alt="AnyCable" src="/images/martian-oss/anycable.svg" class="object-contain h-32 mx-auto" />
      <figcaption>AnyCable: Polyglot replacement for ActionCable server</figcaption>
    </figure>
  </a>
  <a href="https://postcss.org/">
    <figure>
      <img alt="PostCSS" src="/images/martian-oss/postcss.svg" class="object-contain h-32 mx-auto" />
      <figcaption>PostCSS: A tool for transforming CSS with JavaScript</figcaption>
    </figure>
  </a>
  <a href="https://imgproxy.net/">
    <figure>
      <img alt="Imgproxy" src="/images/martian-oss/imgproxy-light.svg" class="object-contain h-32 mx-auto block dark:hidden" />
      <img alt="Imgproxy" src="/images/martian-oss/imgproxy-dark.svg" class="object-contain h-32 mx-auto hidden dark:block" />
      <figcaption>Imgproxy: Fast and secure standalone server for resizing and converting remote images</figcaption>
    </figure>
  </a>
  <a href="https://github.com/DarthSim/overmind">
    <figure>
      <img alt="Overmind" src="/images/martian-oss/overmind.svg" class="object-contain h-32 mx-auto" />
      <figcaption>Overmind: Process manager for Procfile-based applications and tmux </figcaption>
    </figure>
  </a>
  <a href="https://evilmartians.com/oss">
    <figure>
      <div class="h-32 text-2xl flex items-center justify-center">
        <qr-code-vue value="https://evilmartians.com/oss" class="object-contain w-full h-full mx-auto p-4 dark:invert" render-as="svg" margin="1" />
      </div>
      <figcaption style="font-size: 1rem; margin-top: 0; line-height: 1.25rem;">Even more at evilmartians.com/oss</figcaption>
    </figure>
  </a>
</div>

<style>
  a { border-bottom: none !important; }
  figcaption {
    margin-top: 0.5rem;
    font-size: 0.6rem;
    line-height: 1rem;
    text-align: center;
  }
</style>

<!--
One thing that we truly love is Open Source. We love to use it, and we also love to give back enhancements to the community. We eager to share results of our work as a ruby gem or npm package, it will help us in the first place to re-use our own solutions, and we can help others to solve their problems and often we will get feedback or patches back. It is a win-win.

And for many years we've created literally over a hundred of open source products, big and small, famous and not so. Very probably your application already depends on a few martian Ruby gems, so check out your Gemfile and count how many of them you have.

For example I personally made Yabeda, a mini framework for instrumenting Ruby applications for better monitoring and observability.

Some open source products have even grown into commercial products, like anycable or imgproxy, still staying open source at the time.
-->

---
layout: cover
---

# Threads, <span v-mark.circle.orange>callbacks</span>, and <small class="text-90%">execution context in Ruby</small>

<!--
But let's get back to world view distortions, but now let's take a look at how we, programmers, can sometimes undervalue even most basic language constructs, just because we are so used to them. [click] Let's talk about callbacks in Ruby.
-->

---

## Let's talk about callbacks…

<div class="text-2xl mt-10">

What callbacks?

```ruby
class User < ApplicationRecord
  after_create :send_welcome_email
end
```

<v-click>

No, not Rails callbacks,<br />no-no-no!

<iframe src="https://giphy.com/embed/12XMGIWtrHBl5e" class="absolute bottom-5 right-0 w-50% h-50%" frameBorder="0" allowFullScreen></iframe>
</v-click>
</div>

<!--
What callbacks? [click] No, not these callbacks in Rails models! I know, many people have had bad experience with them, me included, but this is a whole separate topic, I only want you to **not** do things like this example, never ever. Don't send emails from your models, I beg you. Please use ActiveRecord callbacks only for keeping data consistency.
-->

---

## Let's talk about **blocks** as callbacks


<div class="grid grid-cols-2 grid-rows-1 gap-4">

```ruby {all}{class:'!children:text-xl'}

3.times do
  puts "Hello RubyConf AU!"

end
```

```ruby {all}{class:'!children:text-xl'}
i = 0
while i < 3
  puts "Hello RubyConf AU!"
  i += 1
end
```

</div>

It feels like these two code samples are identical, right?

<v-click>

<p class="mt-20 mx-auto px-4 py-2 border-2 border-red-500 text-center text-5xl font-bold">
WRONG!
</p>

</v-click>

<!-- Let's talk about Ruby blocks. I believe that it is so idiomatic and so common to use `times` method on integers in case if something needed to be repeated several times, that no one really realize,  that while it does the job, [click] it is technically is very different from `for` loop in Ruby and other languages as well.
-->

---

## Blocks are separate pieces of code

<div class="text-2xl mt-10">

Block is separate entity, that is passed to `times` method as an argument and got **called** by it.

```ruby{all|1-3|5-7}
greet = proc do
  puts "Hello, RubyConf Australia!"
end

3.times { |i| greet.call(i) }
# or
3.times(&greet)
```
</div>

<!--
Block in Ruby isn't just a sequence of instructions between `do` and `end`, but internally is much more similar to functions or methods. [click] First, you declare a block, [click] and then it got called by `times` method several times, passing counter as an argument.
-->

---

## Blocks are called by methods

![](/images/Ruby_under_a_microscope_calling_a_block.png){class="object-contain max-w-full max-h-85 mx-auto"}


Illustration from the [Ruby under a microscope](https://patshaughnessy.net/ruby-under-a-microscope) book, chapter 2.

<qr-code url="https://patshaughnessy.net/ruby-under-a-microscope" caption="Ruby under a microscope" class="w-36 absolute bottom-50px right-40px" />

<!--
And internally executing a block is very much the same as calling method: CRuby needs to create a special C data structure, push it to the stack along with parameters, if any, move code sequence pointer there. A lot.

I won't deep dive into internals today, instead I would recommend you to read awesome book called “Ruby Under a Microscope” – it is really enlightening reading if you are familiar with how computers work on a low level.
-->

---

## And there is difference in performance

Empty `while` loop is twice as faster than `times` with a block.

```ruby {all}{class:'!children:text-xs'}
Benchmark.ips do |x|
  x.report("blocks") { 1_000_000.times { |i| i } }
  x.report("while") { i = 0; while i < 1_000_000; i += 1; end  }
end

Warming up --------------------------------------
              blocks     4.000 i/100ms
               while     9.000 i/100ms
Calculating -------------------------------------
              blocks     40.186 (± 0.0%) i/s -    204.000 in   5.076671s
               while     89.914 (± 1.1%) i/s -    450.000 in   5.005051s
```

However, **difference is negligible with real workloads**.

And it is not a point of this talk…

<!--
All these extra steps to execute a block of course make it a bit slower than just a `for` loop. But in real world applications, where you have to do some real work in a loop, something more than just incrementing a counter, this difference become negligible. Don't sacrifice readability by looking on micro-benchmark results. And it also not is the point of this talk either.

The point is…
-->

---
layout: statement
---

# Blocks ARE callbacks

We often use blocks as callbacks to **hook** our own behavior for someone's else code.

<!--
that in Ruby blocks are used as callbacks. If, for example, in JavaScript or Golang you would pass a function as an argument to another function to be called back, Ruby has a special entity and special syntax for doing that. And this is great. Ruby has callbacks as a first-class citizens, unlike many other languages.
-->

---
class: text-xl annotated-list
---

## Blocks are closures as well

<div class="grid grid-cols-2 grid-rows-1 gap-4">

<div>

  - A Ruby code to execute

  - Environment to be executed in:

    Blocks can access local variables and `self` at the place where block was **created**

<hr class="my-5">

> …blocks have in some sense a dual personality. On the one hand, they behave like separate functions: you can call them and pass them arguments just as you would with any function. On the other hand, they are part of the surrounding function or method.
>
> _Ruby Under a Microscope_{class="text-sm block text-right"}
</div>

```ruby {all|2-5|8-9}
def greeter
  conf = "RubyConf AU 2024"
  proc do
    puts "Hello #{conf}"
  end
end

greeter.call
# => Hello RubyConf AU 2024

# LAMBDA CALCULUS FTW!!!!1
```

</div>

<!--
What is important to know, is that a block is more than just a kind-of function. It is also a closure.

Every Ruby block can be represented as consisting of two parts: the code to execute and the environment where this code should be executed.

First part is simple: you put instructions between `do` and `end`, and they become the block body.

Environment is a bit more tricky: [click] block remembers all local variables that were defined before its definition. Also it remembers object in that scope it was created, the `self` object.

[click] And when you call a block, it executes in this context, magically referencing variables defined very far away (and maybe very far ago) from the place it was executed.
-->

---
class: text-xl annotated-list
---

## Blocks are closures as well

  - A Ruby code to execute

  - Environment to be executed in:

    Blocks can access local variables, self, etc at the place where block was _created_

<p class="mt-40 mx-auto px-4 py-2 text-red-500 text-center text-5xl font-bold">
BUT
</p>

<p class="mt-20 mx-auto px-4 py-12 border-2 border-red-500 text-center text-xl font-bold">
Some environments can be changed unexpectedly
</p>

<!--
**But**, this environment isn't immutable, it can be changed. Moreover, there are more things that can be included into the term “environment”. And this is the thing I want to focus on today.
-->

---

## Environment changes: `self`

`instance_exec` and `class_exec`

<div class="grid grid-cols-2 grid-rows-1 gap-4">

```ruby {all|all|6-8|7|none|none|none|none|none}{at:'1'}
class Conference
  def title
    "RubyConf AU 2024"
  end

  def say(&block)
    puts instance_exec(&block)
  end
end
```

```ruby {all|none|none|none|all|4-8|6|none|2,6}{at:'1'}
class Speaker
  attr_reader :name, :conf

  def greet
    conf.say do
      "Hello #{conf.title}, I'm #{name}"
    end
  end
end
```
</div>

<RenderWhen context="print">
```text {all}{at:'1',class:'!children:text-base'}
undefined local variable or method `conf' for an instance of Conference
```
<template #fallback>
```text {hide|hide|hide|hide|hide|hide|hide|all|all}{at:'1',class:'!children:text-base'}
undefined local variable or method `conf' for an instance of Conference
```
</template>
</RenderWhen>

<!--
Most frequently changed thing in the environment is the binding, the `self` object. Usually this change is done on purpose, to allow building nice domain-specific languages. However, if you don't aware, that method that receives your block will use `instance_exec` you will be surprised by cryptic messages about undefined methods.

[click] For example, [click] let's take a look at a method, that receives a block, [click] and executes it in the context of this method's object using `instance_exec`. [click]

[click] If the code using this API will try to define a block, [click] that uses `self` of the block creation place, [click] it will fail with mysterious error message, complaining that it can't find the code that obviously [click] is right here. [draw!]

This can blow your mind, be prepared!
-->

---

## Environment changes: local vars

Someone can change variables enclosed in the block's environment…


```ruby {all|1-6|8-12}
conf = "RubyConf AU 2024"
greeter = proc do
  puts "Hello #{conf}"
end
greeter.call
# => Hello RubyConf AU 2024

# later…
conf = "RubyConf AU 2025"
greeter.call
# => Hello RubyConf AU 2025
```

<!--
Okay, you can think that as block is a closure, you can save current `self` into a local variable and reference it later. It will work, but is not reliable either.

[click] Say, you have a block that uses a local variable, [click] and then you reassign this variable, and then you call the block again. **But** you will see that block that was defined earlier, will use the new value of the variable, not the one it was created with.
-->

---

## Environment changes: local vars

It is done to allow multiple blocks to work with a single enclosed variable.

<div class="grid grid-cols-2 grid-rows-1 gap-4">

![How local variables enclosed by blocks under the hood](/images/local-vars-in-blocks-underneath.png)

<div>

```ruby {all}{class:'!children:text-sm'}
i = 0

increment_function = lambda do
  puts "Incrementing from #{i} to #{i+1}"
  i += 1
end

decrement_function = lambda do
  i -= 1
  puts "Decrementing from #{i+1} to #{i}"
end
```

Illistration and code: Ruby under microscope{class="text-sm text-right font-italic"}
</div>

</div>

<!--
This behaviour is truly mind-blowing, but still perfectly valid. The reason for it is to allow multiple blocks to share access to common variables, like working with a counter like in this example from the same Ruby under a microscope book.
-->

---

## Blocks can be called from other threads

```ruby {all|8-10|9,12}{class:'!children:text-sm'}
result = []

work = proc do |arg|
  # Can you tell which thread is executing me?
  result << arg # I'm closure, I can do that!
end

Thread.new do
  work.call "new thread"
end

work.call "main thread"

# And guess what's inside result? 🫠
```

Can you feel how thread-safety problems are coming?

<!--
Let's explore other things that we can include into the term “environment”. One example of this is the current thread that is executing the block.

[click] Fun fact, that thread creation itself uses blocks to define a body of the new thread.

[click] Anyway, you can call a block from different threads, even from multiple threads at the same time, and enclosed variables will be accessible.

And it is fine as long as you don't try to mutate these shared objects like in this example, as here be dragons.
-->

---

## Different threads

What if we have some units of work that relies on thread local state…

```ruby {all|2,8|1-5|7-11}
work1 = proc do
  Thread.current[:state] ||= 'work1'
  raise "Unexpected!" if Thread.current[:state] != 'work1'
  SecureRandom.hex(4)
end

work2 = proc do
  Thread.current[:state] ||= 'work2'
  raise "Unexpected!" if Thread.current[:state] != 'work2'
  SecureRandom.hex(8)
end
```

<!--
The real problems will arise if you will try to rely on a thread-local data from a block that can be executed from different threads. [click] To be specific, if you try to use `Thread.current` with the same key from [click] one block and [click] from the other…
-->

---

## Different threads

And then try to execute them using concurrent-ruby `Promise` that utilizes thread pools to execute blocks…

```ruby {all}
promises = 100.times.flat_map do
  Concurrent::Promise.execute(&work1)
  Concurrent::Promise.execute(&work2)
end

Concurrent::Promise.zip(*promises).value!
#=> Unexpected! (RuntimeError)
# But it also might be okay (chances are low though)
```

<!--
Because sometimes there will be no guarantee that the same thread will be used to execute the same block, especially if you use higher-level concurrencty primitives like thread pools or promises from the `concurrent-ruby` gem.

Or, most probably if some library uses them internally, and you don't aware of that…
-->

---

## Example: NATS client

NATS is a modern, simple, secure and performant message communications system for microservice world.

```ruby
nats = NATS.connect("demo.nats.io")

nats.subscribe("service") do |msg|
  msg.respond("pong")
end
```

In earlier versions every subscription was executed in its own separate thread.

<a href="https://github.com/nats-io/nats-pure.rb"><img alt="nats-pure.rb" src="/images/og-nats-pure.png" class="absolute bottom-0 object-contain max-h-36" /></a>

<qr-code url="https://github.com/nats-io/nats-pure.rb" caption="nats-pure gem" class="w-36 absolute bottom-10px right-10px" />

<!--
And this is not a theoretical problem.

For example, let's take a look at NATS client for Ruby, `nats-pure` gem. It uses blocks as a callbacks to process incoming messages from subscriptions.

Okay, what is NATS? Who knows?

It is a performant and simple to set up message broker written in Go. It is very fast, very simple to set up (anyone who tried to deploy Kafka to Kubernetes will understand), and have many features aside of pub/sub: persistend message queues, for example. Basically, NATS is nice.
-->

---

## Example: NATS client

<div class="grid grid-cols-2 gap-4">

<div>

Before:

```ruby {all|all|none}{at:'1',class:'!children:text-sm'}
def subscribe(topic, &callback)
  # Sent subscribe request to NATS
  Thread.new do
    while msg = message_queues[topic].pop do
      callback.call(msg)
    end
  end
end

while msg = incoming_messages.pop do
  message_queues[msg.topic].push(msg)
end
```
</div>

<div>

After:

```ruby {all|none|all}{at:'1',class:'!children:text-sm'}
def subscribe(topic, &callback)
  # Sent subscribe request to NATS
  callbacks[topic] = callback
end



while msg = incoming_messages.pop
  Concurrent::Promise.execute do
    callbacks[topic].call(msg)
  end
end
```
</div>
</div>

<!--
Here is the story: [click] before version 2.3.0, NATS Ruby client created a separate Thread for every subscription, and for every new message for the same subscription, callback was executed in the same dedicated thread. And if you would try to use `Thread.current` in the callback, it would work then.

But then I came there and… [click] changed implementation to use a thread pool, and now every callback is executed in some thread from the pool. And you can't rely on `Thread.current` anymore.
-->

---
layout: image
image: /images/nats-pure-thread-pool-pull-request.png
---

<!--
Why I did that? Because of performance, of course!

As you should know, context switching is **very** expensive operation, and creating a new thread for every subscription to handle occasionally received messages is a waste of resources.

Moreover, there is a limit on number of threads that can be created in a process, and if you have a lot of subscriptions, you can easily hit this limit. On my machine, Ruby crashed after around 30_000 threads were created.

With fixed size thread pool, you can control the number of threads, and reuse them, and avoid context switching overhead. My benchmarks showed that for tens of thousands of subscriptions, thread pool is several times faster than individual threads, solely by reducing context switching overhead.
-->

---

## Can I use `Thread.current` in NATS callbacks?

<div class="text-2xl mt-10">

```ruby
nats = NATS.connect("demo.nats.io")

nats.subscribe("service") do |msg|
  Thread.current[:handled] ||= 0
  Thread.current[:handled] += 1
end
```
</div>

<div class="text-2xl mt-10 mb-8">

Q: So, can I?

A: It depends on gem version! 🤯 <small>(you could before version 2.3.0)</small>
</div>

Hint: better not to anyway!

<!--
After this change got merged I started to think about whether behaviour of the NATS client changed or not. And I realized that it did, and, frankly speaking, it could be a breaking change, but only if this behaviour was specified somewhere.

But I wasn't specified, there was no promise that a subscription callback will be executed in the same thread every time, so it can be treated as a mere internal implementation detail… 🤷‍♂️ No need to bump major version, yay!

Hopefully no one used `Thread.current` in the NATS callbacks, it seem to be not that common technique anyway.
-->

---

## Where you can find thread pools?

- Puma
- Sidekiq
- ActiveRecord `load_async`
- NATS client
- …and many more

Good thing is that you don't have to care about them most of the time.

<div class="mx-auto px-4 my-2 border-2 border-blue-500">

**Pro Tip:**

In Rails use `ActiveSupport::CurrentAttributes` instead of `Thread.current`!

</div>

<!--
What is important to understand, in my opinion, is that in real world applications, your code is working inside of some thread pool, will it be Puma application server or Sidekiq worker, so… just don't use `Thread.current`.

Use “current attributes” in Rails instead, for example.
-->

---
layout: statement
---

# How to understand?

When and where and how a block will be called?

<v-click>
<div class="text-center text-5xl mt-20 mb-8">

No easy way to know! 😭

</div>

Only by reading API documentation and source code.
</v-click>

<!--
The most disappointing thing here is [click] there is no easy way to understand, whether your block will be executed within `instance_exec`, or in a separate thread, or will it be executed at all.

The only way is to read and grok the source code of the library you are using, which is a good practice anyway.
-->

---

# Recap

<div class="mt-5 text-2xl">

- Blocks are primarily used as callbacks
- Blocks can be executed in a different threads
- And this thread can be different each time!
  - Think twice before using `Thread.current`
- Blocks can be executed with a different receiver
- Even captured local variables can be changed

<hr class="my-5" />

**And you have to remember that!** {class="text-3xl"}

</div>

<!--
I hope that you enjoyed my little tour into the Ruby blocks world.

And may the new knowledge help you to avoid some bugs in your applications. Amen.
-->

---

# Thank you!

<div class="grid grid-cols-[8rem_3fr_4fr] mt-12 gap-2">

<div class="justify-self-start">
<img alt="Andrey Novikov" src="https://secure.gravatar.com/avatar/d0e95abdd0aed671ebd0920c16d393d4?s=512" class="w-32 h-32 object-contain" />
</div>

<ul class="list-none">
<li><a href="https://github.com/Envek"><logos-github-icon class="dark:invert" /> @Envek</a></li>
<li><a href="https://twitter.com/Envek"><logos-twitter /> @Envek</a></li>
<li><a href="https://facebook.com/Envek"><logos-facebook /> @Envek</a></li>
<li><a href="https://t.me/envek"><logos-telegram /> @Envek</a></li>
</ul>

<div>
<qr-code url="https://github.com/Envek" caption="github.com/Envek" class="w-32 mt-2" />
</div>

<div class="justify-self-start">
<a href="https://evilmartians.com/"><img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="w-32 h-32 object-contain block dark:hidden" /><img alt="Evil Martians" src="/images/02_Evil-Martians_Logo_v2.1_RGB_for-Dark-BG.svg" class="w-32 h-32 object-contain hidden dark:block" /></a>
</div>

<div>

- <logos-github-icon class="dark:invert" /> [@evilmartians](https://github.com/evilmartians?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks)
- <logos-twitter /> [@evilmartians](https://twitter.com/evilmartians/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks)
- <logos-linkedin-icon /> [@evil-martians](https://www.linkedin.com/company/evil-martians/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks)
- <logos-instagram-icon class="dark:invert" /> [@evil.martians](https://www.instagram.com/evil.martians/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks)
</div>

<div>
<qr-code url="https://evilmartians.com/" caption="evilmartians.com" class="w-32 mt-2" />
</div>

<div class="col-span-3">

Our awesome blog: [evilmartians.com/chronicles](https://evilmartians.com/chronicles/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks)!

<p class="text-sm">See these slides at <a href="https://envek.github.io/rubyconfau-threads-callbacks/">envek.github.io/rubyconfau-threads-callbacks</a></p>

</div>
</div>

<style>
  ul a { border-bottom: none !important; }
  ul { list-style-type: none !important; }
  ul li { margin-left: 0; padding-left: 0; }
</style>

<!--
That's it! Please check out Evil Martians blog, we have a lot of interesting articles about Ruby, Rails, frontend, design and other things.

Also, I have Martian stickers, so find me after the talk and grab one. And also let's have chat about anything: be it Ruby or Japan or motorcycles…

Don't hesitate to reach me in social media and ask questions after the conference.

Thank you a lot for your attention! Thanks to Ruby Australia for organizing this amazing event, choosing this talk and giving me a chance to speak here.

And let the lunch begin!
-->
