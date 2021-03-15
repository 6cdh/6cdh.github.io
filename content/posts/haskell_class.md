---
title: Hierarchy of Haskell
date: 2021-03-14T22:02:49+08:00
tags: [haskell]
---

The standard Haskell libraries has a number of type classes. To be a expert of Haskell
requires us to understand each type class and its relationship to other type classes.

Here I made a list of typeclasses and their hierarchy structure.

<!--more-->

## Typeclasses

```haskell
-- The 'Eq' class defines equality ('==') and inequality ('/=').
-- All the basic datatypes exported by the "Prelude" are instances of 'Eq',
-- and 'Eq' may be derived for any datatype whose constituents are also
-- instances of 'Eq'.
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool

-- The 'Ord' class is used for totally ordered datatypes.
class Eq a => Ord a where
  compare :: a -> a -> Ordering
  (<) :: a -> a -> Bool
  (<=) :: a -> a -> Bool
  (>) :: a -> a -> Bool
  (>=) :: a -> a -> Bool
  max :: a -> a -> a
  min :: a -> a -> a

-- The `Num` class is basic numeric class.
class Num a where
  (+) :: a -> a -> a
  (-) :: a -> a -> a
  (*) :: a -> a -> a
  negate :: a -> a   -- Unary negation.
  abs :: a -> a      -- Absolute value.
  signum :: a -> a   -- Sign of a number. For real numbers, the 'signum' is
                     -- either -1 (negative), 0 (zero) or 1 (positive).
  fromInteger :: Integer -> a -- Conversion from an 'Integer'.

-- The `Show` is a specialised variant of 'showsPrec', using precedence context
-- zero, and returning an ordinary 'String'.
class Show a where
  ...
  show :: a -> String
  ...

-- The `Read` class parsing of 'String's, producing values.
class Read a where
  readsPrec :: Int -> ReadS a -- attempts to parse a value from the front of
                              -- the string, returning a list of (parsed
                              -- value, remaining string) pairs.  If there is no
                              -- successful parse, the returned list is empty.
  readList :: ReadS [a] -- The method 'readList' is provided to allow the
                        -- programmer to give a specialised way of parsing
                        -- lists of values.

-- The 'Bounded' class is used to name the upper and lower limits of a
-- type.  'Ord' is not a superclass of 'Bounded' since types that are not
-- totally ordered may also have upper and lower bounds.
class Bounded a where
  minBound :: a
  maxBound :: a

-- Class 'Enum' defines operations on sequentially ordered types.
class Enum a where
  succ :: a -> a -- the successor of a value. For numeric types,
                 -- 'succ' adds 1.
  pred :: a -> a -- the predecessor of a value. For numeric types,
                 -- 'pred' subtracts 1.
  toEnum :: Int -> a   -- Convert from an 'Int'.
  fromEnum :: a -> Int -- Convert to an 'Int'.
  enumFrom :: a -> [a] -- Used in Haskell's translation of [n..] with [n..] = enumFrom n
  enumFromThen :: a -> a -> [a] -- Used in Haskell's translation of [n,n'..]
                                -- with [n,n'..] = enumFromThen n n'
  enumFromTo :: a -> a -> [a] -- Used in Haskell's translation of [n..m] with
                              -- [n..m] = enumFromTo n m
  enumFromThenTo :: a -> a -> a -> [a] -- Used in Haskell's translation of [n,n'..m] with
                                       -- [n,n'..m] = enumFromThenTo n n' m

class (Num a, Ord a) => Real a where
  toRational :: a -> Rational -- the rational equivalent of its real argument
                              -- with full precision

-- Arbitrary-precision rational numbers, represented as a ratio of
-- two 'Integer' values.  A rational number may be constructed using
-- the '%' operator.
type Rational = GHC.Real.Ratio Integer

-- Fractional numbers, supporting real division.
class Num a => Fractional a where
  (/) :: a -> a -> a -- Fractional division.
  recip :: a -> a    -- Reciprocal fraction.
  fromRational :: Rational -> a -- Conversion from a 'Rational' (that is 'Ratio' 'Integer').

-- Integral numbers, supporting integer division.
class (Real a, Enum a) => Integral a where
  quot :: a -> a -> a -- integer division truncated toward zero
  rem :: a -> a -> a  -- integer remainder, satisfying (x `quot` y)*y + (x `rem` y) == x
  div :: a -> a -> a  -- integer division truncated toward negative infinity
  mod :: a -> a -> a  -- integer modulus, satisfying (x `div` y)*y + (x `mod` y) == x
  quotRem :: a -> a -> (a, a) -- simultaneous 'quot' and 'rem'
  divMod :: a -> a -> (a, a)  -- simultaneous 'div' and 'mod'
  toInteger :: a -> Integer   -- conversion to 'Integer'

-- Extracting components of fractions.
class (Real a, Fractional a) => RealFrac a where
  properFraction :: Integral b => a -> (b, a) -- takes a real fractional number x
                                              -- and returns a pair (n,f) such
                                              -- that x = n+f, and n is an integral
                                              -- number with the same sign as x, f
                                              -- is a fraction with the same type
                                              -- and sign as x, and with absolute
                                              -- value less than 1.
  truncate :: Integral b => a -> b -- returns the integer nearest a between zero and a
  round :: Integral b => a -> b -- returns the nearest integer to a;
                                -- the even integer if a is equidistant
                                -- between two integers
  ceiling :: Integral b => a -> b -- returns the least integer not less than a
  floor :: Integral b => a -> b -- returns the greatest integer not greater than a

-- 'Floating' defines trigonometric and hyperbolic functions and related functions.
class Fractional a => Floating a where
  pi :: a
  exp :: a -> a
  log :: a -> a
  sqrt :: a -> a
  (**) :: a -> a -> a
  logBase :: a -> a -> a
  sin :: a -> a
  cos :: a -> a
  tan :: a -> a
  asin :: a -> a
  acos :: a -> a
  atan :: a -> a
  sinh :: a -> a
  cosh :: a -> a
  tanh :: a -> a
  asinh :: a -> a
  acosh :: a -> a
  atanh :: a -> a
  GHC.Float.log1p :: a -> a
  GHC.Float.expm1 :: a -> a
  GHC.Float.log1pexp :: a -> a
  GHC.Float.log1mexp :: a -> a

-- Efficient, machine-independent access to the components of a
-- floating-point number.
class (RealFrac a, Floating a) => RealFloat a where
  floatRadix :: a -> Integer -- a constant function, returning the radix of the representation
                             --  (often 2)
  floatDigits :: a -> Int    -- a constant function, returning the number of digits of
                             -- 'floatRadix' in the significand
  floatRange :: a -> (Int, Int) -- a constant function, returning the lowest and highest values
                                -- the exponent may assume
  decodeFloat :: a -> (Integer, Int) -- returns the significand expressed as an 'Integer' and an
                                     -- appropriately scaled exponent (an 'Int').
  encodeFloat :: Integer -> Int -> a -- performs the inverse of 'decodeFloat' in the
                                     -- sense that for finite x with the exception of -0.0
  exponent :: a -> Int -- the second component of 'decodeFloat'
  significand :: a -> a -- the first component of 'decodeFloat'
  scaleFloat :: Int -> a -> a -- multiplies a floating-point number by an integer power of the radix
  isNaN :: a -> Bool -- 'True' if the argument is an IEEE "not-a-number" (NaN) value
  isInfinite :: a -> Bool -- 'True' if the argument is an IEEE infinity or negative infinity
  isDenormalized :: a -> Bool -- 'True' if the argument is too small to be represented in
                              -- normalized format
  isNegativeZero :: a -> Bool -- 'True' if the argument is an IEEE negative zero
  isIEEE :: a -> Bool -- 'True' if the argument is an IEEE floating point number
  atan2 :: a -> a -> a -- a version of arctangent taking two real floating-point arguments.

-- A type f is a Functor if it provides a function 'fmap' which, given any types a and b
-- lets you apply any function from (a -> b) to turn an 'f a' into an 'f b', preserving the
-- structure of f.
class Functor f where
  fmap :: (a -> b) -> f a -> f b
  (<$) :: a -> f b -> f a -- Replace all locations in the input with the same value.

-- A functor with application, providing operations to
--   - embed pure expressions ('pure'), and
--   - sequence computations and combine their results ('<*>' and 'liftA2').
class Functor f => Applicative f where
  pure :: a -> f a -- Lift a value.
  (<*>) :: f (a -> b) -> f a -> f b -- Sequential application.
  GHC.Base.liftA2 :: (a -> b -> c) -> f a -> f b -> f c -- Lift a binary function to actions.
  (*>) :: f a -> f b -> f b -- Sequence actions, discarding the value of the first argument.
  (<*) :: f a -> f b -> f a -- Sequence actions, discarding the value of the second argument.

-- The 'Monad' class defines the basic operations over a /monad/,
-- a concept from a branch of mathematics known as /category theory/.
-- From the perspective of a Haskell programmer, however, it is best to
-- think of a monad as an /abstract datatype/ of actions.
-- Haskell's 'do' expressions provide a convenient syntax for writing
-- monadic expressions.
class Applicative m => Monad m where
  (>>=) :: m a -> (a -> m b) -> m b -- Sequentially compose two actions, discarding any value produced
                                    -- by the first, like sequencing operators (such as the semicolon)
                                    -- in imperative languages.
  (>>) :: m a -> m b -> m b         -- Sequentially compose two actions, passing any value produced
                                    -- by the first as an argument to the second.
  return :: a -> m a                -- Inject a value into the monadic type.

-- The class of semigroups (types with an associative binary operation).
class Semigroup a where
  (<>) :: a -> a -> a -- An associative operation.
  GHC.Base.sconcat :: GHC.Base.NonEmpty a -> a -- Reduce a non-empty list with '<>'
  GHC.Base.stimes :: Integral b => b -> a -> a -- Repeat a value n times.

-- Monoid
-- The class of monoids (types with an associative binary operation that
-- has an identity).
class Semigroup a => Monoid a where
  mempty :: a -- An associative operation
  mappend :: a -> a -> a -- An associative operation
  mconcat :: [a] -> a -- Fold a list using the monoid.

-- Data structures that can be folded.
class Foldable t where
  Data.Foldable.fold :: Monoid m => t m -> m
  foldMap :: Monoid m => (a -> m) -> t a -> m
  Data.Foldable.foldMap' :: Monoid m => (a -> m) -> t a -> m
  foldr :: (a -> b -> b) -> b -> t a -> b
  Data.Foldable.foldr' :: (a -> b -> b) -> b -> t a -> b
  foldl :: (b -> a -> b) -> b -> t a -> b
  Data.Foldable.foldl' :: (b -> a -> b) -> b -> t a -> b
  foldr1 :: (a -> a -> a) -> t a -> a
  foldl1 :: (a -> a -> a) -> t a -> a
  Data.Foldable.toList :: t a -> [a] -- List of elements of a structure, from left to right.
  null :: t a -> Bool -- Test whether the structure is empty. The default implementation is
                      -- optimized for structures that are similar to cons-lists, because there
                      -- is no general way to do better.
  length :: t a -> Int --  Returns the size/length of a finite structure as an 'Int'.
  elem :: Eq a => a -> t a -> Bool -- Does the element occur in the structure?
  maximum :: Ord a => t a -> a -- The largest element of a non-empty structure.
  minimum :: Ord a => t a -> a -- The least element of a non-empty structure.
  sum :: Num a => t a -> a     -- computes the sum of the numbers of a structure.
  product :: Num a => t a -> a -- computes the product of the numbers of a structure.
```

