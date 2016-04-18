---
layout: post
title: Metamodel (Ecore) Design Checklist
categories: [draft]
tags: [draft]
---

**Do not compromise on your domain model !** Never, ever!
Som many aspects of your tool will trickle down from your Ecore model that it pays a lot to pause for a bit and do some basic sanity checks.
I compiled the following checklist based on my personal experience, this is not exhaustive and I expect it to live and get richer over time.

By the way, feel free to [tell me about your own rules](https://twitter.com/bruncedric), I might add it to the list!


## Identification card

### States the questions a model based on this metamodel will answer and to who

A model is a representation of a system **for a given purpose**. Just like Object Oriented Programming never was about "structuring code so that it's close to the real world" a metamodel doesn't have to match the real world. It makes no sense if it isn't aimed at answering a specific set of questions.

As such a good first task is to start by stating those questions. And for who. The "**who**" is using the model as this might have important implications regarding the naming of the concepts. 
My tool of chocie for this is to take a few minutes and write down a [Personna](https://en.wikipedia.org/wiki/Persona_(user_experience) and get back at it when I need to justify a given choice.

Example: Models from this metamodel will enable *researcher in agriculture* to answer the questions: *how many resources (water, machines, humans) are needed for a given farm structure, in a given region, and for set of cultures (wheat, sorgho..).*

Example: Models from this metamodel will enable *software architects* to answer the questions: *which are the services existing in my system, what are their non-functional characteristics, their signature, who owns them and how are the related to each others ?*

### The nsURI is the definitive one and is consistent with your naming conventions

As part of this first step of setting up an identification card for your Metamodel, you **have** to stop for a minute for the the EPackage nsURI.
This nsURI will identify your Ecore model, starting now and forever. It is used in many places, in the generated Java code, in the ``plugin.xml`` file of your project, but more importantly other tools or Ecore models are likely to use this URI to identify your Ecore model (in code generators, model transformers...).

Changing this is a pain. Make sure the ``nsURI`` you picked is sensible and matches your naming conventions in this regard. The most important thing is to be consistent and that's not a given.

<figure>
    <a href="{{ site.url }}/images/blog/nsURIs.png"><img src="{{ site.url }}/images/blog/nsURIs.png"></a>    
    <figcaption>NsUris in the modeling package.</figcaption>
</figure>

### Make sure sub EPackages are not used

An additional note from the previous one. There is no such thing as a **sub**-EPackage. Allowing the definition of subpackages within an EPackage was, in retrospect, a bad decision as there is no special meaning here. A domain =  an EPackage, its identification => the nsURI.
There is a good reason Ed did not allow this to be when designing XCore. Just don't use it or you'll be exposed to slightly different intepretation of this among tools (do I have to import a subpackage when I already imported the parent one ? The other way around ?).

One EPackage,  one ``.ecore`` file.

## Design

### Names are existing in a dictionnary, are precise and consistent

Naming things is hard, and just like in every design activity it is of the most critical importance. For non-native english speakers it gets even harder as we might lack some vocabulary or some subtil interpretation might escape us.

Basic rules of thumb:
- use [PowerThesaurus](https://www.powerthesaurus.org/), make sure the name is the most precise you can get.
- use the user background to pick the right name (having defined the Personna comes in handy)
- try to avoid names which are so general or abstract that they could be interpreted in many different ways by your target users. ``Artifact``, ``Element`` are probably fairly bad names (but again, use the context to decide).


### Reference and attribute names are consistent

Check that you stick with a concistent convention for your references. The main decisions which are in front of you:
- do you pluralize the references with a **many** upper bound ?
- do you add a prefix like `owned` for any containment reference?
- do you add a prefix like `parent` for any container reference?
- do you add a prefix for any `derived` reference or attribute? 

### Are all the non-abstract EClasses supposed to be instanciated ?

I experienced this quite a lot in the very early phases, you start with a concept as an EClass and at some point you specialized it, and in the end you have an abstract EClass but you just forgot to make it abstract, leading to the possibility to instanciate it. 
Hold on, go through all the concepts which you don't want to be instanciable and make sure they are "abstract" or "interface".

### Required, non required ?

Go through all the attributes and reference and think again: does an instance **makes any sense** if this attribute is not valued ?

### Contained by who ?

Ecore provides a notion of **containment** reifying the basic lifecycle of an instance. If an object `A` is contained in an object `B` then whenever the object `B` is removed or deleted the object 'A' is too. Thinking about your model as a tree helps in those cases: either your object is expected to be a the root of a resource or it has to be contained by another object.  The goal here is to make a conscious decision about when should an instance disappear from the model. 

Also note that this containment relationship might be leveraged as part of the referencing of an element.

### Named every validation rule which is not enforced by the Ecore model structure itself.

While designing capture and name every validation rule which comes up. You should be able to come up with a name and hopefully a description of valid and invalid cases.

### The concepts are all documented?

Make sure you have documented all the EClasses or relationship which are not completely obvious. We use annotations directly in Ecore to capture the developper or specifier facing documentation. You can also value an attribute in the Genmodel for the user documentation.

### Boolean attributes

Boolean attributes can get tricky over time, a simple EClass with a couple of EAttributes can grow to a monster with many more, each one acting as a configuration "flag". When thinking about all the possible combinations and making sure they are all making gets hairy, you might want to consider a couple of EEnumeration to capture the transversal characteristics instead.

Also check your boolean attributes naming. The EMF Java generator will add an "is" prefix on your API, you don't have to do it, but make sure ``isMyName`` makes sense.

## Scalability related


**Instances which will be present a lot in the models have a terse serialization**

Ask yourself:  how many instances of this EClass will I have in a nominal model? If the answer is "quite a lot"(100K for instance) then check how will they be serialized and make sure there is not an improvement you could bring here. This is also very true for custom datatypes. Once you define this custom datatype EMF requires you to write the ``fromString()`` and ``toString()`` methods for it. If this datatype is being used by a lot of instances then make sure your serialization is terse.

**I'm positive everything which is serialized needs to be serialized**

I need this model, but are there parts which have no need to be serialized? This boils down to which information is captured by the users versus infered or with a lifecycle which is shorter than "load the file"->"get the data".

**Every EClass deserve to lead to full blown EObjects and would not be better as an EDatatype**

Any EClass used for an important number of instances should be inspected and a conscious decision should be made about whether it is best modeled as an EClass or as an EDatatype. Even with the [EMF Ultra Slim Diet](http://ed-merks.blogspot.fr/2009/01/emf-ultra-slim-diet.html) an EObject comes with an overhead, both in term of memory usage but even more importantly in the overhead framework code might induce (cross-referencers, change recorders..)

Rule of thumb: if you have might have many instances and don't care about the indidual notifications of each attribute of the EObject, it's probably best to model it as an EDatatype. An EClass only having EAttributes and not EReferences is also a clear indication that this might be a good candidate for being an EDatatype.


## Java-related implications

**Multiple inheritance is not over-used**

Ecore allows for multiple inheritance. But in the end your Ecore model is transformed into Java code and Java only allows multiple interfaces to be implemented. The EMF code generator hides that for you and might ends up dupplicating code to make sure everything works as expected. A few things to keep in mind:
- the order of the inheritance matters: the implementation class will extends the implementation class of the **first** EClass in the list of supertypes, the subsequent classes will lead to dupplicated code.
- just like for Object Oriented designs, having a lot of multiple inheritance screams of a design which is not really splitting concerns (or not the right ones)

**Custom DataType**

EMF provides off-the-shelve datatypes for Strings, Integer, Float, Long and their primitive counterparts as *EString*, *EInt* ...

But *String* is a technical concern and it often is relevant to replace usages of the *EString* by your own EDataType which express a domain specific type but is mapped to ``java.lang.String`` all the same. It makes the Ecore model more explicit and paves the way for a behavior which can be specific to this particular type.

## Outside world

**I though about how instances are going to be referenced from the outside**

Any EObject which is contained in a resource has a URI and might be referenced by others. But there are so many ways to identify an instance. You roughly have to decide in between: Resource specific identification like XMI-IDs, or domain related identification by defining an **id** EAttribute  or by using the EReference **eKeys**.

The default behavior uses the containment relationship and the index of the object within it's containing reference. This solution is not suitable for 99% of the cases as any addition or removal in a reference might break references (but this is the default one in EMF as it is the only one which assumes nothing about the serialization format or the EClasses).

**One can't introduce cyclic references in between model fragments**

If you are planning to split your model on multiple files or if part of it is to be referenced by other models, then you should make sure that introducing such references is not supposed to modify the referenced instance. These situations can easily arise when using EOpposite references. Keep in mind that many EMF technologies will provides you with a way to easily and efficiently navigate on inverse references which are not designed as such in the Ecore model.
For instance using Sirius you might write queries like:
`aql:self.eInverse(some::Type)` to retrieve any instance of `Type` referencing the object `self` 
or `aql:self.eInverse(anEReferenceName)` to navigate on the inverse of the reference `anEReferenceName` from the objeect `self`.


**My EPackage have low coupling with other EPackages**

Inheritance or references in between EPackages can quickly get tricky (and the former even sooner than the later). It is so easy to do using the modeling tools that one can easily abuse it, but in the end your Ecore model is translated to Java and OSGi components, you'll have to deal with the technical coordination.

As such, only introduce inter-EPackage relationships for compelling reasons and when you do, make sure you either only reference EClasses or if you need to subclass make sure you are able to cope with a strong coupling on the corresponding component.

**The places which might be extended by subtypes are clearly identified**

This item is symetric from the previous one: if one of your goal is for others to provide subtypes your domain model, explicitely design for it and document it.







