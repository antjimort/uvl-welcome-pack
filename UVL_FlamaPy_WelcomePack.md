# UVL + Flamapy: Welcome Pack (First versión)

---

## Index
1. [Introduction](#1-introduction)
2. [What is UVL?](#2-what-is-uvl)

    2.1 [UVL model example](#21-uvl-model-example)

    2.2 [Basic structure of a UVL model](#22-basic-structure-of-a-uvl-model)

    2.3 [Syntax: Language specification](#23-syntax-language-specification)

    2.4 [Languaje design](#24-language-design)
    
    - [mandatory](#mandatory)
    - [optional](#optional)
    - [alternative](#alternative)
    - [or](#or)
    - [Cross-Tree Contraints](#cross-tree-contraints)

    2.5 [Language levels](#25-language-levels)
    - [Boolean level](#boolean-level)
        - [Group cardinality](#group-cardinality)
    - [Arithmetic level](#arithmetic-level)
        - [Feature cardinality](#feature-cardinality)
        - [Aggregate functions](#aggregate-functions)
    - [Type level](#type-level)

3. [What is UVLHub?](#3-what-is-uvlhub)
   

4. [What is Flamapy?](#4-what-is-flamapy)
    4.1 [Flamapy components](#41-flamapy-components)
    4.2 [Basic flow](#42-basic-flow)
    4.3 [Metamodels](#43-metamodels)
    4.4 [Operations](#44-operations)


---

## 1. Introduction
Variability management in software systems is a fundamental discipline for modeling, configuring and analyzing product families and customizable systems.
In this context, two key tools emerge as reference standards: UVL (Universal Variability Language) and Flamapy.

UVL is a feature modeling language that allows the different possible variants and configurations of a system to be represented in a simple, modular and human-friendly way. Its readable syntax and modular approach facilitate both the creation and maintenance of variability models, making UVL an ideal choice for new users and projects of any size.

Flamapy, on the other hand, is an extensible and modular framework developed in Python, designed to work directly with variability models such as those described in UVL. It provides a common infrastructure to perform essential operations such as reading, transforming, analyzing and solving models, connecting different solvers and supporting different formats.

This document aims to provide a quick start guide for new users of UVL and Flamapy, covering the basic concepts you need to understand, the tools you will use and how to start working with them efficiently.

In the next sections you will learn:
- What UVL is and how to build feature models.
- How to use UVL Hub, an online platform to easily create and validate UVL models.
- What Flamapy is and how you can use it to process and analyze UVL models using Python.
- How UVL, UVL Hub and Flamapy are connected within a practical workflow.
- How to quickly install the tools and perform your first usage example in a few minutes.
- Additional resources for further learning and practical tips for beginners.

The goal is that, by the end of this guide, you will have a clear vision of how to model variability using UVL and how to take advantage of Flamapy's potential to analyze and work with your models.


---

## 2. What is UVL?

**UVL** (*Universal Variability Language*) is a **textual** and **modular** language designed to describe **feature models** in a clear, structured and easily processable way by both humans and automatic tools.

A feature model is a representation of all possible configurations of a system or product family, describing the different functionalities and options that can be included or excluded.

UVL stands out for being:

- **Easy to read and write**: Its syntax is light and close to natural language, making it easy for people with no previous experience in formal languages to understand and edit the models.
- **Modular**: Allows large models to be divided into multiple independent modules, facilitating reuse, maintenance and collaboration in complex projects.
- **Interoperable**: Designed as an open standard, UVL can act as a bridge between different modeling, analysis and variability configuration tools.

Thanks to these features, UVL has become a modern and accessible option for modeling the variability of complex systems in different domains, such as configurable software, industrial product lines or digital platforms. In the following, we will detail a little more about what we can do with this feature language.

You can access to the UVL repositories here: https://github.com/Universal-Variability-Language
Plus, this document is based on this two resources:
- Research paper with full documentation (by David Benavides, Chico Sundermann, Kevin Feichtinger, José A. Galindo, Rick Rabiser, Thomas Thüm): https://www.sciencedirect.com/science/article/pii/S0164121224003704?via%3Dihub
- Oficial web page: https://universal-variability-language.github.io/

### 2.1. UVL model example
The following is a basic example of a UVL model describing the possible configurations of a car in a UVL file:

```
features
    Smartwatch
        mandatory
            Display
                alternative
                    LCD
                    OLED
                    "E-Ink"
            Connectivity
                or
                    Bluetooth
                    "Wi-Fi"
                    LTE
            Sensors
                mandatory
                    "Heart Rate Monitor"
                optional
                    GPS
                    "Blood Oxygen Sensor" {Accuracy 95}
                    Accelerometer
                    Gyroscope
            Battery
                alternative
                    Standard {Capacity 300}
                    "Long Life" {Capacity 450}
        optional
            "Music Player" {Storage 2}
            NFC
            "Voice Assistant"
            "Fall Detection"
            "Sleep Tracking"
            "Water Resistance"
                alternative
                    IP67
                    IP68
                    IPX8
            "Watch Faces" cardinality [5..50]
            Accessories
                [0..2]
                    "Leather Strap"
                    "Metal Strap"
                    "Silicone Strap"
            String Color
            Float Weight
            String "Serial Number"

constraints
    LTE => (GPS & Bluetooth)
    IPX8 => (!LCD)
    "Fall Detection" => (Accelerometer & Gyroscope)
    "Voice Assistant" <=> "Wi-Fi"
    sum(Capacity) >= 400
    avg(Accuracy) >= 90
    len("Serial Number") == 10
    Weight > 30.0
    Color != "Pink"
```


### 2.2 Basic structure of a UVL model
Every UVL file has two main blocks:
- `features`: defines the hierarchical structure of features, so it is the core of the document.
- `constraints`: defines logical rules between features that cannot be defined in `features` and that must be fulfilled to guarantee the consistency of the described model, either by business rules of a product or by other requirements.

There are two optional blocks that you may use if you need them:
- `include`: allows importing models or extensions.
- `import`: allows to import other feature submodels from the same directory. This helps to have a cleaner organization of models and features.

These last two must be put at the top of the UVL file.

### 2.3 Syntax: Language specification

#### Imports
The imports mechanism allows you to split a large model into smaller, more manageable submodels. Using the `imports` keyword, other .uvl files can be imported, facilitating model organization and teamwork. Each submodel can be aliased by `as`, which allows referencing its features using the `alias.FeatureName` notation. Both in the feature tree and in the constraints, imported elements can be accessed in this way. Imports help to keep the model clean, modular and allow reuse of common parts, making maintenance and growth of the model much easier.

For instance:
```
imports
    platform as pl
    security

features
    eShop
        mandatory
            security.Security
        optional
            pl.Platform
```
where the models `platform` and `security` are submodels defined in a UVL file located in the same directory.

#### Feature tree
In UVL, the feature tree is composed of features and groups. The tree must have exactly one root `feature` and each feature can have one or more groups of features. The relationship between features and groups is expressed by indentation: groups are indented with respect to their parent feature, and features are indented with respect to their parent group. Each feature must have a unique name as identifier, using quotation marks if it contains special symbols or whether the name of the feature requires more than one word. In addition, each feature can be assigned a feature cardinality `[n..m]`, which indicates how many times it can be selected, and a list of attributes {att1 value1, att2 value2, ...}. UVL supports five types of groups: Optional, Mandatory, Or, Alternative (all Boolean level), and Cardinality (a minor level of the Boolean level).


### 2.4 Language design
UVL defines variability models using a tree-like structure.
Each node of the tree is a feature and the relationships between nodes capture the selection rules.

The core of UVL includes several important concepts to specify how features can or should be selected:

#### `mandatory`
A mandatory feature must be selected if its parent feature is selected. For example, for the model provided:
```
Sensors
    mandatory
        "Heart Rate Monitor"
```
where if _Sensor_ is selected, _"Heart Rate Monitor"_ must be selected too.

#### `optional`
An optional feature can be selected or not, depending on whether it provides value in that particular configuration. For example, for the model provided:
```
Sensors
    optional
        GPS
        "Blood Oxygen Sensor" {Accuracy 95}
        Accelerometer
        Gyroscope
```
where if _Sensor_ is selected, you can decide whether or not to add _GPS_, _"Blood Oxygen Sensor"_, _Accelerometer_ or _Gyroscope_.

#### `alternative`
Of all possible options, exactly one child characteristic must be selected if its parent is selected. For example, for the model provided:
```
Display
    alternative
        LCD
        OLED
        "E-Ink"
```
where if _Display_ is selected, you must choose exactly one between _LCD_, _OLED_ or _"E-Ink"_.

#### `or`
An OR group allows to select one or more daughter characteristics if their parent is selected. For example, for the model provided:
```
Connectivity
    or
        Bluetooth
        "Wi-Fi"
        LTE
```
where multiple options can be chosen, at least one (e.g. _"Wi-Fi"_ and _Bluetooth_).

#### Cross-Tree Contraints
Logical constraints that relate features that are not in the same branch of the tree and that allow defining types of constraints that cannot be defined in the ways discussed above. They are defined in the `contraints` block of the UVL file. 

There are many types:
 - Negation with `!A`: used in booleans to express negation.
 - Conjunction with `A & B`: both A and B must be selected.
 - Disjunction with `A | B`: at least one between A or B must be selected. It is analogous to `or`.
 - Implication with `A => B`: indicates that, if A is selected, then B must also be selected.
 - Equality with `A <=> B`: indicates that either both are selected, or neither is selected.

For more complex logical formulas between characteristics, parentheses can always be used to enclose clauses.

For example, we have: 

```
constraints
    "Fall Detection" => (Accelerometer & Gyroscope)
```
where if _"Fall Detection"_ is selected, then it must be selected to be _Accelerometer_ and not _Accelerometer_ and not _Gyroscope_.

Otro ejemplo sería:
```
constraints
    "Voice Assistant" <=> "Wi-Fi"
```
where _"Voice Assistant"_ and _"Wi-Fi"_ go together, so if _"Voice Assistant"_ is selected, _"Wi-Fi"_ must be selected, and vice versa.


### 2.5 Language levels
In UVL, language levels serve several purposes, including separating the expressiveness of the language into different layers or levels, allowing users and tools to decide how much complexity they want to support, and even extending the basic core of UVL without complicating it for users who only need simple models.

If a tool or user wants a simple UVL, it can work only at the Boolean level. In the case of needing more expressiveness (numbers, text, operations), it can activate higher levels.

There are three hierarchical levels:
- **Boolean Level**: only Boolean features and logical constraints.
- **Arithmetic Level**: numeric attributes, arithmetic operations.
- **Type Level**: text or number attributes, operations on types.

Each of these levels inherently and completely includes the previous one. In addition, there are minor levels, which are optional extensions that add extra capabilities but are not mandatory to pass from one level to another.

#### Boolean level
This is the most basic major level. It therefore represents the core of the language, since it is the basis for the feature tree, the dependencies (mandatory, optional, alternative, or), the basic logical constraints between features `!`, `&`, `|`, `=>`, `<=>`, and the _Group Cardinality_ (minor level).
For example:
```
features
    Smartwatch
        mandatory
            Display
                alternative
                    LCD
                    OLED
            Connectivity
                or
                    Bluetooth
                    Wi-Fi
        optional
            GPS
            "Heart Rate Monitor"
            "Music Player"
            "Water Resistance"
                alternative
                    IP67
                    IP68
            Accessories
                [1..2]
                    "Leather Strap"
                    "Metal Strap"
                    "Silicone Strap"

constraints
    Wi-Fi => Bluetooth
    "Water Resistance" => Display
```
where there is only presence or absence of features.

##### Group Cardinality
This is a _minor level_ of the _boolean level_. It allows to select between `n..m` elements of a group. In the example above, it defines that a maximum of 2 _Complement_ can be chosen from those described. In order to use it, it must be included in the file, as you can see at the beginning of the example.


#### Arithmetic level
This major level includes the Boolean level. It allows to define numeric attributes on features, to make arithmetic constraints on attributes using `+`, `-`, `*`, `/`, `=`, `<`, `>`, `<=`, `>=`, and to be able to use the _minor level_ _Aggregate Functions_ and _Feature Cardinality_.
For example:
```
includes
    Arithmetic.feature-cardinality
    Arithmetic.aggregate-function

features
    Smartwatch
        mandatory
            Display
                alternative
                    LCD
                    OLED
            Connectivity
                or
                    Bluetooth
                    Wi-Fi
            Battery
                alternative
                    Standard {Capacity 300}
                    "Long Life" {Capacity 450}
        optional
            "Music Player" {Storage 2}
            "Watch Faces" cardinality [5..20]

constraints
    Wi-Fi => Bluetooth
    sum(Capacity) >= 400
```
where the sum of battery capacity and motor power must be greater than 150.


##### Feature cardinality
This minor level sets numerical constraints for attribute values. To use it, it must be included with `Arithmetic.feature-cardinality`. In the case of the example above, only from 4 to 7 _Seat_ can be selected.

##### Aggregate functions
This minor level includes the `sum()` and `avg()` functions, which can be used in the `constraints` block.
For example:
```
include
    Arithmetic.aggregate-function
features
    ...

constraints
    sum(PriceEuros) > 200
    avg(Calification) > 5
```
where we have a _PriceEuros_ attribute whose total sum must be less than 200, and a _Calification_ attribute, whose average must be greater than 5.

#### Type level
This major level includes Boolean and Arithmetic level. It introduces type attributes: `Integer`, `Float`, `String`. Thanks to this, values can be asociated to features, and therefore place restrictions on them. This level also includes the string `len()` function, which belongs to the `string-constraints` minor level. With respect to numerical types, constraints can be placed and values can be compared.

For instance:
```
includes
    Type.string-constraints

features
    Smartwatch
        mandatory
            String Color
            Float Weight
            String "Serial Number"

constraints
    len("Serial Number") == 10
    Weight > 30.0
    Color != "Pink"
```
where we have String and Float features, and the first constraint in the _constraints_ block sets that the _"Serial Number"_ feature must include exactly 10 characters.

---

## 3. What is UVLHub?

UVLhub is a complete platform for the management of variability models in UVL format. It acts as a centralized repository to efficiently store, edit, share and analyze feature models. 
UVLhub integrates with Zenodo to provide robust data storage and with Flamapy to provide advanced model analysis capabilities. The platform is designed to support researchers and developers under Open Science principles, promoting collaboration and transparency in working with variability models. UVLhub also offers tools such as Rosemary CLI for advanced management of your environment, support for automated testing (unit, load, GUI), and flexible deployment options, including Docker, Vagrant and Render. 
In short, UVLhub not only centralizes the management of UVL models, but also facilitates their analysis, publication and collaboration, helping to drive feature model innovation.

You can access the full documentation of UVLHub here: https://docs.uvlhub.io/
Plus, you can access to UVLHub here: https://www.uvlhub.io/
Here is the repository of UVLHub: https://github.com/diverso-lab/uvlhub
For further information, check the research paper (by David Romero-Organvidez, José A. Galindo, Chico Sundermann, Jose-Miguel Horcas, David Benavides): https://www.sciencedirect.com/science/article/pii/S016412122400195X?via%3Dihub

---

## 4. What is Flamapy?

[**Flamapy**](https://docs.flamapy.org/) is an advanced Python-based tool for automated analysis of Feature Models, such as those in UVL. It revolutionizes variability model analysis by integrating the capabilities of multiple solvers and metamodels. Flamapy is highly extensible thanks to its plugin generation system, which facilitates the creation and customization of new extensions, and supports variability modeling in real environments with flexibility for attributes and cardinalities. 

In addition, it offers robust solver support by integrating technologies such as PySAT, which gives access to more than ten different solvers, and BDD, which provides efficient analysis using binary decision diagrams. Flamapy is designed to be easy to use and integrate, with a very simple Python interface, straightforward command line, support for browser execution via WASM, and availability of a REST API for advanced integrations.

You can check the full documentation here: https://docs.flamapy.org

Flamapy has his own integrated development environment which you can start creating your own feature models. It implements a set of functionalities that facilitates your work, such as:

- Editing of feature models in various formats (such as UVL, SPLX, etc.).
- Automatic model validation: detect errors or inconsistencies in the model structure.
- Configuration analysis: you can create and validate specific model configurations.
- Model transformations and operations: such as converting between formats or projecting subsets of the model.
- Analysis solutions such as counting possible configurations, listing valid configurations, or detecting dead features (features that can never be selected).
- Lightweight, browser-accessible graphical interface, with no need to install software.

You can access the FlamapyIDE here: https://ide.flamapy.org/