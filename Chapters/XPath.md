## Little Journey into XPath


XPath is the de factor standard language for navigating an XML document and selecting nodes from it. XPath expressions act as queries that identifies nodes. In this chapter we will go through the main concepts and show some of the ways we can access nodes in a xml document. All the expressions can be executed on the spot, so do not hesitate to experiment with them.

### Getting started


You should load the XML parser and XPath library as follows:
```
Metacello new
	baseline: 'XMLParserHTML';
	repository: 'github://pharo-contributions/XML-XMLParserHTML:1.6.x/src';
	load.
```


```
Metacello new
	baseline: 'XPath';
	repository: 'github://pharo-contributions/XML-XPath:2.2.x/src';
	load.	
```



### An example


As an example we will take the possible representation of Magic cards, starting with the 
 Arcane Lighthouse that you can view at [http://gatherer.wizards.com/Pages/Card/Details.aspx?multiverseid=389430](http://gatherer.wizards.com/Pages/Card/Details.aspx?multiverseid=389430)
and is shown in Figure *@ligthouse@*. 

![http://gatherer.wizards.com/Pages/Card/Details.aspx?multiverseid=389430.](figures/lighthouse.png width=100&label=ligthouse)

```
	<?xml version="1.0" encoding="UTF-8"?>

	<cardset>
	  <card>
	    <cardname lang="en">Arcane Lighthouse</cardname>
	    <types>Land</types>
	    <year>2014</year>
		<rarity>Uncommon</rarity>
		<expansion>Commander 2014</expansion>
	    <cardtext>Tap: Add 1 uncolor to you mana pool. 
		1 uncolor + Tap: Until end of turn, creatures your opponents
		 control lose hexproof and shroud and can't have 
		 hexproof or shroud.</cardtext>
	  </card>
	</cardset>
```


### Creating a tree of objects


In Pharo it is always powerful to get an object and interact with it. 
So let us do that now using the `XMLDOMParser` to convert our data in a tree of objects \(as shown in Figure *@inspectorx@*\).
Note that the escaped the `'` with an extra quote as in `can''t`. 

```
	| tree |
	tree := (XMLDOMParser on: 
	'<?xml version="1.0" encoding="UTF-8"?>

	<cardset>
	  <card>
	    <cardname lang="en">Arcane Lighthouse</cardname>
	    <types>Land</types>
	    <year>2014</year>
	    <rarity>Uncommon</rarity>
	    <expansion>Commander 2014</expansion>
	    <cardtext>Tap: Add 1 uncolor to you mana pool. 
		1 uncolor + Tap: Until end of turn, creatures your opponents
		 control lose hexproof and shroud and can''t have 
		 hexproof or shroud.</cardtext>
	  </card>
	</cardset>') parseDocument
```



![Grabbing and playing with a tree.](figures/xpath1.png width=100&label=inspectorx)

### Nodes, node sets and atomic values

We will be working with three kinds of XPath constructs: nodes, node sets, and atomic values.

Node sets are sets \(duplicate-free collections\) of nodes. All node sets produced by XPath location path expressions are sorted in document order, the order in the document source that they appear in.

The following elements are nodes:
```
<cardset> (root element node)

<cardname lang="en">Arcane Lighthouse</cardname> (element node)

lang="en" (attribute node)
```


Atomic values are strings, numbers, and booleans. Here are some examples of atomic values:

```
Arcane Lighthouse
 
"en"

2.5

-1

true

false
```



### Basic tree relationships


Since we are talking about trees, nodes can have multiple relationships with each other: parent, child and siblings. 
Let us set some simple vocabulary. 

- **Parent.** Each node can have at most one parent. The root node of the tree, usually a document, has no parent. In the Arcane Lighthouse example, the card element is the parent of the cardname, types, year, rarity, expansion and cardtext elements. In XPath, attribute and namespace nodes treat the element they belong to as their parent.


- **Children.** Document and element nodes may have zero, one or more children, which can be elements, text nodes, comments or processing instructions. The cardname, types, year, rarity, expansion and cardtext elements are all children of the card element. Confusingly, even though attribute and namespace nodes can have element parents in XPath, they aren't children of their parent elements.


- **Siblings.** Siblings are child nodes that have the same parent. The cardname, types, year, rarity, expansion and cardtext elements are all siblings. Attributes and and namespace nodes have no siblings.


- **Ancestors.** A node's parent, parent's parent, etc. Ancestors of the cardname element are the card element and the cardset nodes. 


- **Descendants** A node's children, children's children, etc. Descendants of the cardset element are the card,cardname, types, year, rarity, expansion and cardtext elements.



### A large example


Let us expand our example to have cover more cases.

```
	
	| tree |
	tree := (XMLDOMParser on: 
	'<?xml version="1.0" encoding="UTF-8"?>

	<cardset>
	  <card>
	    <cardname lang="en">Arcane Lighthouse</cardname>
	    <types>Land</types>
	    <year>2014</year>
	    <rarity>Uncommon</rarity>
	    <expansion>Commander 2014</expansion>
	    <cardtext>Tap: Add 1 uncolor to you mana pool. 
		1 uncolor + Tap: Until end of turn, creatures your opponents
		 control lose hexproof and shroud and can''t have 
		 hexproof or shroud.</cardtext>
	  </card>
	  <card>
	    <cardname lang="en">Desolate Lighthouse</cardname>
	    <types>Land</types>
	    <year>2013</year>
	    <rarity>Rare</rarity>
	    <expansion>Avacyn Restored</expansion>
	    <cardtext>Tap: Add Colorless to your mana pool.
	1BlueRed, Tap: Draw a card, then discard a card.</cardtext>
	  </card>
	</cardset>') parseDocument
```


![Select the raw tab and click on self in the inspector.](figures/xpath2.png width=100&label=inspector2)

Select the raw tab and click on self in the inspector \(as shown in Figure *@inspector2@*\). Now we are ready to learn XPath.

### Node selection


The following table shows the XPath expressions. Often the current node is also named the context. 


| **Expression** | **Description** |  |
| nodename | Selects all child nodes with the name "nodename" |  |
| / | Selects the root node |  |
| // | Selects any node from the current node that |  |
|  | matches the context selection |  |
| . | Selects the context \(current\) node |  |
| .. | Selects the parent of the context \(current\) node |  |
| @ | Selects attributes of the context node |  |


In the following we expect that the variable `tree` is bound the full document tree we previously created parsing the XML string.
Location path expressions return node sets, which are empty if no nodes match. Now let us play with the system to really see how it works.

#### Node tag name selection


There are several way to test and select nodes.


| **nodename** | Selects all child nodes with the name "nodename" |  |
| card | Selects all child nodes with the name "card" |  |
| **prefix:localName** | Selects all child nodes with the qualified |  |
|  | name "prefix:localName"  or if at least one prefix |
|  | or namespace URI pair was declared in the |  |
|  | XPathContext, the child nodes with the local name |  |
|  | "localName" and the namespace URI bound to "prefix" |  |

In standard XPath, qualified name tests like prefix:localName select nodes with the same local name and the namespace URI of the prefix, which must be declared in the controlling XPath context prior to evaluation. The selected nodes from the document can have different prefixes \(or none at all\), because matching is based on local name and namespace URI.

To simplify things, the Pharo XPath library \(unlike others\) by default matches qualified name tests against the literal qualified names of nodes, ignoring namespace URIs completely, and does not require you to pre-declare namespace prefix/URI pairs in the XPathContext object before evaluation. Declaring at least one namespace prefix/URI pair will trigger standard behavior, where all prefixes used in qualified name tests must be pre-declared, and matching will be done based on local names and namespace URIs.

#### Context and parent



| . | Selects the current context node |  |
| .. | Selects the parent of the current context node |  |

The following expression shows that `.` \(period\) selects the context node, initially the node XPath evaluation begins in.

```testcase=true
(tree xpath: '.') first == tree
>>> true
```




#### Matching path-based child nodes


The operator `/` selects from the root node.


| **/** | **Selects from the root node** |  |
| /cardset | Selects the root element cardset |  |
| cardset/card | Selects all the card grandchildren |  |
|  | from the cardset children of the context node |  |

The following expression selects all the card nodes under cardset node.

```
path := XPath for: '/cardset/card'.
path in: tree.
```


`XPath` objects lazily compile their source to an executable form the first time they're evaluated, and the compiled form and its source are cached globally, so caching the `XPath` object itself in a variable is normally unecessary to avoid recompilation and is only slightly faster. The previous expression is equivalent to the following expression using the `xpath:` message.

```
tree xpath: '/cardset/card'
```




#### Matching deep nodes


The `//` operation selects all the nodes matching the selection.



| **//** | Selects from the context \(current\) node and all descendants |  |
| //year | Selects all year node children of the context node and |  |
|  | of its descendants |  |
| cardset//year | Selects all year node children of the cardset context |  |
|  | node children and their descendants |  |

Let us try with another element such as the expansion of a card. 
```testcase=true
tree xpath: '//expansion'
>>>
a XPathNodeSet(<expansion>Commander 2014</expansion> <expansion>Avacyn Restored</expansion>)
```


The XPath library extends `XMLNode` classes with binary selectors to encode certain XPath expressions directly in Pharo. So the previous expression can be expressed as follows using the message `//`:

```testcase=true
tree // 'expansion'
>>>
a XPathNodeSet(<expansion>Commander 2014</expansion> <expansion>Avacyn Restored</expansion>)
```



#### Identifying attributes


`@` matches attributes. 


| **Expression** | **Description** |  |
| @ | Selects attributes |  |
| //@lang | Selects all attributes that are named lang |  |
 


The following expression returns all the attributes whose name is `lang`.
```testcase=true
(tree xpath: '//@lang') 
>>>  a XPathNodeSet(lang=""en"" lang=""en"")
```




### Predicates


Predicates are used to find a specific node or a node that contains a specific value. Predicates are always embedded in square brackets.

Let us study some examples:

#### First element


The following expression selects the first card child of the cardset element.
```testcase=true
tree xpath: '/cardset/card[1]'
>>>
a XPathNodeSet(<card>
    <cardname lang=""en"">Arcane Lighthouse</cardname>
    <types>Land</types>
    <year>2014</year>
    <rarity>Uncommon</rarity>
    <expansion>Commander 2014</expansion>
    <cardtext>Tap: Add 1 uncolor to you mana pool. 
	1 uncolor + Tap: Until end of turn, creatures your opponents
	 control lose hexproof and shroud and can't have 
	 hexproof or shroud.</cardtext>
  </card>)
```


In the XPath Pharo implementation, the message `??` can be used for position or block predicates.

the previous expression is equivalent to the following one

```
tree / 'cardset' / ('card' ?? 1) .
```



Block or position predicates can be applied with `??` to axis node test arguments or to result node sets. 

The following expression returns the first element of each 'card' descendant:

```testcase=true
tree // 'card' / ('*' ?? 1)
>>> "a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname> <cardname lang=""en"">Desolate Lighthouse</cardname>)"
```


#### Other position functions


The following expression selects the last card node that is the child of the cardset node.

```
tree xpath: '/cardset/card[last()]'.
```


The following selects the second to last node. In our case since we only have two elements we get the first. 

```testcase=true
tree xpath: '/cardset/card[last()-1]'.
>>>
a XPathNodeSet(<card>
	    <cardname lang=""en"">Arcane Lighthouse</cardname>
	    <types>Land</types>
	    <year>2014</year>
	    <rarity>Uncommon</rarity>
	    <expansion>Commander 2014</expansion>
	    <cardtext>Tap: Add 1 uncolor to you mana pool. 
		1 uncolor + Tap: Until end of turn, creatures your opponents
		 control lose hexproof and shroud and can't have 
		 hexproof or shroud.</cardtext>
	  </card>)
```


We can also use the position function and use it to identify nodes. The following selects the first two card nodes that are children of the cardset node. 

```testcase=true
(tree xpath: '/cardset/card[position()<3]') size = 2
>>> true
```



#### Selecting based on node value


In addition we can select nodes based on a value of a node. The following query selects all the card nodes \(of the cardset\) that have a year greater than 2014.

```
tree xpath: '/cardset/card[year>2013]'.
```


The following query selects all the cardname nodes of the card children of cardset that have a year greater than 2014.

```testcase=true
/cardset/card[year>2013]/cardname
>>> a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname>)
```


#### Selecting nodes based on attribute value


We can also select nodes based on the existence or value of an attribute.
The following expression returns the cardname that have the lang attribute and whose value is 'en'.
```testcase=true
tree xpath: '//cardname[@lang]
>>> a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname> <cardname lang=""en"">Desolate Lighthouse</cardname>)
tree xpath: '//cardname[@lang='en']
```


Note that we can simply get the card from the name using '..'.

```testcase=true
tree xpath: '//cardname[@lang='en']/..
>>>
```


### Selecting Unknown Nodes


In addition we can use wildcard to select any node. 


| **Wildcard** | **Description** |
| \* | Matches any element node |  |
| \@\* | Matches any attribute node |  |
| node\(\) | Matches any node of any kind |  |

For example `//*` selects all elements in a document. 

```testcase=true
(tree xpath: '//*') size
>>> 15
```


While `//@*` selects all the attributes of any node. 

```testcase=true
tree xpath: '//@*'
>>> a XPathNodeSet(lang=""en"" lang=""en"")
```


For example `//cardname[@*]` selects all cardname elements which have at least one attribute of any kind.
```testcase=true
tree xpath: '//cardname[@*]'
>>> a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname> <cardname lang=""en"">Desolate Lighthouse</cardname>)
```

The following expression selects all child nodes of cardset.
```
tree xpath: '/cardset/*'.
```


The following expression selects all the cardname of all the child nodes of cardset. 
```
tree xpath: '/cardset/*/cardname'.
```


### Handling multiple queries


By using the `|` union operator in an XPath expression you can select several paths.
The following expression selects both the cardname and year of card nodes located anywhere in the document. 

```testcase=true
tree xpath: '//card/cardname | //card//year'
>>> a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname> <year>2014</year> 
<cardname lang=""en"">Desolate Lighthouse</cardname> <year>2013</year>)"
```


### XPath axes


XPath introduces another way to select nodes using _location step_ following the syntax: `axisname::nodetest[predicate]`.
Such expressions can be used in the steps of location paths \(see below\). 

An axis defines a node-set relative to the context \(current\) node. Here is a table of the available axes. 
Except for the namespace axis, all of these have binary selector equivalents.


| **AxisName** | **Result** |
| ancestor | Selects all context \(current\) node ancestors |
| ancestor-or-self | ... and the context node itself |
| attribute | Selects all context \(current\) node attributes |
| child | Selects all context \(current\) node children |
| descendant | Selects all context node descendants |
| descendant-or-self | ... and the context node itself |
| following | Selects everything after the context node closing tag |
| following-sibling | Selects all siblings after the context node |
| namespace | Selects all context node namespace nodes |
| parent | Selects context node parent |
| preceding | Selects all nodes that appear before the context node |
|  | except ancestors, attribute nodes and namespace nodes |
| preceding-sibling | Selects all siblings before the context node |
| self | Selects the context node |


#### Paths 


A location path can be absolute or relative. An absolute location path starts with a slash \( / \) \(/step/step/...\) and a relative location path does not \(step/step/...\). In both cases the location path consists of one or more location steps, each separated by a slash.

Each step is evaluated against the nodes in the context node-set.
A location step, `axisname::nodetest[predicate]`,  consists of:
- an axis \(defines the tree-relationship between the selected nodes and the context node\)
- a node-test \(identifies a node within an axis\)
- zero or more predicates \(to further refine the selected node-set\)


The following example access the year node of all the children of the cardset. 
```testcase=true
tree xpath: '/cardset/child::node()/year').
>>>a XPathNodeSet(<year>2014</year> <year>2013</year>)
```


The following expression gets the ancestor of the year node and selects the cardname.
```testcase=true
(tree xpath: '/cardset/card/year') first  xpath: 'ancestor::card/cardname'
>>> "a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname>)"
```


The previous expression could be rewritten using a position predicate. Parentheses are needed so the predicate applies to the entire node set produced by the absolute location path, rather than just the last step, otherwise it would select the first year of each card, instead of the first year overall:
```testcase=true
(tree xpath: '(/cardset/card/year)[1]/ancestor::card/cardname'
>>> "a XPathNodeSet(<cardname lang=""en"">Arcane Lighthouse</cardname>)"
```







### Conclusion


XPath is a powerful language. The Pharo XPath library developed and maintained by Monty van OS and the Pharo Extras Team  implements the full standard 1.0. Coupled with the live programming capabilities of Pharo, it gives a really powerful way to explore structured XML data. 



