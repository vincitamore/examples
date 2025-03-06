# Example of a bit of vibe coding

- The first [dev plan][original] was generated from this prompt after completing a different task: 
  >Ok, that worked perfectly, but now that I think about it, I would like the user to have the ability to set how many items they want per page and for it to automatically resize the items and styles and whatnot, as well as put each page on its own shadowed card container to make it all look more clean. Does that make sense?     Please consider everything closely and then write a dynamic-user-pdf-layout-adjustment-dev-plan.md with a step by step plan for implementing this without breaking any of the current function or styling
---
- And the [completed dev plan][completed] is the same file after starting a new chat with the prequisite files along with the plan added into the context and prompted thusly:
  >Please proceed with following dev-plan step by step beginning at step 1, making sure to update/summarize the plan with your progress as you go along

> [!NOTE]
>As you'll notice if you pay attention this has an additional 4 steps the original doesn't, which represent me needing to adjust a few things that were missed or went a bit wonkly from intentions
---
- And then the [summary][summary] was made with the following prompt after testing and verifying the final completed steps of the dev plan
  >Awesome, you did great, it's working almost perfect! Before we continue I would like you to examine all the changes we've made and make summary.md with a full breakdown of the key places in the code where the sizing and formatting are happening with process flows and tree diagrams, Keep it as concise and informationally dense as possible while leaving nothing out

> [!NOTE]
>I made this summary because there was a slight bit of the final formatting on the generated pdf in a certain edge case that I was having trouble correctly prompting for and needed sort out the spaghetti a bit better for myself so that I could reset and fix it appropriately

[original]: ./dynamic-user-pdf-layout-adjustment-dev-plan.md
[completed]: ./dynamic-user-pdf-layout-adjustment-dev-plan-FINAL.md
[summary]: ./summary.md
