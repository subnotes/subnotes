# subnotes

[![License: CC BY-SA 4.0](https://licensebuttons.net/l/by-sa/4.0/80x15.png)](http://creativecommons.org/licenses/by-sa/4.0/)

An open and extensible specification for notes

Uses [JSON](http://www.json.org/), [JSON Schema](http://json-schema.org/), and [JSON Pointers](https://tools.ietf.org/html/rfc6901)

> WARNING: This spec is a *very* early rough draft, incomplete, and constantly changing.

### NoteBook

* MUST contain a *Topic* object

```json
{
  "NoteBook": {
    "type": "object",
    "properties": {
      "Topic": {
        "$ref": "#/definitions/Topic"
      }
    },
    "required": [
      "Topic"
    ]
  }
}
```

### Topic

* MUST contain a *topic* - string
* MAY contain a *note* - string
* MAY contain *subnotes* - array of *Note* objects and *NotePointer* JSON pointers
* MAY contain *subtopics* - array of *Topic* objects
* MUST belong to a *Topic* or *NoteBook* object

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
              "$ref": "#/definitions/NotePointer"
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

### NotePointer

A regular expression that matches valid JSON pointers for *Notes*. But it does not validate that a *Note* exists at that path.

```json
{
  "NotePointer":{
    "description": "A regular expression matching a JSON pointer to a Note.",
    "type": "string",
    "pattern": "#\/[0-9]{1,}(\/subtopics\/[0-9]{1,}){0,}(\/subnotes\/[0-9]{1,}){1}$"
  }
}
```

### Extensions

Coming soon...

* Expand JSON schemas above to handle extensions 
* Extensions of extensions
* Extensions for Notes, Topics, NoteBooks, or combinations
* Extension dependencies
