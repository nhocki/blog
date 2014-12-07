---
layout: post
title: "Tower of Hanoi with a stack"
date: 2013-02-05 14:37
comments: false
excerpt: How to implement the Tower of Hanoi problem without recursion using a stack.
categories:
  - algorithms
  - programming
  - interview tips
---

I've recently subscribed to [Coding for Interviews](http://codingforinterviews.com/)
and received my first email today. The content is great and you should join.
Even if you're not looking for a job, it's a short read with nice topics to
refresh your memory.

[Today's assignment][2] was with stacks. I've used stacks to
[reverse a list in O(n)](http://blog.nhocki.com/2012/12/27/reverse-a-list-in-o-n/)
but today's challenge was to solve the [Tower of Hanoi][1] problem without
recursion.

Here's my solution:

```ruby
# https://gist.github.com/nhocki/4716874

Hanoi = Struct.new(:disk, :source, :dest, :spare)

def move(hanoi)
  "Move #{hanoi.disk.abs} from #{hanoi.source} to #{hanoi.dest}"
end

class Stack
  def initialize
    @data = []
  end

  def push(hanoi)
    @data.unshift(hanoi)
  end

  def pop
    @data.shift
  end

  def peek
    @data.first
  end

  def empty?
    @data.empty?
  end
end


# Resolve the Tower of Hanoi problem with a Stack. You can't use recursion
# in it's traditional way, but adding everything to a stack works the same.
#
# Notice you have to invert the order of the calls to the recursive function
# since the stack is **Last In - First Out**.
def stacker hanoi
  stack = Stack.new
  stack.push(hanoi)

  until stack.empty? do
    hanoi = stack.pop

    if hanoi.disk <= 1
      puts move(hanoi)
    else
      left  = Hanoi.new(hanoi.disk - 1, hanoi.source, hanoi.spare, hanoi.dest)
      right = Hanoi.new(hanoi.disk - 1, hanoi.spare, hanoi.dest, hanoi.source)
      hanoi.disk = -hanoi.disk # force printing it next time it appears
      stack.push right
      stack.push hanoi
      stack.push left
    end
  end
end


# Resolve the Tower of Hanoi problem with recursion.
def recursive hanoi
  if hanoi.disk == 1
    puts move(hanoi)
  else
    left  = Hanoi.new(hanoi.disk - 1, hanoi.source, hanoi.spare, hanoi.dest)
    right = Hanoi.new(hanoi.disk - 1, hanoi.spare, hanoi.dest, hanoi.source)
    recursive(left)
    puts move(hanoi)
    recursive(right)
  end
end

h = Hanoi.new(3, 'A', 'B', 'C')

recursive(h)
puts "\n\n"
stacker(h)
```

There's another iterative solution but I haven't implemented it. So, if you did
or have some other way to solve this, I'd love to know on
[Twitter @nhocki](https://twitter.com/nhocki)

[1]:http://en.wikipedia.org/wiki/Tower_of_Hanoi
[2]:http://us2.campaign-archive2.com/?u=cadc6c448cd083a0aeed7f864&id=c550bc59d5&e=5694567579
