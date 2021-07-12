---
title: >
  Mapping between YANG and SDF
abbrev: Mapping between YANG and SDF
docname: draft-kiesewalter-asdf-yang-sdf-latest
date: 2021-07-08

stand_alone: true

ipr: trust200902
keyword: Internet-Draft
cat: info

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  - name: Jana Kiesewalter
    org: Universität Bremen
    email: jankie@uni-bremen.de
  - name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org
    role: editor

normative:
  I-D.ietf-asdf-sdf: sdf
  RFC7950: yang
informative:
  SDF-YANG-CONVERTER:
    -: impl
    target: sdf-yang-converter.org
    title: SDF YANG converter playground
    author:
      name: Jana Kiesewalter
  SDF-YANG-CONVERTER-IMPL:
    -: implsrc
    target: https://github.com/jkiesewalter/sdf-yang-converter
    title: SDF YANG converter
    author:
      name: Jana Kiesewalter
  LIBYANG:
    target: https://github.com/CESNET/libyang
    title: libyang
    date: false
    author:
     - name: Michal Vasko
       org: CESNET
     - name: David Sedlák
     - org: more contributors

--- abstract

<!-- (briefly explain what we are trying to do, e.g.:) -->

YANG and SDF are two languages for modelling the interaction with and
the data interchanged with devices in the network.
As their areas of application (network management, IoT, resp.)
overlap, it is useful to be able to translate between the two.

The present specification provides information about how models in one
of the two languages can be translated into the other.
This specification is not intended to be normative, but to help with
creating translators.


--- middle

