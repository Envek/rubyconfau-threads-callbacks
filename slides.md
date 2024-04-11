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

<!-- Hi everyone! -->

---
layout: image-right
image: ./images/5222215177628405026_121.jpg
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

My name is Andrey, I initially is from Russia, but in 2022 I moved to Japan where happily reside since then. And I'm very happy to finally get to Australia. And you know what? I thought that Japan and Australia are closer to each other than they are. But Pacific ocean is really huge. This kind of world view distortion is bugging me a lot.
-->

---

<a href="https://evilmartians.com/?utm_source=rubyconfau&utm_medium=slides&utm_campaign=threads-callbacks">
<img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="block dark:hidden object-contain text-center m-auto max-h-100" />
<img alt="Evil Martians" src="/images/02_Evil-Martians_Logo_v2.1_RGB_for-Dark-BG.svg" class="hidden dark:block object-contain text-center m-auto max-h-100" />
</a>

<p class="text-2xl text-center"><a href="https://evilmartians.com">evilmartians.com</a></p>

<!--

And, I have to confess, I'm an alien. An Evil Martian agent here on Earth. I came to you from northern hemisphere with peace.

Evil Martians is distributed web development agency helping companies all sizes to live and prosper, we help small startups grow effectively, big enterprises to speed-up their monoliths and test hypotheses. And we focusing on developer tools, open source, and effective software development practices.

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
And for many years we've created a ton of open source products, big and small, famous and not so. Very probably your project will depend on a handful of martian open source Ruby gems.
-->


---
layout: cover
---

# Threads, <span v-mark.circle.orange>callbacks</span>, and <small class="text-90%">execution context in Ruby</small>

<!--
But let's get to view distortions, but now let's take a look at how programmers can sometimes misunderstand even most basic language constructs. Let's talk about callbacks.
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

<!-- No, not these callbacks! I know many people have had bad experience with them, me included. -->

---

## Let's talk about **blocks** as callbacks


<div class="grid grid-cols-2 grid-rows-1 gap-4">

```ruby {1-5}{class:'!children:text-xl'}

3.times do
  puts "Hello RubyConf AU!"

end
```

```ruby {1-5}{class:'!children:text-xl'}
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

---

## Blocks are separate pieces of code

<div class="text-2xl mt-10">

Block is separate entity, that is passed to `times` method as an argument and got **called** by it.

```ruby
greet = proc do
  puts "Hello, RubyConf Australia!"
end

3.times { |i| greet.call(i) }
# or
3.times(&greet)
```
</div>

---

## Blocks are called by methods

![](/images/Ruby_under_a_microscope_calling_a_block.png){class="object-contain max-w-full max-h-85 mx-auto"}


Illustration from the [Ruby under a microscope](https://patshaughnessy.net/ruby-under-a-microscope) book, chapter 2.

<qr-code url="https://patshaughnessy.net/ruby-under-a-microscope" caption="Ruby under a microscope" class="w-36 absolute bottom-50px right-40px" />


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


<!-- The point is… -->

---
layout: statement
---


# Blocks ARE callbacks

We often use blocks as callbacks to **hook** our own behavior for someone's else code.

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

First part is simple: you put instructions between `do` and `end`, and they become the block body.

Environment is a bit more tricky: as you know,
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

---

## Environment changes: `self`

Most well-known example: `instance_exec`

<div class="grid grid-cols-2 grid-rows-1 gap-4">

```ruby {all|all|6-8|7|none|all|none|none}{at:'1'}
class Conference
  def title
    "RubyConf AU 2024"
  end

  def say(&block)
    puts instance_exec(&block)
  end
end
```

```ruby {all|none|none|none|all|4-8|6|none}{at:'1'}
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
```text {hide|hide|hide|hide|hide|hide|hide|all}{at:'1',class:'!children:text-base'}
undefined local variable or method `conf' for an instance of Conference
```
</template>
</RenderWhen>
---

## Environment changes: local vars

Someone can change variables enclosed in the block's environment…

```ruby {all|none|1-6|8-12}
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

---

## Blocks can be called from other threads

```ruby {1-14|1,5|3,9|1,5,9|4,9,12|1-14}{class:'!children:text-sm'}
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

---

## Different threads

And then try to execute them using concurrent-ruby `Promise` that utilizes thread pools to execute blocks…

```ruby {all|1-4|6-8}
promises = 100.times.flat_map do
  Concurrent::Promise.execute(&work1)
  Concurrent::Promise.execute(&work2)
end

Concurrent::Promise.zip(*promises).value!
#=> Unexpected! (RuntimeError)
# But it also might be okay (chances are low though)
```

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

---

## Example: NATS client

<div class="grid grid-cols-2 gap-4">

<div>

Before:

```ruby {all}{class:'!children:text-sm'}
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

```ruby {all}{class:'!children:text-sm'}
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
---
layout: image
image: /images/nats-pure-thread-pool-pull-request.png
---

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

---

## Where you can find thread pools?

- Puma
- Sidekiq
- ActiveRecord `load_async`
- NATS client
- …and many more

<div class="mt-10 text-xl">

Good thing is that you don't have to care about them most of the time.

**Pro Tip:** In Rails use `ActiveSupport::CurrentAttributes` instead of `Thread.current`!

</div>

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
