# ToolSeeds

Programming tools can be transmitted as prompts instead of finished software. ToolSeeds is a library of prompts that help AI coding assistants create or adapt programming tools.

A tool seed describes a tool's purpose and expected behavior so an assistant can build it in the context of your project. Unlike finished software, a seed can be adapted to your language, framework, and constraints.

## Tools

1. [callGraphBrowser](callGraphBrowser/prompt.md): an interactive, layered call-graph page for a project, with functions colored by operation family, hover highlighting of callers and callees, and a side panel for details. Published as an artifact.

## Using a tool

Open a coding agent in the project you want the tool for, then paste in the tool's `prompt.md` along with any relevant project context or constraints. The agent builds the tool against your code. Review the result and run the project's tests.

## Creating a new tool

1. Make a directory named after the tool in camelCase, such as `myToolName/`.
2. Add a `prompt.md` file to it. Write the prompt so it works in any project:
   - Say what the tool is and what it produces.
   - Group the requirements under headings, for example DATA, LAYOUT, INTERACTION.
   - Be specific where a vague prompt would go wrong. Name the edge cases, the
     libraries, and the mistakes you've already seen an agent make.
   - End with a VERIFY step: what the agent must check before calling the job
     done, and what it should report back.
3. Run the prompt on at least one real project and revise it until the result
   is what you want.
4. Add the tool to the list above with a one-line description and a link to
   its `prompt.md`.
