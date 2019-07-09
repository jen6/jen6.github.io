---
layout: post
title: "First week of GSOC, Piece Table Implement"
description: ""
category: 
tags: ["GSOC", "KDE", "markdown", "piece table"]
comments: true
thumbnail: http://drive.google.com/uc?export=view&id=1aVmYiLCsi4DSpwYc6LvYyKB7rZF8-Qq7
---
![](http://drive.google.com/uc?export=view&id=1aVmYiLCsi4DSpwYc6LvYyKB7rZF8-Qq7){: width="50%" height="50%"}

Hi! 
Last week was start of the GSOC coding period.  So I started my project.
Also I opened my code on the KDE git.
 [https://cgit.kde.org/scratch/songeon/kmarkdownparser.git/](https://cgit.kde.org/scratch/songeon/kmarkdownparser.git/)

If you are interested in my project feel free to look and give me some advices.

## Parser using Spirit::x3

First, I started to make the markdown parser using the Boost Spirit X3.

Spirit makes easy to express grammar using the PEG.
But it's templet based library so it was hard to find out which part is wrong.
Also documentation of spirit was limited. 

So I had a lots of trial and error to get compilable source code.

[http://becpp.org/blog/wp-content/uploads/2019/02/Ruben-Van-Boxem-Parsing-CSS-in-C-with-Boost-Spirit-X3.pdf](http://becpp.org/blog/wp-content/uploads/2019/02/Ruben-Van-Boxem-Parsing-CSS-in-C-with-Boost-Spirit-X3.pdf)

[https://ciere.com/cppnow15/using_x3.pdf](https://ciere.com/cppnow15/using_x3.pdf)

this two slides was really helpful for me.

## Markdown AST Structure

if we type markdown document like this

    # hello
    > > 1. - **sadf** - 
    > > 
    > > sadf
    > 
    > asdf

 we will get result like this. 

![](http://drive.google.com/uc?export=view&id=1goYc8X9Z1O9qKqGYE442-twWJq_CkSkj)

Markdown document's can be splited in each **lines**. line is the smallest unit of the markdown document. Also line can be expressed **attributes,** **string, emphasizes**.


- attributes : Line's attribute that change the block style. There are two types of attribute. 
- Single Attribute : # header. Style applied only once.
- Multiple Attributes : > BlockQuote, Lists. Sytle applied recursively.
- String : Content of the document. It contains only string.
- Emphasize : Bold, Cancleline, Underline... It contains index of string to be emphasized


All line can be parsed in parallel on multiple threads. Because each line is independent from other lines. After parsing seperatly it can be reassembled after parsing phase.


So we can express this grammar with the Spirit grammar like this
```cpp
    auto const Line_def= 
        Header[pushFunc] >> String[setContent]
        | (MultipleAttribute)* >> EmphasizeString[getEmphStr];
```


## Piece Table

I had a hard time to implementing the Emphasize String.

All String Emphasize token should be paired. If tokens not paired it should be inserted in original points
```
    **this is paired example**
    
    **This token -> -- is not paired**
```
Like this exampled `--` should be inserted in middle of the string as the normal character.

We can simply using insert method but it has performance issue. 

If there is a original string length N and another string length M to be inserted in position P.

First split the original string at position P and shift the splited string as mush as M.

Then put the string length M in position P. It takes almost O(N). 

If there are alot of non-paired tokens, It will consume more time.

---
So I found the VSCode team's article about reimplementing text buffer using the **Piece Table**.

[https://code.visualstudio.com/blogs/2018/03/23/text-buffer-reimplementation](https://code.visualstudio.com/blogs/2018/03/23/text-buffer-reimplementation)



The main idea of the Piece Table is seperate the original buffer and added buffer. 

And split the strings in a small pieces containing position to insert and the string length.  

Make a table about how to make new string using each pieces.

So I applied this structure in appending non-paired emphasize tokens.
```cpp
    const auto EmphasizeString_def = 
      +(omit[Emphasize[setEmp]] | Content[setEmpText]); 
```


In my grammar definition, it do not check the pairness of Emphasize token.

Basically Emphasize token are temporary stored in Emphasize Stack to check pairness.

First If there is token can be paired, add token in Emphasize vector and clean the non-paired tokens. 

Or there are not tokens can be paired just push the token in the Emphasize Stack and push token's string value in the buffer vector.

---

Strings are also added in the buffer vector and inserted in the **Piece Map.**

Piece Map works same as Piece table but effective on find the piece on some position.

Piece Map is multimap that use Piece's index and Piece as key value. 

So we can iterate whole pieces and make new string with non-paired tokens.
```cpp
    namespace AST {
      struct Piece {
        int start = 0, len = 0;
        int bufIdx = 0;
      };
    
      struct Emphasize {
        int start = 0, end = 0, bufIdx = 0;
        EmphasizeType empType = EmphasizeType::DEFAULT;
      };
    
      using PieceMap = std::multimap<int, Piece>;
      using PieceMapIter = PieceMap::iterator;
    
      class EmphasizeString {
        private:
          int currPos = 0, addedTokenLen = 0;
          std::vector<std::string> buffer;
          std::vector<Emphasize>  empSt, emps;
          PieceMap pieceMap;
    
        private:
          int clearNonComplete(std::vector<Emphasize>::iterator );
        public:
          EmphasizeString();
          void addEmphToken(EmphasizeType empt);
          void appendText(const std::string& text);
          std::string getString();
          const std::vector<Emphasize> &getEmphasizes() const;
      };
    };
```
You can check my whole code on KDE git.

---

## After first week

I think it was good starting for me.  But this week and next week is my final exam in the school.

So me and my mentors Eike and SungJae planned to refactor the code and make view of markdown.

They also mentioned adding the licensing policy and following the KDE's codding convention.

Thank you for reading my article and I'll write next article after my final finish.
