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

SCAD is a Ruby to Ruby compiler (so technically it is a transpiler) which outputs code containing only symbols (in particular, it does not use letters and digits in the source). It also includes decompiler (detranspiler?) to reverse code to the original form.

[Here is hello world](https://github.com/radarek/scad/raw/master/examples/helloworld.rb) compiled by SCAD.

## Usage

```shell
$ scad -h
Usage: scad [options]
    -i, --inplace                    Replace file in place
    -c, --compile                    Compile file
    -d, --decompile                  Decompile file
    -a, --automatic                  Automatically detect and compile or decompile
```

## Why does it exist?

I was inspired by [RubyConf 2017: Esoteric, Obfuscated, Artistic Programming in Ruby by Yusuke Endoh](https://youtu.be/6K7EmeptEHo?t=572) presentation, where author has shown a technique to write a code by using only symbols. He presented a very neat trick to call eval method by using just symbols and string literals (which can be converted to symbols too):

```ruby
->(&_){
  _['', 'eval', '<CODE>']
}[&:"#{'send'}"]
```

I spent couple of minutes to understand how this code works.
After that, I thought that it is a fun idea to write a tool which will automatically convert any Ruby code to this syntax.

## Does it work with any code?

The original code has some unexpected features, which breaks any serious Ruby code:

* it changes `self` to the empty string
* it injects additional local variables to the code
* `__FILE__` keyword (yes, [it is the keyword](https://github.com/ruby/ruby/blob/12883f8fa6222324880e2b0f161f8c6d6cf365c7/defs/keywords#L13)) returns `"(eval)"`
* `__dir__` method returns `nil`

To make it work with any Ruby code, I had to fix all of them.
