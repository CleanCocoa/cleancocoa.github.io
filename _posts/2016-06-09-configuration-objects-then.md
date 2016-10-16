---
title: Configuration Objects and the Then Microframework
created_at: 2016-06-09 18:14:02 +0200
kind: worklog
tags: [ refactoring, value-object, swift ]
url: http://khanlou.com/2016/06/configuration-in-swift/
comments: on
---

When parameter lists grow or two kinds of parameters seem to go together a lot, it's time use the [extract parameter object refactoring](http://refactoring.com/catalog/introduceParameterObject.html) for greater good -- then you can even specify sensible defaults. 

This is a typical way to pass a set of options to an objects during initialization in a language like Swift. In languages like JavaScript or Ruby, dictionaries of key--value pairs can work just as well. Using dictionaries in Swift for this can be a pain, though.

Now [Soroush wrote about a way](http://khanlou.com/2016/06/configuration-in-swift/) that uses the [Then microframework](https://github.com/devxoul/Then) as a replacement for configuaration dictionaries. This way you don't have to promote every property to the initializer's list of parameters. Here's a before and after, where you can see that without `then` you have to write a lot of repeating boilerplate:
    
    #!swift
    // Before
    
    struct FieldData {
        let title: String
        let placeholder: String
        let keyboardType: UIKeyboardType
        let secureEntry: Bool
        let autocorrectType: UITextAutocorrectionType
        let autocapitalizationType: UITextAutocapitalizationType
    
        init(title: String,
            placeholder: String = "",
            keyboardType: UIKeyboardType = .Default,
            secureEntry: Bool = false,
            autocorrectType: UITextAutocorrectionType = .None,
            autocapitalizationType: UITextAutocapitalizationType = .None)
            {
                self.title = title
                self.placeholder = placeholder
                self.keyboardType = keyboardType
                self.secureEntry = secureEntry
                self.autocorrectType = autocorrectType
                self.autocapitalizationType = autocapitalizationType
        }
    }
    
    let fieldData = FieldData(title: "Password", secureEntry: true)

Now with `then`, making non-mandatory properties mutable and getting rid of the boilerplate:

    #!swift
    // After
    
    struct FieldData {
        let title: String
        var placeholder = ""
        var keyboardType = UIKeyboardType.Default
        var secureEntry = false
        var autocorrectType = UITextAutocorrectionType.No
        var autocapitalizationType = UITextAutocapitalizationType.None
    
        init(title: String) {
            self.title = title
        }
    }
    
    let fieldData = FieldData(title: "Password").then({
        $0.secureEntry = true
    })

That's a Swift alternative to Ruby's hash-based option initializers. There, the dictionary's key is used as the setter's name which is then invoked like so:

    #!ruby
    class Example
      attr_reader :name, :age
      
      def initialize(args)
        args.each do |k,v|
          instance_variable_set("@#{k}", v) unless v.nil?
        end
      end
    end
    
    e1 = Example.new :name => 'foo', :age => 33
    #=> #<Example:0x3f9a1c @name="foo", @age=33>
