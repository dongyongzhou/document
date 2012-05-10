---
layout: master
title:  conditional compile
---

##1 Overview

The #if directive, with the #elif, #else, and #endif directives, controls compilation of portions of a source file. 

If the expression you write (after the #if) has a nonzero value, the line group immediately following the #if directive is retained in the translation unit.

##2 Grammar

    #if constant-expression
    #ifdef identifier
    #ifndef identifier
    
    #elif constant-expression
    
    #else
    
    #endif

- Each #if directive in a source file must be matched by a closing #endif directive. 
- Any number of #elif directives can appear between the #if and #endif directives.
- at most one #else directive is allowed. The #else directive, if present, must be the last directive before #endif.

- The #if, #elif, #else, and #endif directives can nest in the text portions of other #if directives. Each nested #else, #elif, or #endif directive belongs to the closest preceding #if directive.

- All conditional-compilation directives, such as #if and #ifdef, must be matched with closing #endif directives prior to the end of file; 


###2.1 constant-expression

The constant-expression is an integer constant expression with these additional restrictions:

- Expressions must have integral type and can include only integer constants, character constants, and the defined operator.
- The expression cannot use sizeof or a type-cast operator.
- The target environment may not be able to represent all ranges of integers.
- The translation represents type int the same as type long, and unsigned int the same as unsigned long.
- The translator can translate character constants to a set of code values different from the set for the target environment. To determine the properties of the target environment, check values of macros from LIMITS.H in an application built for the target environment.
- The expression must not perform any environmental inquiries and must remain insulated from implementation details on the target computer.

###2.2 defined

The preprocessor operator **defined** can be used in special constant expressions, as shown by the following syntax:

    defined( identifier )
    defined identifier

This constant expression is considered true (nonzero) if the identifier is currently defined; otherwise, the condition is false (0). An identifier defined as empty text is considered defined. The defined directive can be used in an #if and an #elif directive, but nowhere else.

**defined** The special operator defined is used in #if and #elif expressions to test whether a certain name is defined as a macro. **defined name** and **defined (name)** are both expressions whose value is 1 if name is defined as a macro at the current point in the program, and 0 otherwise. 

Thus, #if defined MACRO is precisely equivalent to #ifdef MACRO.

**defined** is useful when you wish to test more than one macro for existence at once. For example,

#if defined (MACRO1) || defined (MACRO2)

would succeed if either of the names MACRO1 or MACRO2 is defined as a macro.

##Reference

http://technet.microsoft.com/zh-cn/ew2hz0yd(it-it,VS.90).aspx