Also their hierarchy:

![Haskell Typeclass Hierarchy](/img/haskell_classtype.svg)

## Types

```haskell
data Bool = False | True

-- `Int` is a fixed-precision integer type with at least the range [-2^29 .. 2^29-1].
-- The exact range for a given implementation can be determined by using
-- 'Prelude.minBound' and 'Prelude.maxBound' from the 'Prelude.Bounded' class.
data Int = GHC.Types.I# GHC.Prim.Int#

-- `Integer` is arbitrary precision integers. In contrast with fixed-size integral types
-- such as 'Int', the 'Integer' type represents the entire infinite range of
-- integers.
data Integer

data Ordering = LT | EQ | GT

-- The character type 'Char' is an enumeration whose values represent
-- Unicode (or equivalently ISO/IEC 10646) code points
data Char = GHC.Types.C# GHC.Prim.Char#

-- A 'String' is a list of characters.  String constants in Haskell are values
-- of type 'String'.
type String = [Char]

-- A parser for a type a, represented as a function that takes a
-- 'String' and returns a list of possible parses as (a,'String') pairs.
type ReadS a = String -> [(a, String)]

-- A value of type `'IO' a` is a computation which, when performed,
-- does some I/O before returning a value of type a.
-- There is really only one way to "perform" an I/O action: bind it to
-- Main.main in your program.  When your program is run, the I/O will
-- be performed.  It isn't possible to perform I/O from an arbitrary
-- function, unless that function is itself in the 'IO' monad and called
-- at some point, directly or indirectly, from Main.main.
-- 'IO' is a monad, so 'IO' actions can be combined using either the do-notation
-- or the 'Prelude.>>' and 'Prelude.>>=' operations from the 'Prelude.Monad'
-- class.
newtype IO a
  = GHC.Types.IO (GHC.Prim.State# GHC.Prim.RealWorld
                  -> (# GHC.Prim.State# GHC.Prim.RealWorld, a #))

-- GHC.Base.NonEmpty
-- Non-empty (and non-strict) list type.
data NonEmpty a = a :| [a]
```
