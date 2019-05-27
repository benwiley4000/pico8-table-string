# pico8-table-string

Store a nested Lua data table as one giant string. Functions run in PICO-8 and in standard Lua environments. Works for flat and nested tables.

*NOTE!*: This can be helpful if you have a really large amount of data whose token count you want to cut down, but there are [other techniques you can try first](https://github.com/seleb/PICO-8-Token-Optimizations/blob/master/README.md) which may be more effective generally.

### *Before*

```lua
my_table = {
  hello='world',
  nested={
    some='data\'s nested',
    inside_of='here'
  },
  'and',
  'indexed',
  'data',
  { as='well' }
}
```

### *After*

```lua
my_table=
table_from_string(
 '1and2indexed3data4aswellhelloworldnestedinside_ofheresomedata\'s nested'
)
```

## Why?

There's only one good set of reasons (that I can think of) to store your data in this particular way:

1. You're building a [PICO-8](https://www.lexaloffle.com/pico-8.php) game
2. Your game is too big and your remaining token count is running low
3. You have a lot of string data taking up a lot of tokens

If these are true (or even if not), you can take advantage of this to cut down on your number of tokens. Create your data in a separate Lua file, use this tool to turn it into a giant string, and deserialize your data back into a Lua table when your program boots up.

Note that if the amount of data you're serializing is small, you'll actually add more tokens by including the library. The `table_from_string` function, which is required at runtime to rebuild the table, takes up 139 PICO-8 tokens. But now the actual declaration of the table takes only 5 tokens, so if you have a table taking up 145 or more tokens, this can give you some significant savings.

## Usage

First, clone this repository:

```console
git clone https://github.com/benwiley4000/pico8-table-string.git
cd pico8-table-string/
```

### Serializing

Assuming you have [Lua](https://www.lua.org/start.html) installed, the easiest way to generate a string version of your data is to paste your table into the [print-table-data](/print-table-data) file and run it from the command line:

```console
$ ./print-table-data
my_table=
table_from_string(
 '1and2indexed3data4aswellhelloworldnestedinside_ofheresomedata\'s nested'
)
```

Alternatively, if you want to generate with PICO-8 directly, that's also possible.

First, copy the contents from [stringify.lua](/stringify.lua) into your PICO-8 code editor (excluding the `return` statement at the bottom of the file).

Then add this code:

```lua
-- replace with whatever your table is called
local table_str = serialize_table(my_table)

printh('my_table=','outfile')
printh('table_from_string(','outfile')
printh(' '..table_str,'outfile')
printh(')','outfile')
```

Then check the file `outfile.p8l`:

```
my_table=
table_from_string(
 '1and2indexed3data4aswellhelloworldnestedinside_ofheresomedata\'s nested'
)
```

You can take this and include it in your code.

### Parsing at runtime

Note that at runtime, the only function you need to include is `table_from_string` from [parse.lua](/parse.lua).

## Serialization format

Instead of using something like JSON or a string embedding Lua's own table format (which could have been ideal if we had access to an eval function in PICO-8), we essentially just concatenate all the keys and values from a table into a giant string, with a few ASCII control characters as token delimiters and structure indicators (to allow table nesting). By assuming everything will be a string (except those table keys which are able to be parsed as numbers), we are able to keep the parsing routine relatively simple, which is important since the goal is to save on tokens.

Beyond the question of parsing complexity, the format is also smaller than the alternatives. Compare this output, taking up 91 characters:

```lua
'1and2indexed3data4aswellhelloworldnestedinside_ofheresomedata\'s nested'
```

with this JSON version which uses 126 characters (38% more!): 

```lua
'{"1":"and","2":"indexed","3":"data","4":{"as":"well"},"hello":"world","nested":{"some":"data\'s nested","inside_of":"here"}}'
```

It might be a moot comparison though, since the token cost of parsing JSON would be so great. Take this [pure-Lua JSON library](https://gist.github.com/tylerneylon/59f4bcf316be525b30ab), for example, which would take up nearly 750 PICO-8 tokens.

## Notes

### Types

The main current caveat is all the contained values must strings, since it's simpler to serialize and parse without embedding type information, and I only needed this for strings. I can imagine a use case where this could be useful for numeric information (e.g. large 3d models), so I'd be open to a pull request to add that support, if the token count can stay pretty low. Alternatively, it could just be a separate function.

### Error handling (or not)

In order to keep the token count low, there's no built-in error handling, so make sure any string you send to `table_from_string` was created with `serialize_table` or `stringify_table`.
