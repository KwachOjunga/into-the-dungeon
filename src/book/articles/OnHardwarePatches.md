# On Hardware Patches.
___
```
                                                                            08/03/2026
```

Earlier this week, someone asked me about how they fix hardware vulnerabilities like spectre (the first and second variant)
and meltdown.

And while i do not recall my response, i am under the impression that my answer was close to this; "Well, that can't be fixed 
its hardware."

My then prevailing assumptions were:- 
  - Hardware once described in RTL is final; that is signals move from point A to B and are not bidirectional,
      hence to fix a vulnerability, you must change the RTL.[^1]
  - Assembly language binary encodings are fixed and final; therefore they are the only way 
    to effect a function whose output affects the observable execution environment.

The truth, it's slightly more involving than that black and white declaration:
  "...it can't be fixed in software."
  - By this i mean any mitigations will have to be released in hardware updates.

In the event of a hardware vulnerablity, where the loophole exists in 
the behavior of actual functional components made up of circuits, it is 
true that the "real" solution will have to be a physical implementation.

Alternatively, certain mitigations can be offeered logically. This is where compilers and operating systems
come in. The OS' mitigation is to be found in policing the set of programs it is asked to execute i.e the memory locations 
it is accessing, the signals it is sending among a whole slew of other techniques i am stil unaware of.

As for the compiler, it polices the types of operations it is allowed to perform on the program's data during compilation.
So the solutions come in three forms, all being updates/patches.
In software, hardware and microcode.

Yes, there is such a thing as microcode.


[^1]: RTL - [Register Transfer Language](https://grokipedia.com/page/Register-transfer_level)
