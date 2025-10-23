# Simpleprompts

This readme is still T.B.D.

But here is a quick motivation for why I made this. 
Building prompts started to get really annoying when I need to "conditionally" add or remove stuff and switch between formats (go from xml to markdown for example), so I decided to create this minimal DSL (Domain Specific Langauge) that can help me create prompts programmatically while still being fairly readable.

```python
from simpleprompts import Prompt, p

my_prompt = Prompt(
    p.role(
        "You are a helpful assistant"
    ),
    p.available_tools(
        p.search_web(
            "Use this tool to get up to-date data from the web"
        ),
        p.code_interpreter(
            "Use this tool to execute code"
        )
    ),
    p.guidelines(
        "1. Always use the `search_web` tool if you need up to date info",
        "2. Always use the code_interpreter tool if you need to perform computations"
    )
)

print(my_prompt.render(format="xml"))
```
```
<role>
You are a helpful assistant
</role>

<available_tools>
<search_web>
Use this tool to get up to-date data from the web
</search_web>
<code_interpreter>
Use this tool to execute code
</code_interpreter>
</available_tools>

<guidelines>
1. Always use the `search_web` tool if you need up to date info
2. Always use the code_interpreter tool if you need to perform computations
</guidelines>
```
or you can do 
```python
print(prompt.render(format="markdown"))
```
```
# role
You are a helpful assistant

# available_tools
## search_web
Use this tool to get up to-date data from the web
## code_interpreter
Use this tool to execute code

# guidelines
1. Always use the `search_web` tool if you need up to date info
2. Always use the code_interpreter tool if you need to perform computations
```
You can also do stuff like this:
```python
my_prompt = Prompt(
    p.role(
        "You are a helpful assistant"
    ),
    p.available_tools(
        p.search_web(
            "Use this tool to get up to-date data from the web"
        ).indent(2).format("xml"),
        p.code_interpreter(
            "Use this tool to execute code"
        ).indent(2).format("xml")
    ),
    p.guidelines(
        "1. Always use the `search_web` tool if you need up to date info",
        "2. Always use the code_interpreter tool if you need to perform computations"
    )
)

print(my_prompt.render(format="markdown"))
```
```
# role
You are a helpful assistant

# available_tools
  <search_web>
  Use this tool to get up to-date data from the web
  </search_web>
  <code_interpreter>
  Use this tool to execute code
  </code_interpreter>

# guidelines
1. Always use the `search_web` tool if you need up to date info
2. Always use the code_interpreter tool if you need to perform computations
```

I wil expand more on this minimal library in the incoming days!