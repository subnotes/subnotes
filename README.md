# SubNotes

[![License: CC BY-SA 4.0](https://licensebuttons.net/l/by-sa/4.0/80x15.png)](http://creativecommons.org/licenses/by-sa/4.0/)

An open and extensible specification for notes

Uses [JSON](http://www.json.org/), [JSON Schema](http://json-schema.org/), and [JSON Pointers](https://tools.ietf.org/html/rfc6901)

> WARNING: This spec is a ***very*** early rough draft, incomplete, and constantly changing.
   
What are JSON Pointers and JSON Schema? [Understanding JSON Schema](https://spacetelescope.github.io/understanding-json-schema/index.html) is a short, well written document that covers both subjects.

### Core Spec and Extensions

SubNotes consist of a core specification (spec) and extensions. The core spec and how to extend it are described in this document. The extensions themselves SHOULD be described in their own documents.

### Trees

The core SubNotes spec consist of three different *Nodes* arranged a in tree structure. The core node types are, *Note*, *Topic*, and *Notebook*. Each node type requires a property that shares the same name as the node. When referring to the node, the name starts with a capital, like *Note*. When referring to the node's required property name, it is all lowercase, like *note*. (Note: Other property names SHOULD also start with a lowercase letter).

#### Conventions

> sub\* = child\*

For example, "subnote" is a synonym for "child note", and "subtopics" is a synonym for "child topics."

#### Notes

A *Note* MUST have a *note* property, which MUST be a string, but it is not required to have any other properties. If we set aside extensions for the moment, a *Note* is a leaf node, meaning it has no children (subnodes). A *Note* must belong to a *Topic* node. You can think of a *Note* kind of like a piece of paper that has to be in a folder.

#### Topics

A *Topic* MUST have a *topic* property, which MUST be a string, but it is not required to have any other properties. A *Topic* may also contain an array of *subtopics* (child *Topics*) and an array of *subnotes* (child *Notes*). A *Topic* must belong to another *Topic* node or a *Notebook*. You can think of a *Topic* like a labeled folder that can contain pieces of paper (a *Note*) and other folders (a *Topic*).

#### Notebook

A *Notebook* MUST have a *notebook* property, which must be a *Topic* node, but it is not required to have any other properties. A *Notebook* is the root node. The top level *Topic*. You can think of a *Notebook* like a drawer (or notebook) that can contain folders (*Topics*). 

*Example Notebook - Tree Structure*

```
{Notebook}
 |-{Note 1}
 |-{Topic 1}
 |  |-{Note 1}
 |  |-{Note 2}
 |  |-{Topic 1.1}
 |  |  |-{Note 1}
 |  |-{Topic 1.2}
 |     |-{Note 1}
 |-{Topic 2}
    |-{Note 1}       
```

### Beyond Trees with JSON Pointers

A tree structure can be great for organizing things, but sometimes it can be too rigid. Thankfully there are JSON Pointers. You can think of them as links. In SubNotes a *subtopic* can be a JSON Pointer to another *Topic* (as long as it is not one of its ancestors). This is called a *TopicPointer*. Also, a *subnote* can be a JSON Pointer to another *Note*. This is called a *NotePointer*.

*Example Notebook - Note and Topic Pointers*

```
{Notebook}
 |-{Note 1}
 |-{Note 2}
 |-{Topic 1]} <---------------+
 |  |-{Note 1}                |
 |  |-{Note 2} <-----------+  |            
 |  |-{Topic 1.1}          |  |
 |  |  |-{Note 1}          |  |
 |  |-{Topic 1.2}          |  |
 |     |-{NotePointer 1} --+  |
 |-{Topic 2}                  |
    |-{Note 1}                |
    |-{TopicPointer 2.1} -----+
           
```

### Notebook

* MUST contain a *notebook* - *Topic* object

*JSON Schema*

```json
{
  "Notebook": {
    "type": "object",
    "properties": {
      "notebook": {
        "$ref": "#/definitions/Topic"
      }
    },
    "required": [
      "notebook"
    ]
  }
}
```

### Topic

* MUST contain a *topic* - string
* MAY contain a *note* - string
* MAY contain *subnotes* - array of *Note* objects and *notePointer* JSON pointers
* MAY contain *subtopics* - array of *Topic* objects
* MUST belong to a *Topic* or *NoteBook* object

*JSON Schema*

```json
{
  "Topic": {
    "type": "object",
    "properties": {
      "topic": {
        "description": "A topic. String. Required.",
        "type": "string",
        "minLength": 1
      },
      "note": {
        "description": "A note about the topic. String. Optional.",
        "type": "string"
      },
      "subnotes": {
        "description": "An array of Notes and NotePointers",
        "type": "array",
        "items": {
          "anyOf": [
            {
              "$ref": "#/definitions/Note"
            },
            {
              "$ref": "#/definitions/notePointer"
            }
          ]
        }
      },
      "subtopics": {
        "description": "An array of Topics. Optional.",
        "type": "array",
        "items": {
          "$ref": "#/definitions/Topic"
        }
      }
    },
    "required": [
      "topic"
    ]
  }
}
```

### Note

* MUST contain a *note* - string
* MAY contain a *topic* - string
* MUST belong to a *Topic* object

*JSON Schema*

```json
{
  "Note": {
    "type": "object",
    "properties": {
      "note": {
        "description": "A note. String. Required.",
        "type": "string",
        "minLength": 1
      },
      "topic": {
        "description": "The topic of the note. String. Optional.",
        "type": "string"
      }
    },
    "required": [
      "note"
    ]
  }
}
```

### topicPointer

* MUST be a string that could be a valid JSON pointer for a *Topic* based on its pattern. This does not validate that a *Topic* exists at that path.
* When managing topicPointers three things MUST be done.
  1. Make sure the path is actually pointing to a *Topic*.
  2. Make sure the path is not pointing to a parent *Topic*, creating a infinite loop.
  3. Make sure the path is updated if the target *Topic* or any of its parent *Topic* nodes are moved. 

*topicPointer JSON Schema*
  
```json
{
  "topicPointer":{
    "description": "A JSON pointer to a Topic.",
    "type": "string",
    "pattern": "#\/notebook\/[0-9]{1,}(\/subtopics\/[0-9]{1,}){0,}$"
  }
}
```

### notePointer

* MUST be a string that could be a valid JSON pointer for a *Note* based on its pattern. This does not validate that a *Note* exists at that path.
* When managing notePointers two things MUST be done.
  1. Make sure the path is actually pointing to a *Note*.
  2. Make sure the path is updated if the target *Note* or any of its parent *Topic* nodes are moved. 
  
*JSON Schema*

```json
{
  "notePointer":{
    "description": "A JSON pointer to a Note.",
    "type": "string",
    "pattern": "#\/notebook\/[0-9]{1,}(\/subtopics\/[0-9]{1,}){0,}(\/subnotes\/[0-9]{1,}){1}$"
  }
}
```

### Example Notebook

*JSON*

```json
{
  "notebook": {
    "topic": "Geography",
    "note": "This is a partially filled notebook about different countries through out the world.",
    "subnotes": [
      {
        "topic": "Continent",
        "note": "any of the world's main continuous expanses of land (Africa, Antarctica, Asia, Australia, Europe, North America, South America)."
      },
      {
        "topic": "Country",
        "note": "a nation with its own government, occupying a particular territory."
      }
    ],
    "subtopics": [
      {
        "topic": "Africa"
      },
      {
        "topic": "Antarctica"
      },
      {
        "topic": "Asia",
        "subtopics": [
          {
            "topic": "Afghanistan",
            "note": "a landlocked country located within South Asia and Central Asia",
            "subnotes": [
              {
                "topic": "Capital",
                "note": "Kabul"
              }
            ]
          }
        ]
      },
      {
        "topic": "Australia"
      },
      {
        "topic": "Countries by alphabetical order",
        "subtopics": [
          "#/notebook/2/subtopics/0"
        ]
      }
    ]
  }
}
```

**TODO:**
* Should Notes and Topics keep locations of Pointers pointing to them? Like reverse pointers/mapping. Might make moving Topics or Notes around easier. You wouldn't have to traverse the whole Notebook looking for Note and Topic Pointers to update.
* How are Notebooks, Topics, and Notes going to integrate extensions? Probably just an **extensions** array. 
 
### Extensions

Coming soon...

Initial thoughts...
* Expand JSON schemas above to handle extensions 
* Extensions of extensions
* How to define extensions for Notes, Topics, NoteBooks, or combinations
* Extension dependencies
* Two Types of extensions? 
  * *Meta* (eg {"markdown": "true"} or {"author": "Arthur Conan Doyle"} ) 
  * *Node* (eg {"flashcards": [*array of flashcard Nodes*]} )?
* Initial planned "flagship" extensions
  * *markdown* (meta) (support for CommonMark)
    * *markdown-math* (support for Tex/LaTex markdown extension)
  * *flashcards* (node) (adds flashcards to Notes and Topics)
    * *flashcards-spaced* (adds support for spaced repetition)
