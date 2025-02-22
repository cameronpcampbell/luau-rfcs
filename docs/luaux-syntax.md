# Luaux Syntax.

## Summary
Add jsx-esque (luaux) syntax to the langauage.

## Motivation

Luau is a powerful language used for many purposes, one example being defining user interfaces. User Interfaces often have nested hierarchies of elements which can be cumbersome to represent with curly-brace table syntax, For example:
```luau
{
    name = "Frame",
    size = vector.create(200, 200, 0),
    children = {
        {
            {
                name = "Button",
                size = vector.create(100, 80),
                children = {
                    {
                        {
                            name = "Image", 
                            imagepath = "path/to/image/foo.ext"
                        },
                        "I am a button"
                    }
                }
            }
        } 
    }
}
```

## Design

This RFC aims to solve the issue above by introducing a new syntax to represent tables, optimised for expressing user interfaces.

At its core a Luaux element is simply a table with a `tag` property.

### The following is proposed:

#### Defining An Element:
A Luaux element can be defined with the following syntax.
```luau
<element></element>

-- Self-closing shorthand.
<element />
```

This will result in the following table:
```luau
{
    tag = "element"
}
```

Tags are inferred as a string unless a function with the same name is in scope, in which case the tag is inferred as that function.
```luau
local function element()
    ...
end;

<element /> --[[
    {
        tag = function: 0x...
    }
]]
```

#### Element Properties:
Element properties can be defined as follows:
```luau
<element foo="bar" /> --[[
    {
        tag = "element",
        foo = "bar",
    }
]]
```

#### Element Children:

You can add a child to an element in between its opening and closing tag.
```luau
-- Element with one child.
<element>
    <child />
</element>
```

The resulting table of the element will have a `children` property with its value set to the child.

```luau
{
    tag = "element",
    children = {
        tag = "child"
    }
}
```

If the element has more than one child then the resulting `children` property will be an array-like table containing multiple elements:
```luau
<element>
    <child />
    <foobar />
    <child>
        <foo />
    </child>
</element> --[[
    {
        tag = "element",
        children = {
            {
                tag = "child"
            },
            {
                tag = "foobar"
            },
            {
                tag = "child",
                children = {
                    tag = "foo"
                }
            },
        }
    }
]]
```

**The order children are defined in is preserved in the resulting table.**

#### Interpolation.
Similar to strings, you can interpolate luau expressions inside luaux by wrapping them with curly-braces.

```luau
<element foo={55} bar={vector.create(200, 80)}>
    <child baz={true} />
    {"hello there"}
</element> --[[
    {
        tag = "element",
        foo = 55,
        bar = vector.create(200, 80),
        children = {
            {
                tag = "child",
                baz = true
            },
            "hello there"
        }
    }
]]
```

## Drawbacks

- Luaux adds new syntax which further complicates the language of which is intended to be simple.

## Alternatives

We could not implement this as its not impossible to define user interfaces using curly-brace table syntax.