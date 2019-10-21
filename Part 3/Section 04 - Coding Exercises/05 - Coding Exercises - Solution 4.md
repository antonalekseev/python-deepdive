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

### Coding Exercises - Solution 4


#### Exercise 4


For this exercise suppose you have a web API load balanced across multiple nodes. This API receives various requests for resources and logs each request to some local storage. Each instance of the API is able to return a dictionary containing the resource that was accessed (the dictionary key) and the number of times it was requested (the associated value).

Your task here is to identify resources that have been requested on some, but not all the servers, so you can determine if you have an issue with your load balancer not distributing certain resource requests across all nodes.

For simplicity, we will assume that there are exactly 3 nodes in the cluster.

You should write a function that takes 3 dictionaries as arguments for node 1, node 2, and node 3, and returns a dictionary that contains only keys that are not found in **all** of the dictionaries. The value should be a list containing the number of times it was requested in each node (the node order should match the dictionary (node) order passed to your function). Use `0` if the resource was not requested from the corresponding node.


Suppose your dictionaries are for logs of all the GET requests on each node:

```python
n1 = {'employees': 100, 'employee': 5000, 'users': 10, 'user': 100}
n2 = {'employees': 250, 'users': 23, 'user': 230}
n3 = {'employees': 150, 'users': 4, 'login': 1000}
```

Your result should then be:

```python
result = {'employee': (5000, 0, 0),
          'user': (100, 230, 0),
          'login': (0, 0, 1000)}
```

Tip: 
to find the difference between two sets, you can subtract one from the other:

```python
s1 = {1, 2, 3, 4}
s2 = {1, 2, 3}
s1 - s2
```

Tip: to get the union of two (or more) sets you can use the `|` operator:

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
s1 | s2
```

Tip: to get the intersection of two (or more) sets you can use the `&` operator:

```python
s1 = {1, 2, 3, 4}
s2 = {2, 3}
s1 & s2
```

Hint: It might be helpful to draw out a set diagram and consider what subset you are trying to isolate.


##### Solution


The approach I am going to take here is to merge all the keys into a single set, then remove from it the intersection of all the keys (i.e. remove keys that are common to all dictionaries).
Once I have that set of keys, I will pull the frequency from each dictionary (node) and build up a list of these frequencies.

```python
n1 = {'employees': 100, 'employee': 5000, 'users': 10, 'user': 100}
n2 = {'employees': 250, 'users': 23, 'user': 230}
n3 = {'employees': 150, 'users': 4, 'login': 1000}
```

```python
union = n1.keys() | n2.keys() | n3.keys()
intersection = n1.keys() & n2.keys() & n3.keys()
```

```python
union, intersection, union - intersection
```

```python
def identify(node1, node2, node3):
    union = node1.keys() | node2.keys() | node3.keys()
    intersection = node1.keys() & node2.keys() & node3.keys()
    relevant = union - intersection
    result = {key: (node1.get(key, 0),
                    node2.get(key, 0),
                    node3.get(key, 0))
              for key in relevant}
    return result        
```

```python
result = identify(n1, n2, n3)
for k, v in result.items():
    print(f'{k}: {v}')
```
