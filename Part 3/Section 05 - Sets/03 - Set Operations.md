---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.1'
      jupytext_version: 1.2.4
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

### Set Operations


Let's go over the set operations that are available in Python.


##### Intersections


There's two ways to calculate the intersection of sets:

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
```

```python
s1.intersection(s2)
```

```python
s1 & s2
```

We can computer the intersection of more than just two sets at a time:

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
s3 = {3, 4, 5}
```

```python
s1.intersection(s2, s3)
```

```python
s1 & s2 & s3
```

##### Unions


There's also two ways to calculate the union of two sets:

```python
s1 = {1, 2, 3}
s2 = {3, 4, 5}
```

```python
s1.union(s2)
```

```python
s1 | s2
```

We can compute the union of more than two sets:

```python
s3 = {5, 6, 7}
```

```python
s1.union(s2, s3)
```

```python
s1 | s2 | s3
```

##### Disjointedness


Two sets are disjoint if their intersection is empty:

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
s3 = {30, 40, 50}
```

```python
print(s1.isdisjoint(s2))
print(s2.isdisjoint(s3))
```

Of course we could use the cardinality of the intersection instead:

```python
len(s1 & s2)
```

```python
len(s2 & s3)
```

Or, since empty sets are falsy:

```python
bool(set())
```

```python
bool({0})
```

we can also use the associated truth value:

```python
if {1, 2} & {2, 3}:
    print('sets are not disjoint')
```

```python
if not {1, 2} & {3, 4}:
    print('sets are disjoint')
```

##### Differences


The difference of two sets can also be computed in two different ways:

```python
s1 = {1, 2, 3, 4, 5}
s2 = {4, 5}
```

```python
s1 - s2
```

```python
s1.difference(s2)
```

Of course, with the method we can use iterables as well:

```python
s1.difference([4, 5])
```

<!-- #region -->
Note that the difference operator is not commutative, i.e. it does not hold in general that
```
s1 - s2 = s2 - s1
```
<!-- #endregion -->

```python
s2 - s1
```

##### Symmetric Difference


We can calculate the symmetirc difference of two sets also in two ways:

```python
s1 = {1, 2, 3, 4, 5}
s2 = {4, 5, 6, 7, 8}
```

```python
s1.symmetric_difference(s2)
```

```python
s1 ^ s2
```

Remember that the symmetric difference of two sets results in the difference of the union and the intersection of the two sets:

```python
(s1 | s2) - (s1 & s2)
```

##### Subsets and Supersets


With containmnent we have the notion of proper containment (i.e strictly contained, not equal) and just containment (contained, possibly equal).
This is analogous to the concept of (`i < j` and `i <= j`)

```python
s1 = {1, 2, 3}
s2 = {1, 2, 3}
s3 = {1, 2, 3, 4}
s4 = {10, 20, 30}
```

```python
s1.issubset(s2)
```

```python
s1 <= s2
```

For strict containment there is no set method - we have to use the operator, or a combination of methods/operators:

```python
s1 < s2
```

```python
s1.issubset(s2) and s1 != s2
```

```python
s1 < s3
```

```python
s1 <= s4
```

An analogous situation with supersets:

```python
s2.issuperset(s1)
```

```python
s2 >= s1
```

```python
s2 > s1
```

Be careful with these set containment operators, they do not work quite the same way as with numbers for example:

<!-- #region -->
With numbers, if
```
a <= b --> False
```
then it follows that
```
a < b --> True
```

This is not the case with set containment:
<!-- #endregion -->

```python
s1 = {1, 2, 3}
s2 = {10, 20, 30}
```

As you can see these two sets are non-empty and disjoint, and containment works as follows:

```python
s1 <= s2
```

```python
s1 > s2
```

```python
s1 < s2
```

```python
s1 >= s2
```

```python
s1 == s2
```

There's really not a whole lot more to say about the various set operations themselves - they are quite easy.
Where they really shine is in their application to diverse problems, especially when dealing with dictionary keys as we saw earlier.


##### Enhanced Set Methods


There's a slight wrinkle to some of these operations we just saw.

When we use the operators (`&`, `|`, `-`) we have to deal with sets on both sides of the operator:

```python
{1, 2} & [2, 3]
```

But when we work with the method equivalent, we do not have that restriction - in fact the argument to these methods can be an iterable in general, not just a set:

```python
{1, 2}.intersection([2, 3])
```

What happens is that Python implicitly converts any iterable to a set then finds the intersection.


However, these iterables must contain hashable elements - they need not be unique (they will eventually be made to consist of unique elements):

```python
{1, 2}.intersection([[1,2]])
```

This means that when we want to find the intersection of two `lists` for example, we could proceed this way:

```python
l1 = [1, 2, 3]
l2 = [2, 3, 4]
```

```python
set(l1).intersection(l2)
```

##### Side Note: Why the choice of `&`, `|` , `^` for unions, intersections and symmetric differences?

<!-- #region -->
You might be wondering why Python chose those particular symbols.

Python also uses these operators for bitwise manipulation.

`&` and `|` seem like a perfectly natural fit when you consider that
```
s1 & s2
```
means the elements that belong to `s1` **and** `s2`, 

and
```
s1 | s2
```
means the elements that belong to `s1` **or** `s2`.

Let's look at the bitwise operations:
<!-- #endregion -->

Let's look at these two integers:

```python
a = 0b101010
b = 0b110100
```

```python
a, b
```

And these are just two integers, we just chose to create them using a binary literal:

```python
type(a), type(b)
```

Now consider that `1` means `True`, and `0` means `False`:
* `1 and 0` or `1 & 0` --> `0`
* `1 or 0` or `1 | 0` --> `1`
* and so on


Let's use the bitwise Python and (`&`) operator on those two numbers:

```python
c = a & b
print(c)
```

What we really need to do is look at the representation of this result:

```python
bin(c)
```

<!-- #region -->
So this is the result:
```
1 0 1 0 1 0
1 1 0 1 0 0
-----------
1 0 0 0 0 0
```
<!-- #endregion -->

As you can see we performed a bitwise `and` between the two values. Very similar to asking whether `1` is in the intersection of corresponding slots.

The same happens with `|`, the bitwise `or` operator and unions:

```python
c = a | b
```

```python
bin(c)
```

<!-- #region -->
And again, looking at the bits themselves:
```
1 0 1 0 1 0
1 1 0 1 0 0
-----------
1 1 1 1 1 0
```

this is like asking whether `1` is in the union of corresponding slots
<!-- #endregion -->

<!-- #region -->
Now for the symmetric difference.
There is another boolean algebra operation called `xor`, denoted by `^`.
This one works this way:
```
x xor y --> True if x is True or y is True, but not both
```

<!-- #endregion -->

```python
print(bin(a))
print(bin(b))
print(bin(a^b))
```

<!-- #region -->
Let's see the bits again:
```
1 0 1 0 1 0
1 1 0 1 0 0
-----------
0 1 1 1 1 1
```

If we make two corresponding slots into sets and find the symmetric difference between the two, what do we get?
<!-- #endregion -->

```python
{1} ^ {1}
```

```python
{0} ^ {1}
```

```python
{0} ^ {0}
```

So we can ask if `1` is in `{0} ^ {1}` - which is exactly what the bitwise `xor` (`^`) operator evaluates to in the above example.
