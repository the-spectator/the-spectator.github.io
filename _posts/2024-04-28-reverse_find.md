---
layout: post
title: "Reverse Find in Ruby"
date: 2024-04-28 16:15:06 +0530
categories: ruby
---

One of the things that sets Ruby apart from other programming languages is its impressive set of built-in features.
One of my favourite method is [`Enumerable#detect`](https://ruby-doc.org/3.3.0/Enumerable.html#method-i-detect).
It returns the first element for which the block yields a truthy value and stops searching upon the first hit.

```ruby
require "prime"
arr = (1..10).to_a
arr.detect { |n| Prime.prime?(n) } #=> 2
```

Okay, let's spice it up! How do we find last matching element? A possible solution could be to filter the whole array using [`Enumerable#filter`](https://ruby-doc.org/3.3.0/Enumerable.html#method-i-select), and use the last match.

```ruby
require "prime"
arr = (1..10).to_a
arr.filter { |n| Prime.prime?(n) }.last #=> 7
```

However, this approach means traversing the entire array just to get that final match and discarding rest of the matches.

<h3>Ruby to Rescue</h3>

Ruby provides another awesome method [`Enumerable#reverse_each`](https://ruby-doc.org/3.3.0/Enumerable.html#method-i-reverse_each), it allows to gracefully traverse an array in reverse order without mutating the original array.

Combining it with `Array#detect`, we now have an optimized solution where we can find the last matching element without traversing the full array, making it doubly awesome.

```ruby
require "prime"
arr = (1..10).to_a
arr.reverse_each.detect { |n| Prime.prime?(n) }
```

<h3>Wait! Ruby is actually more awesome!!</h3>

Ruby provides a method that exactly matches our use case: [`Array#rindex`](https://ruby-doc.org/3.3.0/Array.html#method-i-rindex).
This method returns the index of the last element for which the block returns a truthy value.

While similar to `Enumerable#detect`, there is a slight difference between the two: `Enumerable#detect` returns the matched element itself, whereas `Array#rindex` returns the index of that element.

```ruby
require "prime"
arr = (1..10).to_a
index = arr.rindex { |n| Prime.prime?(n) }
arr[index] #=> 7
```

<h3>Benchmark It!!</h3>

```ruby
# frozen_string_literal: true

require "bundler/inline"

gemfile true do
  source "https://rubygems.org"

  gem "benchmark-ips"
  gem "prime"
end

require "benchmark/ips"
require "prime"

ARR = (1..100_000).to_a

Benchmark.ips do |x|
  x.report("reverse_each.detect") { ARR.reverse_each.detect { |n| Prime.prime?(n) } }
  x.report("filter.last") { ARR.filter { |n| Prime.prime?(n) }.last }
  x.report("rindex") { ARR[ARR.rindex { |n| Prime.prime?(n) }] }

  x.compare!
end

# ruby 3.3.0 (2023-12-25 revision 5124f9ac75) +YJIT [arm64-darwin22]
# Warming up --------------------------------------
#  reverse_each.detect     7.306k i/100ms
#          filter.last     1.000 i/100ms
#               rindex     7.623k i/100ms
# Calculating -------------------------------------
#  reverse_each.detect     73.400k (± 1.2%) i/s -    372.606k in   5.077127s
#          filter.last      8.764 (± 0.0%) i/s -     44.000 in   5.025497s
#               rindex     76.693k (± 1.0%) i/s -    388.773k in   5.069692s

# Comparison:
#               rindex:    76693.4 i/s
#  reverse_each.detect:    73400.3 i/s - 1.04x  slower
#          filter.last:        8.8 i/s - 8750.82x  slower
```

The results are in, and it looks like `Array#rindex` is the clear winner! While `reverse_each.detect` puts up a valiant effort, it falls just short of the mark. And as for `filter.last`, well, let's just say it's taking the scenic route... at a snail's pace.
