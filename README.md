# MongoDB Style Guide

## Table of Contents

  1. [General](#general)
  1. [Enumerations](#enumerations)
  1. [Booleans](#booleans)
  1. [Dates](#dates)
  1. [Null and undefined](#null-and-undefined)
  1. [Other types](#other-types)
  1. [Names](#names)
  1. [Object modelling](#object-modelling)
  1. [Todo](#todo)


## General

- [1.1](#general--mix) **Mixed types**: Don't mix values of different types in one column. Restrict yourself to one type per column

    > Why? This way, you don't have to typecheck or cast when you consume the data.

    | :x:           |:white_check_mark: |
    | :------------ | :---------------- |
    | 1             | 1                 |
    | "2"           | 2                 |
    | { value: 3 }  | 3                 |

- [1.2](#general--object-schema) **Object columns**: If you have a column that contains objects, make sure all objects share the same format

    | :x:       | :white_check_mark:     |
    | :---------------- | :------------- |
    | { value: 42 }    | { value: 42 }   |
    | { value: "foo" } | { value: 43 }   |
    | { }              | { value: null } |

- [1.3](#general--falsiness) **Falsiness**: Don't use `""` or `0` for their falsiness

    > Why? It's very easy to write buggy code otherwise which is also why you should not coerce to booleans in your if statements.

    | :x:       | :white_check_mark:   |
    | :---------------- | :------------- |
    | "foo"            | "foo"         |
    | "bar"            | "bar"         |
    | ""               | null          |


## Enumerations

An enumeration is a type that allows a limited number of values, e.g. a userType field that can contain the values "DOCTOR", "NURSE", "PATIENT" and "ADMINISTRATOR"

- [2.1](#enumerations--modelling) **Modelling**: Always model your enums as uppercase string constants, e.g. "WAITING", "IN_PROGRESS" and "COMPLETED"

    > Why?
    > 1. Favoring strings over booleans or numbers means your data is self-documenting. If your gender field contains the value `1`, you don't know if that means male, female or something else
    > 1. Using uppercase strings makes it easy to see at a glance whether something is an enum or a user-facing value

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | 0             | "MALE"              |
    | 1             | "FEMALE"           |

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | "男"             | "MALE"              |
    | "女"             | "FEMALE"           |

- [2.2](#enumerations--explicit) **Explicit states**: Don't use `null`, `undefined`, or any value except upper case string constants in your enums. This includes initial, undecided or unknown states

    > Why? This makes the meaning explicit. If you have an assessmentState field that's set to `null`, you can't tell whether that means "no assessment necessary", "not applicable", "patient didn't show up", "undecided" or any other possible state

    | :x: assessmentState | :white_check_mark: assessmentState |
    | :------------------ | :--------------------------------- |
    | null                | "NOT_REQUIRED"                     |
    | *missing*           | "NOT_APPLICABLE"                   |


## Booleans

- [3.1](#booleans--booleans-and-enums) **Booleans and enums**: Don't model things as booleans that have no natural correspondence to true/false, even if they only have two possible states. Use enums instead

    > Why?
    > 1. Booleans can not be extended beyond two possible values but the field might have to be extended later to include additional values, e.g. a boolean field "hasArrived" could might have to be extended later to include the possibility of cancelled appointments
    > 1. It makes meaning explicit. If your gender field contains the value true, you don't know if that means male or female

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | false             | "MALE"              |
    | true             | "FEMALE"           |

- [3.2](#booleans--prefix) **Prefix**: Model things as booleans that are prefixed with verbs such as "is..." or "has..." (e.g. "isDoctor", "didAnswerPhoneCall" or "hasDiabetes")

- [3.2](#booleans--orthogonal) **Orthogonality**: If you have several mutually exclusive boolean fields in your collection, merge them into an enum

    > Why? So that it's impossible to save invalid data in your db like a car that's green and red simultaneously

    :x:

    | isRed              | isBlue | isGreen |
    | :---------------- | :----------------- | :----------------- |
    | false             | true              |  false              |
    | true             | false         | false         |

    :white_check_mark:
    
    | color |
    | :---------------- |
    | "BLUE"              |
    | "RED"         |

## Dates

- Don't save dates as serialized ISO strings like "2017-04-14T06:41:21.616Z". Do save them as Date objects
- **Don't use Date when all you are concerned with is a timezone independent day value**. Do use strings of the form "YYYY-MM-DD" in those cases

## Null and undefined

- don't overload the meaning. "missing values"

- Don't use null or undefined when there is a sensible default (e.g. when you have a "notes" field, the initial value should be the empty string, not null). Do prefer null over undefined or missing values

## Other types

- Don't save numbers as strings. Do make an exception for data like phone numbers or shenfenzheng ids, for which leading zeroes are significant 
- Don't mix ObjectIds with any other types like string. Do use ObjectIds for _id when there is no obvious unique field suitable to be used as _id

## Names

- Don't use snake_case. Do use camelCase
- Don't use abbreviations except for widely used, domain specific language. Do learn the lingo of your field

## Object modelling

- **Don't let your objects keep growing. Do prune them and make judicious use of nested objects to group properties that belong together** (e.g. combining "footAssessmentState", "eyeAssessmentState" and "nutritionAssessmentState" into an "assessmentStates" nested object with properties "foot", "eye" and "nutrition"
- **Don't excessively nest objects. Do consider breaking up your data if you find yourself needing deeply nested objects**

## Todo

Normalization/denormalization