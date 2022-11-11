---
title: Put subroutine into the library
description: How to write library subroutines
layout: post
post-url: macro-hosted-subroutines
categories: cbm asm
---
Subroutine is an essential concept in programming. Advantages of using subroutines are quite clear - instead of doing a repetitive work, we define chunk of code once and "jump" to it any time we need it. It is a part of DRY principle (do not repeat yourself), which allows to keep our code compact and maintainable. It allows us to keep memory footprint of our code at reasonable level. With possibility of passing execution arguments as well as result, subroutines become fully blown procedures and functions as known from high level languages.

How does MOS 6502 support subroutines? We essentially need two instructions: "**J**ump to **S**ub**R**outine" that does typical long jump to given memory address and "**R**e**T**urn from **S**ubroutine" that returns to the address pointing to the next instruction after "jump". These instructions are named `JSR` and `RTS`, respectively. `JSR` does actually more than simple `JMP` - it pushes return address to the stack (MOS 6502 uses memory page 1 for stack: addresses from `$100` to `$1FF`). `RTS` pulls return address from stack and stores it as program counter (PC).

For all of you my readers who like to be very accurate, the address stored by `JSR` is decremented by 1, `RTS` does increment it back by 1. I don't even want to know why, I bet somebody wanted to make some savings on transistor count, it's just my quess...

Communicating with subroutines - parameters
-------------------------------------------
In 6502 machine code there are at least three commonly used methods for passing parameters to the subroutine:
1.   Using processor registers.
2.   Using fixed memory locations (via global variables).
3.   Using stack.

Using CPU registers is the simplest possible way of communication. Assuming that registers are used wisely, we may get CPU registers already properly loaded with values which speeds up execution and can be essential in some time critical cases. There is however one downside of this method directly connected with very simplistic design of 6502: there are only 3 registers that can be easily used (`A`, `X` and `Y`) each only 1 byte in size - not much indeed.

Fixed memory location for exchanging parameters is fine unless you write libraries of code. With library code we cannot assume that certain memory locations can be used by our subroutines, because there is no way to predict what would be memory layout of the client program.

Finally there is a stack, that is 256 bytes of memory located in already mentioned page 1. Stack is a data structure with very limited API, that is we can push value there or pull value from there, there is a stack pointer that indicates where new value will be pushed or pulled that is incremented or decremented automatically with `PLA`/`PHA` instruction executions. We can put our parameters on the stack right before `JSR` is executed and then use it inside of our subroutine.

In next chapters I will describe latter two methods as the most promising for our purposes. Communication with CPU registers as the most straightforward method can also be used, if only amount of data needed in communication is limited.

Hosted subroutines
------------------
As it has been already mentioned in my [library post][1], libraries in KickAssembler can be realized with `#import` macro by importing source file of library into client code. Anything that is not a macro, label or function, which means any 'free floating' pieces of code or data prepended with memory directives will get its own memory segment and will be assembled together with client code. There is no special control over this process, at least I don't know about any. That is a problem because any piece of code will immediately steal some memory during assembling no matter you're going to use it or not. This will lead us to the design, where each subroutine has its own source file which does not seem to be reasonable nor convenient.

But wait, macros are KickAssembler entities that are not automatically resolved into a code! They must be called first. They can be called many times and this is something, we frequently do with short macros. With long macros (and subroutine macros can be potentially very lenghty) it might be no so good idea because we will waste so much memory.

I have very quickly concluded, that we should use only macros in KickAssembler library code and not free floating code at all. There is only one inconvenience, that we have to call such a macro at least one in library client code to "install" our subroutine.

KickAssembler macros can have parameters. One may think this is a perfect way to communicate with our subroutines. But this is false thinking - macro parameters can be used to communicate with macro, not with hosted subroutine.

Let's consider following example:

    .macro add8(left, right) {
        lda #left
        adc #right
    }

This macro, when used loads accumulator with value `left`, adds value `right` to it and leaves result in accumulator. One may think "cool, that's useful". But in reality it is not. We get "subroutine" that takes two parameters, but values of these parameters must be calculated during assembling process. No way to pass accumulator values, or memory values or anything calculated. All we can do is to:

    add8(2, 2)

and get ``4`` in accumulator.

Make global variable communication useful
-----------------------------------------
As it was already mentioned, biggest challenge with communicating using memory location is that this location must be somehow fixed. This theoretically disqualifies such method for librarian subroutines. But what if such memory location is made configurable?

Let's consider our trivial example once more, this time we will write macro based subroutine that takes values from two memory locations, adds them together and leaves result in accumulator:

    .macro add8Mem(leftPtr, rightPtr) {
        lda leftPtr
        adc rightPtr
        rts
    }

