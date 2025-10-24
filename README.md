<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/logo-simpleprompts-dark.png" />
  <source media="(prefers-color-scheme: light)" srcset="./assets/logo-simpleprompts-light.png" />
  <img alt="SimplePrompts logo" src="logo-simpleprompts-light.png" />
</picture>


Simpleprompts is a very minimal (and simple) library for constructing LLM prompts.  

# Motivation
Writing basic prompts is ~fine, it does however start to become a bit messy once you have to construct this prompt dynamically, keep/remove bits based on certain conditions, basically, once you need certain elements of control flow within the prompt construction itself. The typical solutions I've seen for constructing such prompts are:  
A) Programmatically create the prompt by stitching strings together  
```python

my_prompt = """
<role>
...
</role>\n"""

for tool in my_tools:
    my_prompt += f"<tool>\nTool Name: {tool.name}\nTool Desc: {tool.desc}</tool>\n"

my_prompt += ... # add more stuff
```
B) using a templating language like jinja2
```python
from jinja2 import Template

tmpl = """
<role>
...
</role>

<available_tools>
{% for tool in tools %}
<tool>
Tool Name: {{ tool.name }}
Tool Desc: {{ tool.desc }}
</tool>
{% endfor %}
</available_tools>

...more stuff...
"""

my_prompt = Template(tmpl).render(tools=tools)
```

Option (A) is easy to implement, but its sometimes hard to "see" the final prompt.  
With Option (B) its kind of easy to see the final prompt but you need to be aware of the templating language syntax.  

SimplePrompts is an attempt to find a middle ground that keeps you in familiar python land syntax while being reasonably readable (and hopefully powerful).

# Installation
`pip install simpleprompts`

# SimplePrompts: Examples

You can create basic prompts with `simpleprompts` in the following way:
```python
from simpleprompts import Prompt, p

my_prompt = Prompt(
    p.persona(
        "You are an expert chef who specializes in simple, 30-minute meals."
    ),
    p.instructions(
        "Generate a recipe based on the ingredients I provide. The recipe should be easy to follow and include a short, creative name."
    )
)

print(my_prompt.render())
```
Output:
```text
<persona>
You are an expert chef who specializes in simple, 30-minute meals.
</persona>

<instructions>
Generate a recipe based on the ingredients I provide. The recipe should be easy to follow and include a short, creative name.
</instructions>
```
Prefer markdown over xml? no worries.
```python
print(my_prompt.render(format="markdown"))
```
Output:
```text
# persona
You are an expert chef who specializes in simple, 30-minute meals.

# instructions
Generate a recipe based on the ingredients I provide. The recipe should be easy to follow and include a short, creative name.
```
The prompt builder namespace `p` will turn any arbitrary attr access op to a new prompt section `p.*`.
```python
prompt = Prompt(
    p.random(
        p.prompt(
            p.sections(
                "It can be nested as well :D"
            )
        )
    )
)
print(prompt.render(format="markdown"))
print()
print(prompt.render(format="xml"))
```
```text
# random
## prompt
### sections
It can be nested as well :D

<random>
<prompt>
<sections>
It can be nested as well :D
</sections>
</prompt>
</random>
```
If you would like, you can also have a specific format for each section  
```python
prompt = Prompt(
    p.random(
        p.prompt(
            p.sections(
                "It can be nested as well :D"
            ).format("xml")
        )
    )
)
print(prompt.render(format="markdown"))
```
```text
# random
## prompt
<sections>
It can be nested as well :D
</sections>
```
You can also indent specific prompt sections.
```python
prompt = Prompt(
    p.your_contacts(
        p.contact_1(
            "Name: Bob",
            "Number: +12345"
        )
        .indent(4),
        p.contact_2(
            "Name: Bolf",
            "Number: +67890"
        )
        .indent(4)
    )
)
print(prompt.render())
```
```text
<your_contacts>
    <contact_1>
    Name: Bob
    Number: +12345
    </contact_1>
    <contact_2>
    Name: Bolf
    Number: +67890
    </contact_2>
</your_contacts>
```

What if you need to create your prompt section names dynamically? in that case you would not be able to do `p.<dynamic_name>`, the alternative is to do `p(<dynamic_name>).content(...)`.
```python
from simpleprompts import Prompt, p

available_tools = [("code_interpreter", "can be used to execute code"), ("ask_human", "raise question to a human")]

prompt = Prompt(
    p.role(
        "Your role is..."
    ),
    p.instructions(
        "You should use the available tools to..."
    ),
    p.available_tools(
        *[
            p(n).content(d) 
            for n, d in available_tools
        ]
    )
)
print(...)
```
Output:
```text
<role>
Your role is...
</role>

<instructions>
You should use the available tools to..
</instructions>

<available_tools>
<code_interpreter>
can be used to execute code
</code_interpreter>
<ask_human>
raise question to a human
</ask_human>
</available_tools>
```
> Tip: `p.section_name(...)` is equivelant to `p(section_name).content(...)`.

# What's Next?
I'm still experiminting with the library's apis, and there are a few features that I'm planning to add such as:
- Image support (direct support to PIL images)
- Allow custom, user-defined renderers alongside xml and markdown
- Add more control over the sections (more methods next to `indent` and `format`)