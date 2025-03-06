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
>I made this summary because there was a slight bit of the final formatting on the generated pdf in a certain edge case that I was having trouble correctly prompting for and needed sort out the spaghetti a bit better for myself so that I could reset and fix it appropriately.
---

There is quite a bit from the chat history I'm not showing, had to go back and forth a couple times in a couple places too. My prompts here are far far from optimized and this did made me realize that I need to go back through and reoptimize my custom cursor rules bc I just realized I was still only using this basic one I set awhile ago and I likely see better results if I updated this better for some of my different project folders.
```txt
You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.  Always use Powershell commands. Always work through solutions step by step for yourself and me.
  
  Code Style and Structure
  - Write concise, technical TypeScript code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  
  Naming Conventions
  - Use lowercase with dashes for directories (e.g., components/auth-wizard).
  - Favor named exports for components.
  
  TypeScript Usage
  - Use TypeScript for all code; prefer interfaces over types.
  - Avoid enums; use maps instead.
  - Use functional components with TypeScript interfaces.
  
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative JSX.
  
  UI and Styling
  - Use Shadcn UI, Radix, and Tailwind for components and styling.
  - Implement responsive design with Tailwind CSS; use a mobile-first approach.
  
  Performance Optimization
  - Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
  - Wrap client components in Suspense with fallback.
  - Use dynamic loading for non-critical components.
  - Optimize images: use WebP format, include size data, implement lazy loading.
  
  Key Conventions
  - Use 'nuqs' for URL search parameter state management.
  - Optimize Web Vitals (LCP, CLS, FID).
  - Limit 'use client':
    - Favor server components and Next.js SSR.
    - Use only for Web API access in small components.
    - Avoid for data fetching or state management
     
  Follow Next.js docs for Data Fetching, Rendering, and Routing. 
  
```

[original]: ./dynamic-user-pdf-layout-adjustment-dev-plan.md
[completed]: ./dynamic-user-pdf-layout-adjustment-dev-plan-FINAL.md
[summary]: ./summary.md
