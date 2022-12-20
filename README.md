```ruby
   _=$$/$$;$>.<<''         <<(-~_*(-~_*(-~               _*(            -~_*(-~_*(-~_*(-+~
_*_               [_]   +_)               )+_         )))   +_)         +_)               <<(
-~_                     *(-                        ~_*         (-~      _*(               -+~
   _*(-~_*(-~_*(-~      _*_                     [_]               +_)   )))               )++
                  _)+   +_)                     <<(-~_*(-~_*(-~_*(-~_   *(-               ~_*
(-~               _*(   -~_               *_[   _]+               _))   )))               )++
   _)<<(-~_*(-~_*(         -~_*(-~_*(-~_*(      -~_               *(-   ~_*_[_]+_))))+_)))
```

# SCAD - Symbolic Compiler and Decompiler

SCAD is a Ruby to Ruby compiler (so technically it is a transpiler) which outputs code containing only symbols (in particular, it does not use letters and digits in the source). Code is formatted so it's shape looks like the original text. It also includes decompiler (detranspiler?) to reverse code to the original form.

[Here is hello world](https://github.com/radarek/scad/raw/master/examples/helloworld.rb) compiled by SCAD.

## Usage

```shell
Usage: scad [options] [file ...]
    -i, --inplace                    Replace file in place (back up file first to be safe!)
    -c, --compile                    Compile file
    -d, --decompile                  Decompile file
    -a, --automatic                  Detect automatically and compile or decompile file
    -f, --font=FONT                  Use specific font when formatting compiled code (default: font4.txt). See fonts/ directory.
    -v, --version                    Show version
    -h, --help                       Show this message
```

## Why does it exist?

I was inspired by [RubyConf 2017: Esoteric, Obfuscated, Artistic Programming in Ruby by Yusuke Endoh](https://youtu.be/6K7EmeptEHo?t=572) presentation, where author has shown a technique to write a code by using only symbols. He presented a very neat trick to call eval method by using just symbols and string literals (which can be converted to symbols too):

```ruby
->(&_){
  _['', 'eval', '<CODE>']
}[&:"#{'send'}"]
```

I spent couple of minutes to understand how this code works.
After that, I thought that it will be a fun idea to write a tool which automatically converts any Ruby code to this syntax.

## Does it work with any code?

The original code has some unexpected features, which breaks any serious Ruby code:

* it changes `self` to the empty string
* it injects additional local variables to the code
* `__FILE__` keyword (yes, [it is the keyword](https://github.com/ruby/ruby/blob/12883f8fa6222324880e2b0f161f8c6d6cf365c7/defs/keywords#L13)) returns `"(eval)"`
* `__dir__` method returns `nil`

To make it work with any Ruby code, I had to fix all of them.

To fix `self` reference, we just need to pass somehow `self` reference instead of empty string. We can't however do this directly - we would break the rule of not using letters and digits. The trick is to pass an additional parameter to our lambda. This parameter is yet another empty lambda `->{}`. Every lambda remembers its environment it was created in. It also includes `self`. It can be taken through `lambda.binding.receiver`. Of course, we still can't call it directly (remember the rule?) but now we can use the same trick with "eval-through-lambda" to evaluate this code. In our code, it will look like this:

```ruby
->(_, &__){
  __[
     __['', 'eval', '_.binding.receiver'],
     'eval',
     'p "Hello World!", self, local_variables, __FILE__, __dir__'
   ]
}[->{}, &:"#{'send'}"]
```

Next thing to fix are local variables. When we call `eval` there are some already defined local variables. Specifically `_` and `__`. They are accessible in the evaled code. It is not a big deal, but we want to get rid of them. To achieve this, we have to specify an additional argument - the binding. We have to create somehow empty binding. Unfortunately, Ruby does not allow creating it manually. It's not a problem. We can get it... from our new lambda (using `binding` method). Because it was created in outer scope and there are no local variables there, then it will be empty. We will use "eval-through-lambda" one more time:

```ruby
->(_, &__){
  __[
     __['', 'eval', '_.binding.receiver'],
     'eval',
     'p "Hello World!", self, local_variables, __FILE__, __dir__',
     __['', 'eval', '_.binding'],
   ]
}[->{}, &:"#{'send'}"]
```

We are left with two last problems. `__FILE__` and `__dir__` returns `"(eval)"` in the code which is evaled. To fix this, we need to pass an additional argument to the `eval` so it will treat the code as it is run from the file. In outer scope we could pass original `__FILE__` to our lambda but... yes, the rule. As you probably already noticed, we use lambda for every problem we have, and it's not different this time. We can get source location by using `lambada.source_location[0]`:

```ruby
->(_, &__){
  __[
     __['', 'eval', '_.binding.receiver'],
     'eval',
     'p "Hello World!", self, local_variables, __FILE__, __dir__',
     __['', 'eval', '_.binding'],
     __['', 'eval', '_.source_location[0]'],
   ]
}[->{}, &:"#{'send'}"]
```

The final code has some more additional tricks, so code can be freely formatted. You can discover it on your own. Good luck!

## Why the source code of this project looks weird?

Because it was compiled by a SCAD itself. If you want to see original source, just decompile it with:

```
scad -di lib/scad/compiler.rb
```