Introduction        {#intro}
============

(See Abstract for now, add references {{-yang}} and {{-sdf}}.)


Pairing SDF and YANG features
======================

{{yang-to-sdf}} gives an overview over how language features of YANG
can be mapped to SDF features. In many cases, several translations
are possible, and the right choice depends on the context. The mappings
in this draft often accommodate the use of the YANG parser Libyang
{{LIBYANG}}.

For YANG statements that are not mentioned in the table no conversion to SDF
was found that preserves the statement's semantics. For possible
conversions of YANG's built-in types please refer to {{yangtosdf}}.
Please note that a 'type object' is not the same as an
sdfObject but refers to SDF's built-in type 'object', also called
compound-type. This built-in type makes use of the 'properties'
quality which is not to be confused with the sdfProperty class.
The data types number/decimal64, integer, boolean, string are also referred to
as simple (data) types. In turn, the types array and object are sometimes
referred to as complex (data) types. Concerning YANG, the expression
'schema tree' refers to the model's tree whereas 'data tree' describes
the tree of an instance of the model.

| YANG statement  | remark on YANG statement                | converted to SDF                                                                                                                                              |
|-----------------|-----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| module          |                                         | SDF model (i.e., info block, namespace section & definitions)                                                                                                  |
| submodule       | included in supermodule                 | integrated into SDF model of supermodule                                                                                                                      |
|                 | on its own                              | SDF model                                                                                                                                                     |
| container       | top-level                               | sdfObject                                                                                                                                                     |
|                 | one level below top-level               | sdfProperty of type object\* (compound-type)                                                                                                                  |
|                 | on any other level                      | property\* (type object\*) of the 'parent' definition of type object\* (compound-type)                                                                        |
| leaf            | on top-level and one level below        | sdfProperty (type integer/number/boolean/string)                                                                                                              |
|                 | on any other level                      | property\* (type integer/number/boolean/string) of the 'parent' definition of type object\* (compound-type)                                                    |
| leaflist        | on top-level and one level below        | sdfProperty of type array                                                                                                                                     |
|                 | on any other level                      | property\* (type array) of the 'parent' definition of type object\* (compound-type)                                                                           |
| list            | on top-level and one level below        | sdfProperty of type array with items of type object\* (compound-type)                                                                                         |
|                 | on any other level                      | property\* (type array with items of type object\* (compound-type)) of the 'parent' definition of type object* (compound-type)                                |
| choice          |                                         | sdfChoice                                                                                                                                                     |
| case            | belonging to choice                     | element of the sdfChoice                                                                                                                                      |
| grouping        |                                         | sdfData of compound-type (type object\*) at the top level which can then be referenced                                                                        |
| uses            | referencing a grouping                  | sdfRef to the SDF definition corresponding to the referenced grouping                                                                                         |
| rpc             |                                         | sdfAction at the top-level of the SDF model                                                                                                                   |
| action          |                                         | sdfAction of the sdfObject corresponding to a container the action is a descendant node to                                                                    |
| notification    |                                         | sdfEvent                                                                                                                                                      |
| anydata         |                                         | not converted                                                                                                                                                 |
| anyxml          |                                         | not converted                                                                                                                                                 |
| augment         |                                         | augment's target is converted with the augmentation already applied, mentioned in the description                                                             |
| type            | referring to a built-in type            | type with other data qualities (e.g., default) if necessary                                                                                                    |
| type            | referring to a typedef                  | sdfRef to the corresponding sdfData element                                                                                                                   |
| base            |                                         | sdfRef to the sdfData definition corresponding the the base                                                                                                   |
| bit             |                                         | 'parent' definition is of compound-type and gets one entry in the properties quality of type boolean for each bit                                             |
| enum            |                                         | each enum statement's argument is added as an element to the SDF enum quality's string array                                                                  |
| fraction-digits |                                         | multipleOf quality                                                                                                                                            |
| length          | single length range                     | minLength/maxLength qualities                                                                                                                                 |
|                 | single value                            | minLength and maxLength qualities set to the same value                                                                                                       |
|                 | contains alternatives                   | sdfChoice with alternatives for minLength/maxLength qualities                                                                                                 |
| path            |                                         | sdfRef to the corresponding SDF definition                                                                                                                    |
| pattern         | single pattern                          | pattern quality                                                                                                                                               |
|                 | multiple patterns                       | pattern quality, the regular expressions are combined using positive lookahead                                                                                |
|                 | invert-match                            | pattern quality, the regular expression is modified using negative lookahead                                                                                  |
| range           | single range                            | minimum/maximum qualities                                                                                                                                     |
|                 | single value (constant)                 | const quality                                                                                                                                                 |
|                 | contains alternatives                   | sdfChoice with either minimum/maximum or const quality as alternatives                                                                                        |
| typedef         |                                         | sdfData definition, sdfRef where it is used                                                                                                                   |
| identity        |                                         | sdfData definition, sdfRef where it is used                                                                                                                   |
| config          | of a container that became an sdfObject | set writable for all elements in the sdfObject that can be marked as writable (i.e., that use the data qualities)                                              |
|                 | of any other YANG element               | set writable                                                                                                                                                  |
| import          |                                         | the module that the import references is converted (elements can now be referenced by sdfRef) and its prefix and namespace are added the to namespace section |
| revisions       |                                         | first revision date becomes version in information block                                                                                                      |
| namespace       |                                         | added to namespace section                                                                                                                                    |
| prefix          |                                         | added to namespace section                                                                                                                                    |
{: #yang-to-sdf title="Mapping YANG to SDF"}


{{sdf-to-yang}} provides the inverse mapping.


| SDF quality             | remark on SDF quality                                                           | converted to YANG                                                                                                                                                                                           |
|-------------------------|---------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sdfThing                |                                                                                 | container node                                                                                                                                                                                              |
| sdfObject               |                                                                                 | container node                                                                                                                                                                                              |
| sdfProperty             | type integer/number/boolean/string                                              | leaf node                                                                                                                                                                                                   |
|                         | type array with items of type integer/number/boolean/string                     | leaf-list node                                                                                                                                                                                              |
|                         | type array with items of type object (compound-type)                            | list node                                                                                                                                                                                                   |
|                         | type object (compound-type)                                                     | container node                                                                                                                                                                                              |
| sdfAction               | at the top-level, __not__ part of an sdfObject                                  | rpc node                                                                                                                                                                                                    |
|                         | inside of an sdfObject                                                          | action node as child node to the container corresponding to the sdfObject                                                                                                                                   |
| sdfEvent                |                                                                                 | notification node with child nodes that were translated like sdfProperty                                                                                                                                    |
| sdfData                 | type integer/number/boolean/string                                              | typedef                                                                                                                                                                                                     |
|                         | type array with items of type integer/number/boolean/string                     | grouping node with leaf-list child node                                                                                                                                                                     |
|                         | type array with items of type object (compound-type)                            | grouping node with list child node                                                                                                                                                                          |
|                         | type object (compound-type)                                                     | grouping node                                                                                                                                                                                               |
| sdfRef                  | referenced definition was converted to typedef                                  | type is set to the typedef corresponding to the sdfData definition                                                                                                                                          |
|                         | referenced definition was converted to leaf or leaf-list node                   | leafref                                                                                                                                                                                                     |
|                         | referenced definition was converted to grouping node                            | uses node that references corresponding grouping (and refine if necessary)                                                                                                                                  |
| sdfRequired             | referenced definition was converted to a leaf or choice node                    | set the mandatory statement of the corresponding leaf/choice node to true                                                                                                                                   |
|                         |                                                                                 | find the first descendant node that is either a leaf/choice node and set their mandatory statement to true or that is a leaf-list/list node and set their min-elements statement to 1 (if not already >= 0) |
| sdfChoice               |                                                                                 | choice node with one case node for each alternative of the sdfChoice, each alternative is converted like sdfProperty                                                                                        |
| type                    |                                                                                 |                                                                                                                                                                                                             |
| const                   | corresponding YANG element has empty range                                      | range statement with a single value                                                                                                                                                                         |
|                         | range not empty                                                                 | add single value alternative to range statement (must be disjunct)                                                                                                                                          |
| default                 | type is one of integer/number/boolean/string or array with items of these types | default statement of leaf/leaf-list nodes                                                                                                                                                                   |
| minimum/maximum         | corresponding YANG element has empty range                                      | range statement                                                                                                                                                                                             |
|                         | range not empty                                                                 | add range alternative to range statement (must be disjunct)                                                                                                                                                 |
| multipleOf              |                                                                                 | fraction-digits statement                                                                                                                                                                                   |
| minLength/maxLength     |                                                                                 | length statement                                                                                                                                                                                            |
| pattern                 |                                                                                 | pattern statement                                                                                                                                                                                           |
| minItems/maxItems       |                                                                                 | min-elements/max-elements statements                                                                                                                                                                        |
| uniqueItems set to true | if the 'parent' SDF definition is converted to a list node                      | unique statement mentioning all of the leaf/leaf-list nodes in the list node's sub-tree                                                                                                                     |
| items                   |                                                                                 | sub-statements of list/leaf-list node corresponding to the item quality's 'parent' definition                                                                                                               |
| properties              |                                                                                 | child nodes of container/grouping node corresponding to the properties quality's 'parent' definition                                                                                                        |
| unit                    |                                                                                 | units statement                                                                                                                                                                                             |
| enum                    |                                                                                 | type enumeration with enum statements for each string in the SDF enum quality                                                                                                                               |
| sdfType                 | has value 'byte-string'                                                         | built-in type 'binary'                                                                                                                                                                                      |
| writable                |                                                                                 | config statement                                                                                                                                                                                            |
{: #sdf-to-yang title="Mapping SDF to YANG"}


Mapping from YANG to SDF        {#yangtosdf}
========================
This section specifies one possible mapping for each of the YANG statements to SDF in detail. For reference on the individual YANG statements see {{-yang}} and {{-sdf}} for SDF.

Module                 {#design-module}
------

* YANG: {{Section 7.1 (module) of -yang}}
* SDF:
  * {{Section 3.1 (information block) of -sdf}}
  * {{Sections 3.2 and 4 (namespaces section) of -sdf}}
  
After conversion the SDF model as a whole corresponds to the YANG module. The argument of the `namespace` statement of the YANG module is added to the SDF `namespace section` together with the argument of the YANG module's `prefix` statement which also becomes the `default namespace` in the SDF model. Additionally, the namespaces and prefixes of each of the modules mentioned in the `import` statements are added to the namespace of the SDF model. Libyang loads the imported modules automatically and in the correct revision. These modules are then also converted and stored so their definitions can be referenced via the `sdfRef` common quality when necessary.

The contents of the `organization`, `contact` and `yang-version` statements are stored alongside the actual `description` of the YANG module in a special sdfData definition designated to hold information on the module that does not fit into the SDF information block. This is done in the way described in {{design-roundtrips}} to facilitate round trips in the future. The module's description is scanned for information regarding copyright and licensing which are then transferred to the `copyright` and `license` qualities of the SDF model's information block. The `version` quality of the SDF model's information block is set to the first revision date given in the YANG module's `revision` statement. All other revision dates are ignored as of now.

YANG modules can define features via the `feature` statement to make parts of the module conditional. The abilities of a server are checked against the module's features. Nodes reference features as an argument to the `if-feature` statement. If a server does not support a certain feature nodes that reference that feature are ignored by the server. Since this functionality cannot be represented in SDF yet YANG features are stored in the description of the sdfData definition designated to hold information on the module. The note that is added to the descriptions looks as described in {{design-roundtrips}}.

If the `deviation` statement (introducing a deviation from the original YANG module) is present in the YANG module Libyang applies the deviation directly and the converter converts the module that way. The presence of the deviation in the original YANG module is not indicated in the resulting SDF model as of now which might cause inconsistencies after round trips.

Submodule
---------

* YANG: {{Section 7.2 (submodule) of -yang}}
  
A sub-module that is included into its super-module via the `include` statement is integrated into the super-module and converted that way. This is the simplest option due to the way Libyang represents included sub-modules. A sub-module that occurs without its super-module is converted to its own SDF model as described in {{design-module}}.

Container Statement
-------------------

* YANG: {{Section 7.5 (container) of -yang}}
* SDF:
  * {{Sections 2.2.1 and 5.1 (sdfObject) of -sdf}}
  * {{Sections 2.2.6 and 6.3 (sdfThing) of -sdf}}

YANG uses container nodes to group together other nodes. Containers on the top-level of a module are converted to sdfObjects. A container that is a direct child node to a top-level container node is converted to a compound-type sdfProperty inside of said sdfObject. Any other container becomes a property of the compound-type definition corresponding to the container's parent node. Since the first SDF draft did not contain the compound-type as a possible argument to the type quality containers used to be translated to sdfThings. This, however, was not a very fitting conversion semantically. At that time, sdfThings were the only elements that could contain elements of the same class, i.e., sdfThings could contain other sdfThings. This ability is required to represent the tree structure of YANG where e.g., containers can contain other containers. In the next SDF draft the compound-type was introduced. This feature effectively makes it possible for elements of the sdfData and sdfProperty classes to contain elements that share the same qualities.
A sub-statement to the container statement that cannot be represented in SDF as of now is the optional `presence` statement. The argument of the presence statement assigns a meaning to the presence or absence of a container node in an instance of the module. This concept is expressed in the description of the SDF definition analogue to the container node as shown in {{design-roundtrips}}.

Leaf Statement         {#design-leaf}
--------------

* YANG: {{Section 7.6 (leaf) of -yang}}
* SDF:
  * {{Sections 2.2.2 and 5.2 (sdfProperty) of -sdf}}
  * {{Section 4.7 (data qualities) of -sdf}}
  
Leaf nodes in YANG represent scalar variables. If a leaf node occurs at the top-level of the module or as a direct child node of a top-level container (which is converted to sdfObject) it is transformed to an sdfProperty. On any other level a leaf becomes a property of the compound-type definition equivalent to the leaf's parent node. In both cases the SDF `type` data quality is set to one of the simple data types because leaf nodes can only be of simple data types. Leaf nodes can be assigned default values which are used in case the leaf node does not exist in an instance of the YANG module. A leaf's default value is converted to SDF as the data quality `default`. The `units` sub-statement of a leaf node in YANG becomes the SDF data quality `unit`. This quality is is constrained to the SenML unit names. Although it could cause conformance issues, the content of the YANG units statement is not processed to fit the SenML unit names as of now. This is due to the low probability that a unit from a YANG module is not listed in the SenML unit names in comparison to the time required to implement a mechanism to check conformance and convert non-conforming units. This feature might be added in later versions of the converter, though. YANG leaf nodes can be marked as mandatory to occur in an instance of the module by the `mandatory` statement. The statement takes `true` and `false` as arguments. This can easily be mapped to SDF through the sdfRequired common quality. A reference to the SDF definition equivalent to the YANG leaf node marked as mandatory is added to the containing sdfObject's sdfRequired quality. If the sdfRequired quality does not already exist in the sdfObject it is added now.

Leaf-List Statement    {#sec-map-leaflist}
-------------------

* YANG: {{Section 7.7 (leaf-list) of -yang}}
* SDF:
  * {{Sections 2.2.2 and 5.2 (sdfProperty) of -sdf}}
  * {{Section 4.7 (data qualities) of -sdf}}
  
Similarly to leaf nodes, leaf-list nodes hold data of simple types in YANG but as items in an array. As such, leaf-list definitions are converted to sdfProperty if they occur on the top-level or one level below in a module. On any other level a leaf-list becomes a property of the compound-type definition corresponding to the leaf-list's parent definition. In both cases the type is set to `array`. The items of the array are of simple data types since leaf-list definitions can only have simple data types as well. The minimal and maximal number of elements in a YANG leaf-list can be specified by the `min-elements` and `max-elements` sub-statements. This is analogue to SDF's `minItems` and `maxItems` data qualities which are set accordingly by the converter. A YANG leaf-list can specify whether the system or the user is responsible for ordering the leaf-lists entries. This information is stored in the `ordered-by` statement in YANG which is represented in SDF by a remark in the description (as shown in {{design-roundtrips}}) of the SDF equivalent to the leaf-list node in question. Since leaf-list nodes are just leaf nodes that can occur multiple times the `units` and `default` statements of leaf-list nodes are converted as described in {{design-leaf}}.

List Statement
--------------

* YANG: {{Section 7.8 (list) of -yang}}
* SDF:
  * {{Sections 2.2.2 and 5.2 (sdfProperty) of -sdf}}
  * {{Section 4.7 (data qualities) of -sdf}}

Since list nodes are similar to leaf-list nodes with the difference that they represent a an assortment of nodes that can occur multiple times they are also converted similarly. List nodes one the top-level or one level below become sdfProperties. On any other level a list node is converted to a property of the compound-type definition corresponding to the list's parent node. The type is set to `array` for both alternatives. Since lists contain a set of nodes the items of the corresponding array are of type object.  The minimal and maximal number of elements in a YANG list can be specified by the `min-elements` and `max-elements` sub-statements. This is analogue to SDF's `minItems` and `maxItems` data qualities which are set accordingly by the converter. List nodes in YANG can define one or multiple keys that identify the list entries via the `key` statement. There is no SDF quality representing this feature. To preserve the information the list keys are stored in the description of the SDF definition analogue to the YANG list node as described in {{design-roundtrips}}.
The YANG list's `unique` sub-statement defines a list of descendant leaf nodes of the list that must have a unique combination of values each. This concept is comparable SDF's `uniqeItems` data quality. However, the boolean-typed uniqueItems quality specifies whether all items of an SDF array have to be unique as opposed to only a selection of unique items in the YANG statement unique. To mitigate this discrepancy a note is added to the SDF equivalents of all descendant leaf nodes of a list that are marked as unique as shown in {{design-roundtrips}}.
Since list nodes are similar to leaf-list nodes the ``ordered-by`` statement of a list node is converted as described in {{sec-map-leaflist}}.

Grouping Statement
------------------

* YANG: {{Section 7.12 (grouping) of -yang}}
* SDF: {{Section 5.5 (sdfData) of -sdf}}
  
Grouping nodes are very similar to container nodes with the difference that the set of nodes defined in a grouping do not occur in the data tree unless the grouping has been referenced one or more times by uses nodes. Thus, a grouping node is converted to a compound-type sdfData definition which also defines a reusable definition that is not a declaration.

Uses Statement
--------------

* YANG: {{Section 7.13 (uses) of -yang}}
* SDF: {{Section 4.4 (sdfRef) of -sdf}}
  
A uses node has the purpose of referencing a grouping node. The set of child nodes of the referenced grouping are copied to wherever the uses node is featured. Some of the referenced grouping's sub-statements can be altered via the refine statement of the uses node. In SDF a uses node is represented by the sdfRef quality which is added to the definition corresponding to the parent node of the uses node. As an argument the sdfRef contains a reference to the sdfData definition corresponding to the grouping referenced by the uses node. If the uses node contains a refine statement its contents are converted as they would be if they occurred in a node.

Choice Statement
----------------

* YANG: {{Section 7.9 (choice) of -yang}}
* SDF: {{Section 4.7.2 (sdfChoice) of -sdf}}
  
Conversion of the choice definitions from YANG is quite simple since it is similar to the sdfChoice quality. A choice definition is converted to an sdfChoice definition. The case definitions or other child definitions of the choice become one of the named alternatives of the resulting sdfChoice each.

RPC Statement
-------------

* YANG: {{Section 7.14 (rpc) of -yang}}
* SDF: {{Sections 2.2.3 and 5.3 (sdfAction) of -sdf}}
  
Remote procedure calls (RPCs) can be modelled in YANG with RPC nodes which have up to one `input` child node holding the commands input data and up to one `output` node for the output data. In YANG RPCs can only occur on the top-level because in contrast to actions in YANG they do not belong to a container. This can easily be represented by sdfActions. The corresponding sdfAction is not placed inside an sdfObject or sdfThing but at the top-level of the SDF model to represent independence from a container. The input node of the RPC is converted to the sdfInputData quality of the sdfAction which is of type object. Equivalently the output node of the RPC becomes the sdfAction's sdfOutputData which is also of type object. Groupings and typedefs in the RPC are converted to sdfData definitions inside the sdfAction.

Action Statement
----------------

* YANG: {{Section 7.15 (action) of -yang}}
* SDF: {{Sections 2.2.3 and 5.3 (sdfAction) of -sdf}}

Action nodes in YANG work similarly to RPC nodes in the way that they are used to model operations that can be invoked in the module and also have up to one input and output child node respectively. As mentioned before YANG actions are affiliated to a container though. The representation of this affiliation is not quite trivial because YANG containers are not translated to sdfObjects in all cases. Only sdfObjects can have sdfActions, though. If an action occurs in a container that is a below-top-level container (and thus not converted to sdfObject) the affiliation cannot be represented directly in SDF as of now. To keep the semantics of the affiliation a copy of the contents of the converted container is added to the sdfAction's sdfInputData. Like for RPC nodes, the input nodes of the action are converted to the sdfInputData quality of the sdfAction which is of type object. Equivalently the output nodes of the action become the sdfAction's sdfOutputData which is also of type object. Groupings and typedefs in the action node are converted to sdfData definitions inside the sdfAction.

Notification Statement
-----------------------

* YANG: {{Section 7.16 (notification) of -yang}}
* SDF: {{Sections 2.2.4 and 5.4 (sdfEvent) of -sdf}}

In YANG, notification nodes are used to model notification messages. Notification nodes are converted to sdfEvent definitions. Their child nodes are converted to the sdfEvent's sdfOutputData which is of type object. Groupings and typedefs in the notification node are converted to sdfData definitions inside the sdfEvent.

Augment Statement
-----------------

* YANG: {{Section 7.17 (augment) of -yang}}
* SDF: {{Section 4.6. (common qualities) of -sdf}}

The augment statement can either occur at the top-level of a module to add nodes to an existing target module or sub-module or in a uses statement to augment the targeted grouping. The conversion of the augment statement to SDF is not trivial because SDF does not feature this mechanism directly. Since the tool used to deserialize YANG modules (libyang) adds the nodes into the augment statement's target automatically this is adopted for conversion. The SDF model or sdfData definition that corresponds to the augment statement's target is converted with the augmentation already applied. A comment is added to the description as described in {{design-roundtrips}} to preserve where the augmentation was made from. If the resulting SDF model has to be converted back to YANG definitions that are marked as augmentations are converted back accordingly.

Anydata and Anyxml Statements
-----------------------------

* YANG: {{Sections 7.10 and 7.11 (augment) of -yang}}
* SDF: {{Section 4.6 (common qualities) of -sdf}}

The `anydata` and `anyxml` statements are designated for nodes in the schema tree whose structure is unknown at the module's design time or in general. Since this is not a concept that cannot be represented in SDF as of now, anydata and anyxml nodes are note converted. To preserve the information (e.g., for round trips) a comment is added to the SDF element corresponding to the anydata/anyxml node's parent node as described in {{design-roundtrips}}.

Type Statement
--------------

* YANG: {{Section 7.4 (type) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

Conversion for the `type` statement from YANG is very straightforward if the argument is a simple data type since the SDF data qualities also contain a `type` quality. A derived type used as an argument to the YANG `type` statement is converted to an `sdfRef` to the `sdfData` corresponding to that derived type. If the derived type is restricted (e.g., via the length statement) the restrictions are converted as they would be for the base type and added to the SDF definition containing the type in question.

There are multiple sub-statements to the type statement that depend on its value.

String Built-In Type {#design-string}
--------------------

* YANG: {{Section 9.4 (string) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

The YANG built-in type string is converted to SDF's built-in type string. Strings in YANG can be restricted regarding their length and patterns (containing regular expressions).

The length statement can specify either a constant length, a lower inclusive length, an upper inclusive length or both a lower and upper inclusive length. A length statement can also specify more than one disjoint constant length or length ranges. The values `min` and `max` in a length statement represent the minimum and maximum lengths accepted for strings. If the length statement in YANG does not contain a constant value but a length range it is converted to the `minLength` and `maxLength` data qualities in SDF. If a constant value is defined through the YANG length statement the `minLength` and `maxLength` qualities are set to the same value. If the length statement specifies multiple length ranges or constant values the sdfChoice quality is used for conversion. The named alternatives of the sdfChoice contain the single converted length ranges or constant values each. If the `min` and `max` values are present in the YANG length statement they are converted to the respective minimum and maximum lengths accepted for strings.

To represent YANG string patterns the `pattern` data quality of SDF can be used. One problem in the conversion of patterns is that YANG strings can be restricted by multiple patterns but SDF definitions of type string can have at most one pattern. To represent multiple patterns from YANG in SDF the patterns are combined into one regular expression with the help of positive look-ahead. This, however, does not always convey the meaning of the original regular expression. Another issue is the possibility to declare invert-match patterns in YANG. These types of patterns are converted to SDF by adding negative look-ahead to the regular expression. To preserve the original patterns and to facilitate round trips the original patterns are stored in the description of the containing definition as described in {{design-roundtrips}}.

Another, more general problem regarding the conversion of regular expressions from YANG to SDF is the fact that YANG uses a regular expression language as defined by W3C Schema while SDF adopts the one from JSON Schema. Both regular expression languages share most of their features but differ in some details. Since this does not cause problems in most cases and regarding the time constraints of this thesis, this issue is not given any further attention beyond what was stated in this paragraph. There is, however, a project of the IETF Network Working Group to create an interoperable regular expression format I-Regexp. Once the work on the draft has progressed the format might be adopted by the converter.

Decimal64 Built-In Type {#design-dec64}
-----------------------

* YANG: {{Section 9.3 (decimal64) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

The decimal64 built-in type of YANG is converted to the number type in SDF. A decimal64 type in YANG has a mandatory `fraction-digits` sub-statement that specifies the possible number of digits after the decimal separator. The value of the fraction-digits statement is converted to the `multipleOf` data quality of SDF which states the resolution of a number, i.e., the size of the minimal distance between number values.

A YANG decimal64 type can be restricted by means of the range statement specifying either a constant value, a lower inclusive bound, an upper inclusive bound or both a lower and upper inclusive value. A range statement can also specify more than one disjoint constant values or ranges. The values `min` and `max` in a range represent the minimum and maximum values of the type in question. If the range statement in YANG contains a range and not a constant value it is converted to the `minimum` and `maximum` data qualities in SDF. If a constant value is defined through the YANG range the SDF `const` data quality is set accordingly. If the range specifies multiple ranges or constant values the sdfChoice quality is used for conversion. The named alternatives of the sdfChoice contain the single converted ranges or constant values each. If the `min` and `max` values are present in the YANG range they are converted to the respective minimum and maximum values for the type in question.

Integer Built-In Types
----------------------

* YANG: {{Section 9.2 (integer) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}


In YANG there are 8 different integer types (int8, uint8, int16, uint16, int32, uint32, int64, uint64). Each of them is converted to integer in SDF. A comment specifying the exact type is added as described in {{design-roundtrips}}. Additionally, the `minimum` and `maximum` qualities of the SDF definition the converted type belongs to are set to the respective minimum and maximum values of the integer type in question. If the YANG type also specifies a range the minimum and maximum SDF qualities are altered accordingly. Like the decimal64 YANG built-in type integer types can also be restricted by a range statement. This range statement is converted as described in {{design-dec64}}.

Boolean Built-In Type
---------------------

* YANG: {{Section 9.5 (boolean) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

YANG's boolean built-in type is converted to SDF's boolean type. There are no further sub-statements to this type in YANG.

Binary Built-In Type
--------------------

* YANG: {{Section 9.8 (binary) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

If the argument of the YANG type statement is `binary` the SDF type quality is set to string. In addition the sdfType quality is set to `byte-string`. A YANG binary can have a sub-statement restricting its length. This is converted to SDF via the `minLength` and `maxLength` data qualities. Like the string YANG built-in type binary types can also be restricted by a length statement. This length statement is converted as described in {{design-string}}

Enumeration Built-In Type
-------------------------

* YANG: {{Section 9.6 (enumeration) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

The YANG built-in type `enumeration` is used to map string-valued alternatives to integer values. Additionally each string can have a description and other sub-statements. SDF also specifies an `enum` quality which is used to represent YANG enumerations. Since the SDF enum quality only holds an array of strings all other information is stored in the description of the SDF definition the enum belongs to.

Bits Built-In Type
------------------

* YANG: {{Section 9.8 (bits) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

Since SDF does not specify a built-in type to represent a set of named bits and their positions like YANG does this YANG built-in type has to be converted to SDF type object with one property of type boolean for each bit. The property is named after the bit's name and the bit's position is stored in the property's description as described in {{design-roundtrips}}.

Union Built-In Type
-------------------

* YANG: {{Section 9.12 (union) of -yang}}
* SDF: {{Section 4.7.2 (sdfChoice) of -sdf}}

Although the `union` built-in type of YANG does not exist as a built-in type in SDF its meaning can be easily represented by the sdfChoice quality. YANG unions hold a set of alternative types. The corresponding sdfChoice contains a set of named alternatives each containing only the SDF type quality and named after the respective type in the YANG union.

Leafref and Identityref Built-In Types
--------------------------------------

* YANG: {{Section 9.9 (leafref) of -yang}} {{Section 9.10 (identityref) of -yang}}
* SDF: {{Section 4.4 (sdfRef) of -sdf}}

YANG's built-in types `leafref` and `identityref` are used to reference a leaf node or identity definition respectively. They are represented in SDF by the sdfRef quality. As an argument said sdfRef quality contains a reference to the SDF element corresponding to the target of the leafref or identityref statement.

Empty Built-In Type
-------------------

* YANG: {{Section 9.11 (empty) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

Another concept that is not contained in SDF directly is that of YANG's built-in type `empty`. YANG elements with this type convey meaning by their mere existence or non-existence. This is represented in SDF using the compound-type with an empty set of properties.

Instance-Identifier Built-In Type
---------------------------------

* YANG: {{Section 9.13 (instance-identifier) of -yang}}

The `instance-indentifier` built-in type of YANG cannot be represented in SDF as of now since there is currently no possibility to specify SDF instances. This feature might be added to SDF in the future, though
.

Derived Type (Typedef) Statement
--------------------------------

* YANG: {{Section 9.3 (typedef) of -yang}}
* SDF: {{Section 4.4 (sdfRef) of -sdf}}

The SDF class `sdfData` is used to represent YANG `typedefs` after conversion. The usage of a derived type via the `type` statement is converted to an `sdfRef` to the corresponding sdfData definition. If a derived type is restricted according to its base type (e.g., with a range statement) the restrictions are converted as they would be for the base type and added to the sdfData definition.

Identity Statement
------------------

* YANG: {{Section 7.18 (identity) of -yang}}
* SDF: {{Section 5.5 (sdfData) of -sdf}}
  
The YANG identity statement is used to denote the name and existence of an identity. Identities can be based on one ore more other identities. They are referenced with the `identityref` statement. This concept is converted to SDF by sdfData definitions for each identity. If an identity is based on one other identity this is represented in by an sdfRef to the sdfData element corresponding to the base identity. If an identity has multiple base identities it is converted to a compound-type sdfData definition with one property for each base identity. Each property contains an sdfRef to the sdfData element corresponding to one of the base identities.

Config Statement
----------------

* YANG: {{Section 7.21.1 (config) of -yang}}
* SDF: {{Section 4.7 (data qualities) of -sdf}}

The config statement of YANG can have the boolean values `true` or `false` as arguments. If config is set to true the element containing the config statement represents readable and writable configuration data. If config is set to false he element containing the config statement represents read-only state data. This is transferred to SDF via the `readable` and `writable` data qualities. YANG config with the argument true is converted to readable and writable being set to true, config with the argument false is converted as readable set to true and writable set to false. There are, however, cases in which the SDF definition corresponding to the YANG element containing the config statement is not one that can use data qualities (i.e., is not sdfData or sdfProperty). This is the case if a top-level container which is converted to sdfObject holds a config statement. In this case, all definitions inside the sdfObject that can use data qualities have readable and writable set as described above.

Status Statement
----------------

* YANG: {{Section 7.21.2 (status) of -yang}}
* SDF: {{Section 4.6 (common qualities) of -sdf}}

The status statement of YANG is used to express whether a definition is current, deprecated or obsolete which are the three possible arguments of the statement. In SDF there is no quality with a similar meaning. Thus, the YANG status statement is represented by a note in the description of the SDF definition corresponding to the YANG element the status statement occurred in as described in {{design-roundtrips}}.

Reference Statement
-------------------

* YANG: {{Section 7.21.4 (reference) of -yang}}
* SDF: {{Section 4.6 (common qualities) of -sdf}}

In YANG the reference statement holds a human-readable reference to an external document related to its containing YANG definition. This is simply stored in the description of the SDF definition analogue to the reference statement's parent YANG definition as described in {{design-roundtrips}}.

When and Must Statements
------------------------

* YANG: {{Section 7.5.3 (must) of -yang}} {{Section 7.21.5 (when) of -yang}}
* SDF: {{Section 4.6 (common qualities) of -sdf}}

As mentioned before in {{design-module}} YANG provides means to impose conditions on its definitions. If a node in an instance of the YANG module has an unfulfilled must or when condition it is invalidated. Must and when conditions use XML Path Language expressions to indicate dependencies. This feature is not realisable in SDF as of now. However, there is a query language similar to XML Path Language for JSON called JSONPath (CITE!). If SDF adopts JSONPath or something similar in the future the converter can be extended to process the functionality of must and when statements.

Extension Statement
-------------------

* YANG: {{Section 7.19 (extension) of -yang}}
* SDF: {{Section 4.6 (common qualities) of -sdf}}

The extension statement in YANG has the purpose of defining new statements for the YANG language. This is not a concept that can be transferred to SDF yet and thus has to be stored in the description of the SDF definition analogue to the extension statement's parent YANG definition as described in {{design-roundtrips}}.

Mapping from SDF to YANG
========================
In this section the conversion of each element of SDF to YANG is explained in detail. For reference on the individual YANG statements see {{-yang}} and {{-sdf}} for SDF.

Information Block
-----------------

* SDF: {{Section 3.1 (information block) of -sdf}}
* YANG: {{Section 7.1 (module) of -yang}}

At the top of an SDF model the information block holds meta data (title, version, copyright and license information) about the model. The content of the `title` quality is used as the name for the YANG module. For this, the title string has to be modified to only contain lower case letters, digits and the characters \"\_\", \"\-\" and \"\.\". If the `version` quality contains a date in the format month-day-year it is analogue to YANG's revision statement and converted as such. The strings from the copyright and license qualities are stored in the description of the resulting YANG module since there are no dedicated YANG statements equivalent to these qualities.

Namespace Section
-----------------

* SDF: {{Sections 3.2 and 4 (namespaces section) of -sdf}}
* YANG: {{Section 7.1.3 (namespace) of -yang}} {{Section 7.1.5 (import) of -yang}}

The purpose of SDF's namespace section is to specify the namespaces of the external models whose definitions are used in this model and possibly the namespace of this model. The namespace section has a `namespace` quality mapping namespace URIs to a shortened name for that URI (used as a prefix for external definitions). If an SDF model is supposed to contribute globally available definitions a value is given for the `defaultNamespace` quality and mapped to a namespace URI in the `namespace` quality. To map this to YANG three of its statements are necessary. To be able to use definitions from external modules in YANG the modules' names have to be declared by one `import` statement each. Thus, each external SDF model that is mentioned in the namespace map is converted to a YANG module as well. The corresponding SDF model files have to be available in the same directory as the model file of this model. The external SDF model's default namespace is represented in the `prefix` sub-statement of the `import` statement. To represent the own namespace and short name for it (if present) the YANG `namespace` and `prefix` statements that are both top-level statements are set accordingly.

sdfThing
--------

* SDF: {{Sections 2.2.6 and 6.3 (sdfThing) of -sdf}}
* YANG: {{Section 7.5 (container) of -yang}}

An sdfThing definition holds the model of a complex device that can be made up of one or more sdfObject and/or other sdfThing definitions. SdfThings are converted to YANG container nodes.

sdfObject
---------

* SDF: {{Sections 2.2.1 and 5.1 (sdfObject) of -sdf}}
* YANG: {{Section 7.5 (container) of -yang}}

SdfObject definitions are the main building blocks of an SDF model grouping together definitions of the classes sdfProperty, sdfData, sdfAction and sdfEvent. They can also be used as arrays via their `minItems` and `maxItems` qualities. An sdfObject is mapped to a YANG container node if it is not defined as an array. Otherwise the sdfObject is converted to a list node with the `min-elements` and `max-elements` statements set analogous to the `minItems` and `maxItems` qualities.

Common Qualities {#sec-map-comquali}
----------------

* SDF: {{Section 4.6 (common qualities) of -sdf}}
* YANG: 
   * {{Section 7.21.3 (description) of -yang}} 
   * {{Section 7.3 (typedef) of -yang}} 
   * {{Section 9.9 (leafref) of -yang}} 
   * {{Section 7.13 (uses) of -yang}} 
   * {{Section 3 (terminology for mandatory) of -yang}}

The set of qualities that is grouped under the name of common qualities can be used to provide meta data for SDF definitions. The `description` quality is converted to the YANG description statement. The `label` quality is ignored because it is identical to the definitions identifier in most cases.

The `sdfRef` quality is supposed to hold references to other definitions. The qualities of the referenced definition are copied into the referencing definition if they are not overwritten in the referencing definition. The conversion of an sdfRef depends on what is referenced by it and what that is converted to. If the referenced definition is converted to a typedef the sdfRef is analogous to the `type` statement in YANG which points to the typedef. If the referenced definition is mapped to a leaf or leaf-list node it can be referenced by the `leafref` built-in type in YANG. If the referenced definition's equivalent in YANG is a grouping node the sdfRef gets converted to a uses node which points to said grouping. In all other cases the referenced definition's equivalent cannot be referenced directly but has first to be packaged in a grouping node. This is done by first creating a grouping as a sibling node to the referenced definition's equivalent YANG node and copying the equivalent node into the new grouping. After that the equivalent node is replaced it with a uses node referencing the grouping. This is done to avoid redundancy. Lastly, the actual sdfRef is represented by another uses node referencing the newly created grouping.

The common quality `sdfRequired` contains a list of SDF declarations that are mandatory to be present in an instance of the SDF model. The issue with the conversion of this quality is that in YANG only leaf and choice nodes (and anyxml and anydata nodes but these are not used for conversion) can be labelled as mandatory while in SDF all declarations (i.e., sdfProperties, sdfActions and sdfEvents that occur in an sdfObject) can be mentioned in the sdfRequired list. Not all SDF declarations are always converted to YANG leaf or choice nodes, however. To partially make up for this discrepancy, if the YANG node equivalent of the mandatory SDF declaration is container node the node's sub-tree is traversed until a leaf or choice node is found. This leaf or choice node is labelled as mandatory, now making its parent container mandatory as well because one of its child nodes is mandatory. Consequently, if the parent node of the now mandatory container was a container it would now be mandatory as well. Alternatively, if a list or leaf-list node is found first the node's `min-elements` statement is set to one if it is not already set to a value greater than zero. This also makes a node mandatory. To facilitate retracing the declaration originally listed in the `sdfRequired` quality, e.g., for round trips, the `sdf-spec` extension statement is set as described in {{design-roundtrips}}.

Data Qualities {#sec-map-dataquali}
--------------

* SDF: {{Section 4.7 (data qualities) of -sdf}}
* YANG: 
   * {{Section 7.4.1 (type) of -yang}} 

The set of qualities labelled as data qualities contains qualities that SDF borrowed from json-schema.org as well as qualities specifically defined for SDF. In the first group there is a total of 18 qualities out of which some are interdependent.

The quality that a lot of the other qualities presence or absence depends on is the `type` quality. The type can be one of `number`, `string`, `boolean`, `integer`, `array` or `object`. This quality is directly converted to the YANG `type` statement for all simple types (where `number` becomes `decimal64` and integer becomes `int64`). The types `array` and `object` cannot be converted to a YANG built-in type directly. Instead SDF definitions with these types are converted as described in sections {{sec-map-sdfProp}} and {{sec-map-sdfData}}.

The SDF data quality `const` hold the constant value of a definition if there is one. If the value of the `type` quality is `number` or `integer` the `const` quality is mapped to the `range` sub-statement of YANG's `type` statement which can also contain a single value. For constant string values the YANG `pattern` statement containing the constant string is be used. Unfortunately, constant values of types boolean, array and object have to be ignored since there is no way to represent them in YANG.

The `default` data quality in SDF holds the default value for its definition. Since YANG leaf and leaf-list nodes have a `default` sub-statement SDF default values of simple types or of type array with items of simple types can easily be represented. Default values of either compound-type or type array with compound-type items cannot be represented in YANG, unfortunately.

The data qualities `minimum`, `maximum`, `exclusiveMinimum` and `exclusiveMaximum` which are only valid for the types number and integer are converted using the YANG `range` statement again. For exclusive boundaries the range is reduced accordingly in YANG. If the range already contains a constant value an alternative range can be added. The alternatives in the range have to be disjoint, however. If the constant value lies inside the range specified by the `minumum` and `maximum` qualities this cannot be represented simultaneously in YANG. Whichever of the two qualities is converted first is represented.

The `multipleOf` data quality is one that can only be used in conjunction with the number type in SDF and states the resolution, i.e., the number of possible digits after the decimal separator, of the decimal number. This quality is converted to the `fraction-digits` sub-statement to the `type` statement in YANG.

SDF's `minLength` and `maxLength` data qualities are used to hold a strings minimal and maximal length. This concept can be transferred to YANG by using the `length` sub-statement of the `type` statement that specifies a length range.

The SDF `pattern` data quality holds regular expressions for string typed definitions. This can be converted directly to the `pattern` sub-statement to the `type` statement in YANG. As already mentioned in {{design-string}} regular expressions cannot be converted directly between SDF and YANG in theory due to the differing languages used for regular expressions. Due to the time limitations of this thesis, however, no further measures are taken to insure the conformance of converted regular expressions.

The string type in SDF can be concretized by the `format` quality. This quality can specify one of the JSON schema formats. This could be translated to YANG referencing typedefs from the widely used `ietf-yang-types` module. However, to not rely on external modules the format is only mentioned in the description of the YANG equivalent to the `format` quality's SDF definition.

The length of an array in SDF can be restricted by the `minLength` and `maxLength` data qualities. Both list and leaf-list nodes use the sub-statements `min-elements` and `max-elements` to express the same concept which are used to convert the SDF array length qualities.

Another restriction for SDF arrays is the `uniqueItems` quality that can be set to either `true` or `false`. If it is set to `true` all items of an array have to be unique. YANG specifies a `unique` sub-statement for list nodes but it can only be applied to leaf and leaf-list nodes in the sub-tree. Thus, if the SDF array in question is converted to a YANG list node and the `uniqueItems` quality is set to true the list's `unique` statement mentions all of the list's descendant leaf or leaf-list nodes. It is not possible, unfortunately, to represent the `uniqueItems` quality in leaf-list nodes that stem from SDF arrays.

The `items` data quality of SDF is a quality that specifies item constraints for the items of an array-typed SDF definition using a subset of the common and data qualities. SDF definitions with the type array are converted to list or leaf-list nodes. These node types in themselves indicate the type array. Thus, the qualities defined in the array's item constraints are converted to the list and leaf-list node's sub-statements as described in this section.

The SDF data qualities include the `properties` quality. These properties are different from sdfProperties. The `properties` quality is used in conjunction with the object type and contains a set of named definitions made up of data qualities themselves. SDF defintions of type object are converted to container or grouping nodes thus the single named properties in the `properties` quality are transformed to the node's child nodes. The SDF type `object` was only introduced in SDF 1.1. This feature made conversion significantly more complicated. To label the properties as mandatory the `required` quality is used. Since it is resembling the `sdfRequired` quality it is translated in the same way.

The second group of qualities that is part of the data qualities includes 11 qualities as well as the common qualities described in {{sec-map-comquali}}.

The `unit` quality can be set to any of the SenML unit names to represent an SDF definitions unit. There is also a `unit` statement as a sub-statement to typedefs, leaf nodes and leaf-list nodes. The `unit` statement in YANG can contain any string and thus is simply set to the SenML unit name from the SDF definition.

An important data quality is the `sdfChoice` quality. It represents the choice between several sets of named definitions made up of data qualities themselves. YANG provides a very similar statement, the `choice` statement. An sdfChoice is turned into a YANG choice node. Each of the sdfChoice's alternatives is converted like an sdfProperty (see {{sec-map-sdfProp}}) and added to the choice node inside its own case node. SdfChoice definitions that give the choice between the `type` quality could also be mapped to the YANG type union. This is omitted for reasons of simplicity.

SDF also offers the possibility to define the choice between string values by means of the `enum` data quality. It consists of an array of strings. This concept also exists in YANG with the `enumeration` type and `enum` sub-statement to the `type` statement. For an SDF definition that contains the `enum` quality the YANG type of its equivalent is set to `enumeration`. Each of the strings in the array of the `enum` SDF quality is converted to an `enum` entry in the `type` statement in YANG.

SDF's `contentFormat` quality can provide an additional IANA content type.
This is turned into a note in the description of the SDF definition's YANG equivalent.

Another way to specify the `type` quality is the `sdfType` quality that can either be set to `byte-string` or `unix-time`. A byte string is converted to the YANG type `binary`. There is no built-in YANG type corresponding to unix time thus a note is added in the description of the SDF definition's YANG equivalent.

SDF defines the `readable` and `writable` qualities to flag whether read or write operations are allowed on definitions. Read operations are always allowed in YANG modules so a `readable` quality that is set to false cannot be represented in YANG. YANG's `config` statement can be used to represent the value of the `writable` quality, however. If an SDF definition is explicitely marked as writable `config` is set to `true`. Otherwise, it is set to `false`.

The `observable` and `nullable` qualities in SDF cannot be represented in YANG.

sdfData {#sec-map-sdfData}
-------

* SDF: {{Sections 2.2.5 and 5.5 (sdfData) of -sdf}}
* YANG: 
   * {{Section 7.13 (uses) of -yang}} 
   * {{Section 7.12 (grouping) of -yang}}

Elements of the sdfData class are meant to hold data type definitions to be shared by sdfProperty, sdfAction and sdfEvent definitions. SdfData definitions can make use of the data qualities and the common qualities described in sections {{sec-map-dataquali}} and {{sec-map-comquali}} respectively. Because an sdfData element embodies a data type definition the YANG statements the `typedef` and `grouping` have to be used for conversion. Which of the two is used depends on the value of the `type` quality of the sdfData element. If the type is one of the simple data types, i.e., integer, number, boolean or string, the sdfData definition is converted to a YANG typedef. If the type is compound-type the sdfData definition is mapped to a grouping node with each of the compound-type's properties being mapped to a child node of the grouping. For sdfData definitions with type array the type mentioned in the `type` quality of the `items` quality is essential as well. If an array has items of any of the simple types the resulting YANG element is a grouping node containing a single leaf-list node. Otherwise, if the array items are compound-types the sdfData definition is converted into a grouping node containing a single list node. The list node's child nodes are equivalent to the compound-type item's properties. One issue with converting sdfData definitions of type array is the added grouping node that is necessary to hold the leaf-list/list node. If the grouping is used in the schema tree the added level will cause model instances of the original and converted model to be in-equivalent. If the sdfData definition is referenced in the SDF model via the `sdfRef` common quality this is represented in YANG with the `uses` statement pointing to the grouping equivalent to the sdfData definition.
The `sdfRef` quality can occur at most once in each definition while there can be multiple `uses` statements in the same container/list/grouping. Thus, the aforementioned issue with array-typed sdfData definitions could be solved by, instead of representing definitions containing an sdfRef by a parent node containing a `uses` node, replacing the parent node with the uses node itself, effectively removing the excess level. This, however, gives rise to other issues because the name of the sdfRef's superordinate definition is lost. If the sdfData definition is converted to a typedef no such issues arise. The typdef in question is inserted as an argument to the YANG type quality wherever the original sdfData definition was referenced by an sdfRef. Another issue is a different view on global accessibility of data type definitions in YANG and SDF. In SDF all definitions are globally available as long as a default namespace is defined in the SDF model. In YANG on the other hand, only data type definitions (i.e., groupings and typedefs) that occur on the top-level of YANG module are globally accessible. Thus, to represent the global accessibility of all data type definitions in SDF, all converted sdfData definition equivalents in YANG are added to the top-level of the created module.

sdfProperty {#sec-map-sdfProp}
-----------

* SDF: {{Sections 2.2.2 and 5.2 (sdfProperty) of -sdf}}
* YANG: 
   * {{Section 7.6 (leaf) of -yang}} 
   * {{Section 7.7 (leaf-list) of -yang}}
   * {{Section 7.8 (list) of -yang}}

SdfProperty definitions represent elements of state as suggested by their name. SdfProperty definitions can make use of the data qualities and the common qualities described in sections {{sec-map-dataquali}} and {{sec-map-comquali}} respectively. The mapping of an sdfProperty definition to YANG depends on the value of the `type` quality. SdfProperties with simple types are mapped to leaf nodes in YANG. If the type is complex, i.e., compound-type, conversion results in a container node with each of the compound-type's properties being mapped to a child node of the container. If the sdfProperty is of type array the deciding factor is the `type` quality inside the `items` quality. If an array has items of a simple type it is converted to a leaf-list node. Otherwise, if the items are of compound-type the sdfProperty becomes a list node in YANG. The list node's child nodes are equivalent to the compound-type's properties.

sdfAction
---------

* SDF: {{Sections 2.2.3 and 5.3 (sdfAction) of -sdf}}
* YANG: 
   * {{Section 7.14 (rpc) of -yang}} 
   * {{Section 7.15 (action) of -yang}}

To represent commands/operations that can be invoked in a model the sdfAction class is used. Since commands can have input and output data the sdfAction class is equipped with the `sdfInputData` and `sdfOutputData` qualities that can both make use of the data qualities and the common qualities described in sections {{sec-map-dataquali}} and {{sec-map-comquali}} respectively. An sdfAction can also define its own set of data types in the form of sdfData definitions. Whether an sdfAction is converted to an RPC (which can only occur at the top-level of a module) or an action node (which is always tied to a container node) depends on its location inside the SDF model. SdfActions that are not part of an sdfObject but can be found independently at the top of an SDF model are converted to RPC nodes. All other actions occurring inside an sdfObject become action nodes inside the sdfObject's container equivalent in YANG. The sdfInputData and sdfOutputData of an sdfAction are converted like sdfProperties (see {{sec-map-sdfProp}}) and added as the input and output node of the RPC/action respectively.

sdfEvent
--------

* SDF: {{Sections 2.2.4 and 5.4 (sdfEvent) of -sdf}}
* YANG: {{Section 7.16 (notification) of -yang}}

The sdfEvent class' purpose is to model signals that inform about occurrences or \"happenings\" in an sdfObject. To represent the emitted output data sdfEvents can make use of the `sdfOutputData` quality which in turn uses the data qualities. An sdfEvent is converted to a notification node with one child node representing the sdfOutputData definition. The


Challenges
==========
Since conversion between SDF and YANG is not always trivial this section takes a look at the various challenges that arose in the process of finding an adequate mapping for each of the language's features to one another.


Differences in Expressiveness of SDF and YANG
---------------------------------------------
SDF and YANG differ in their expressiveness in different areas. Compared to the other format, both are stronger in some areas and weaker in others. Areas in which YANG is more expressive are type references, regular expressions, operations and some of the built-in types (specifically `bits`, `empty`} and `union`). SDF offers more possibilities to define default and constant values, the latter especially in conjunction with minimum and maximum values for which a single statement is used in YANG. Labelling definitions as readable, observable and nullable, as possible in SDF, is foreign to YANG.


Round Trips      {#design-roundtrips}
-----------
One of the bigger issues in developing a mapping for each language feature of SDF and YANG was the facilitation of round trips, i.e., converting a model from one format to the other and in a next step back to the original. This issue is tightly linked to the differences in expressiveness between the two formats which makes mapping between them non-injective and thus non-traceable.

To be able to track the origins of an SDF element after conversion from YANG, currently, a "conversion note" is added to the description of said element. The note states a statement and an argument. An example for a note hinting the original argument to the `type` statement was `union` could be: "!Conversion note: type union!". This issue was also discussed in one of the ASDF working group's Interim Meetings where the possibility to introduce a new round trip mechanism native to SDF was suggested.

To preserve the original SDF language element after conversion to YANG a new extension is defined in YANG. The extension states the original SDF quality.

The eventuality that round trips occur in model conversion makes implementing a converter significantly more complex because all features of the target format have to be accounted for. Features of the target format that would otherwise not be used for conversion must now be considered in the case of a round trip.


Type References
---------------
Both SDF and YANG offer the possibility to reference predefined types. SDF uses only a single quality for this purpose (`sdfRef`) whereas YANG has several statements that are all used in different referencing contexts (`leafref`, `identityref`, `type`, `uses`).

The `sdfRef` quality is supposed to hold references to other definitions. The qualities of the referenced definition are copied into the referencing definition if they are not overwritten in the referencing definition. The conversion of an sdfRef depends on what is referenced by it and what that is converted to. If the referenced definition is converted to a typedef the sdfRef is analogous to the `type` statement in YANG which points to the typedef. If the referenced definition is mapped to a leaf or leaf-list node it can be referenced by the `leafref` built-in type in YANG. If the referenced definition's equivalent in YANG is a grouping node the sdfRef gets converted to a uses node which points to said grouping. In all other cases the referenced definition's equivalent cannot be referenced directly but has first to be packaged in a grouping node. This is done by first creating a grouping as a sibling node to the referenced definition's equivalent YANG node and copying the equivalent node into the new grouping. After that the equivalent node is replaced it with a uses node referencing the grouping. This is done to avoid redundancy. Lastly, the actual sdfRef is represented by another uses node referencing the newly created grouping.



Implementation Considerations
=============================

An implementation of an initial converter between SDF and YANG can be
found at {{-impl}}; the source code can be found at {{-implsrc}}.

IANA Considerations
==================

This document makes no requests of IANA.


Security considerations
=======================

The security considerations of {{-yang}} and {{-sdf}} apply.

--- back

Acknowledgements
================
{: numbered="no"}

TBD.



<!--  LocalWords:  sdfRequired sdfObject SDF sdfProperty sdfObject's
 -->
<!--  LocalWords:  sdfChoice sdfAction sdfRef sdfData sdfEvent anyxml
 -->
<!--  LocalWords:  sdfObjects sdfThings sdfActions sdfAction's YANG's
 -->
<!--  LocalWords:  sdfInputData sdfOutputData sdfEvent's typedefs
 -->
<!--  LocalWords:  sdfThing sdfProperties sdfEvents anydata SDF's
 -->
<!--  LocalWords:  deserialize sdfType typedef leafref identityref
 -->
<!--  LocalWords:  nullable ASDF boolean
 -->
