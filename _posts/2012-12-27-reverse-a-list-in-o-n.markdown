---
layout: post
title: "Reverse a list in O(n)"
date: 2012-12-27 16:53
comments: false
categories:
  - programming
  - algorithms
---

For about a year, I've always had a question I ask someone I'm interviewing for
a programming position. I've asked this around 3 times and got some nice
answers, but I had never implemented it. The question is really simple:

**Given a linked list, how would you reverse it in `O(n)`?**

Here's my solution:

```ruby
# https://gist.github.com/nhocki/4392454

Node = Struct.new(:id, :next)

# Build the list
first = Node.new(1, nil)
nodes = 10.times.map{|x| Node.new(x + 2, nil)}
nodes.each_with_index{|node, index| node.next = nodes[index + 1] }
first.next = nodes.first

def reverse_list(node)
  prox = node.next  # Get the next node on the list.
  node.next = nil   # Make the new tail point to nil (end of the new list).

  last = node       # Save that new tail on a tmp variable (to point to it later)
  node = prox       # Change the `current` node to the `next` one. Move ahead.

  while node          # If I'm not at the end of the list.
    prox = node.next  # Save the next node on the original list.
    node.next = last  # Reverse the list (point back from `current`)
    last = node       # Save the current node to point later.
    node = prox       # Move ahead once.
  end
  last
end

def print_list(node)
  while node
    puts node.id
    node = node.next
  end
end

print_list(first)
puts "\n"
print_list(reverse_list(first))
```

How would you do it? There's a really simple solution using a
[stack](https://gist.github.com/nhocki/4392454#comment-676651), but if you
find another one, I'd love to hear about it on
[Twitter @nhocki](https://twitter.com/nhocki)
