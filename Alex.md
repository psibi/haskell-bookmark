# Introduction

* Alex is a tool for generating lexical analysers in Haskell.
* Alex takes descriptions of tokens based on regular expressions and
  generates a Haskell module for scanning text efficiently.

## Sample code

``` lex
{
module Main (main) where
}

%wrapper "basic"

$digit = 0-9      -- digits
$alpha = [a-zA-Z]   -- alphabetic characters

tokens :-

  $white+       ;
  "--".*        ;
  let         { \s -> Let }
  in          { \s -> In }
  $digit+       { \s -> Int (read s) }
  [\=\+\-\*\/\(\)]      { \s -> Sym (head s) }
  $alpha [$alpha $digit \_ \’]* { \s -> Var s }

{
-- Each action has type :: String -> Token
-- The token type:
data Token =
     Let     |
     In      |
     Sym Char  |
     Var String  |
     Int Int
     deriving (Eq,Show)

main :: IO ()
main = do
   s <- getContents
   print (alexScanTokens s)
}
```

* The lines between the `{` and `}` provide a code scrap (inlined
  Haskell code) to be placed directly in the output.
* The line `%wrapper basic` controls what kind of support code Alex
  should produce along with basic scanner. The `basic` wrapper
  tokenises a `String` and returns a list of tokens.
* The line with `$digit` and `$alpha` defines macros for use in token
  defintions.
* The `tokens :-` line ends the macro definition and starts the
  definition of the scanner.

Creating a module:

```
$ stack install alex happy
$ alex sample1.x -o sample1.hs
```

## Experimenting with the module


```shellsession
$ stack ghci sample1.hs
λ> :t alexScanTokens
alexScanTokens :: String -> [Token]
λ> alexScanTokens "hi 3 kfaj 2"
[Var "hi",Int 3,Var "kfaj",Int 2]
λ> alexScanTokens "hi 3 let x = 5 in y"
[Var "hi",Int 3,Let,Var "x",Sym '=',Int 5,In,Var "y"]
```
