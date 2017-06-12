# From C to Ruby through VALUE

When I test stuff in IRB, I always see this weird hex 'address like' value next to the class name:

```ruby
2.4.0 :001 > class ViktorsIntellect; end
 => nil
2.4.0 :002 > ViktorsIntellect.new
 => #<ViktorsIntellect:0x007fe2259a7d10>
```

So I started wondering where this address(0x007fe2259a7d10) comes from. That process took some time, because my intellect is as high as the method count in the class above. It looks like it's the address of the instance and due to the fact Matz' Ruby is written in C - probably it's something related to pointers.

In Ruby everything is an object and every variable is a reference to an object. Also it's a dynamic typed language. On the contrary, C is a static typed language and object oriented stuff is not so much in the game there. So how are the two languages able to 'communicate' with each other?

To represent Ruby objects in C, the type VALUE is used. If you have a new school machine it will be alias for  **uintptr\_t** and it's defined [here](https://github.com/ruby/ruby/blob/ruby_2_4/include/ruby/ruby.h#L79).

```c
typedef uintptr_t VALUE;
```

The type **uintptr\_t** is added with C99 and it should be big enough to hold any pointer. Although it is an unsigned integer type, the name uint**ptr**\_t shows the intention of using it as pointer store. Another cool thing about it - it's portable.

At this point I started suspecting that the address above is the value of the VALUE variable that points to the ViktorsIntellect object, but wanted to prove it.

So let's build the same class in C, compile it and include it in IRB.

```c
#include "ruby.h"

static VALUE
t_show_yourself(VALUE self)
{
    printf("%lx\n", self); //printing the self variable which is VALUE type
    return self;
}

VALUE cViktorsIntellect;

void
Init_ViktorsIntellect() {
    cViktorsIntellect = rb_define_class("ViktorsIntellect", rb_cObject);
    rb_define_method(cViktorsIntellect, "show_yourself", t_show_yourself);
}
```

The equivalent of this code in Ruby (without the printf part) would be:

```ruby
class ViktorsIntellect
  def show_yourself
    self
  end
end
```

So if we compile the C code ([how to do that?](https://en.wikibooks.org/wiki/Ruby_Programming/C_Extensions)) and require it in IRB we get the following:

```ruby
2.4.0 :001 > require './ViktorsIntellect'
 => true
2.4.0 :002 > ViktorsIntellect.new.show_yourself
101b5fe30
 => #<ViktorsIntellect:0x00000101b5fe30>
```

And that was [the wire moment](https://www.youtube.com/watch?v=BOZv6IOtbFU).