Assuming that we have encapsulated this macro in library, namely ``math.asm``, we can now write following "client" program:

    #include "math.asm"

    *=$0801 "Basic upstart"
    BasicUpstart(start)

    *=$3000 "Program"
    start:
        lda #2      // take 2...
        sta $02     // and store it as first argument
        lda #4      // take 4...
        sta $03     // and store it as second argument
        jsr add8Mem // call subroutine that will adds values from $02 and $03
        sta $04     // result from A can be then stored anywhere, i.e.: under $04

        ...         // and the program continues...

    add8Mem:        // under this address we install our subroutine
        add8Mem($02, $03) // subroutine is configured to use $02 and $03 as
                          // parameter placeholders

Here we did several things:
- We have installed our subroutine using library macro under address of choice (here indicated by ``add8Mem`` address (I'm going to use label name that follows macro name, but it does not need to go like this).
- We have configured this subroutine to use addresses of choice for parameter placeholders (we did it using macro parameters).
- We can use subroutine with simple ``JSR``, all we need to do is to load placeholders with data first.

Subroutine, once installed, can be called many times, from different places of code - it will work just fine.

As you can see, with this approach we can use macros as hosts for our subroutines and additionally we have ability to freely configure placeholders.

Using stack
-----------
Stack in MOS 6502 is just a regular place in memory. CPU has three additional components that handles stack in convenient and fast way: it is stack pointer register (SP) and families of push and pull instructions that can transfer data from and to CPU registers. When we use stack for subroutine parameters, it is particularily convenient, because we don't need to care which exact memory cells should we use for this purpose. Just free place on stack will be used. Moreover, this method of communication supports nesting of subroutine calls, and with particularily clever design it may even handle recursive calls.

Lets consider ``add8`` subroutine hosted by a macro (similar to ``add8Mem`` from above) but using stack as communication method. First we have to deal with return address. It is very important to preserve it, because without correctly defined return address our CPU will get jammed as soon as ``RTS`` is called.

Return address is pushed to the stack once ``JSR`` is called and taken back when ``RTS`` is called. We are going to push our parameters to the stack just before ``JSR``, so at the beginning of our subroutine we must pull this address and store somewhere else before accessing our parameters.

    .macro add8() {
        // pull return address and store in placeholder
        pla; sta returnPtr
        pla; sta returnPtr + 1

        // pull second argument
        pla; sta rightPtr + 1
        // pull first argument
        pla; sta leftPtr + 1

      leftPtr:
        lda #$00
      rightPtr:
        adc #$00
        sta resultPtr

        // restore return address
        lda returnPtr + 1; pha
        lda returnPtr; pha
        // move result to A
        lda resultPtr
        rts
      returnPtr: .word 0
      resultPtr: .byte 0
    }

Our program that utilizes following hosted subroutine can look like this:

    #include "math.asm"

    *=$0801 "Basic upstart"
    BasicUpstart(start)

    *=$3000 "Program"
    start:
        lda #2      // take 2...
        pha         // and store it as first argument
        lda #4      // take 4...
        pha         // and store it as second argument
        jsr add8    // call subroutine that will adds values that are stored on the stack
        sta $04

        ...         // and the program continues...

    add8:           // under this address we install our subroutine
        add8()

It is actually plenty of coding. We can simplify this with bunch of KickAssembler macros. This time we will utilize macros as they should be utilized - to replace short, repetitive pieces of code.

    .macro @invokeStackBegin(placeholderPtr) {
        pla
        sta placeholderPtr
        pla
        sta placeholderPtr + 1
    }
    
    .macro @invokeStackEnd(placeholderPtr) {
        lda placeholderPtr + 1
        pha
        lda placeholderPtr
        pha
    }
    
    .macro @pushParamB(value) {
        lda #value
        pha
    }

    .macro @pullParamB(placeholderPtr) {
        pla
        sta placeholderPtr
    }

With these macros we can code our macro hosted subroutine in simpler manner (please appreciate the beauty of this self modifying code):

    .macro add8() {
        invokeStackBegin(returnPtr)
        pullParamB(rightPtr + 1)
        pullParamB(leftPtr + 1)

      leftPtr:
        lda #$00
      rightPtr:
        adc #$00
        sta resultPtr

        invokeStackEnd(returnPtr)
        lda resultPtr
        rts
      returnPtr: .word 0
      resultPtr: .byte 0
    }

and calling code can look like this:

    start:
        pushParamB(2)
        pushParamB(4)
        jsr add8    // call subroutine that will adds values that are stored on the stack
        sta $04

        ...         // and the program continues...

    add8:           // under this address we install our subroutine
        add8()

Helper macros for implementing stack based subroutines are available as KickAssembler library as a part of c64lib. You must include ``invoke.asm`` from common repository (https://github.com/c64lib/common).

Summary
-------
We just saw how subroutines can be effectively implemented as part of libraries. With this approach they can be easily installed into target code and reused by many programs. We've learnt also how can we pass parameters to the subroutines. With a little help from self modifying code and some macros from c64lib common library we can write compact code that communicates via stack. I will post some more complex examples in subsequent posts, including indirect subroutine parameters.

## References
* \[1\] [assembler-libraries][1]

[1]: assembler-libraries
