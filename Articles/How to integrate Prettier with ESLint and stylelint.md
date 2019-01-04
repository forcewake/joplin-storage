How to integrate Prettier with ESLint and stylelint

## or How to never worry about code styling again

[![Go to the profile of Abhishek Jain](../_resources/85cccd1d1be447649771c039beeac053.jpeg)](https://medium.freecodecamp.org/@01abhishekjain?source=post_header_lockup)

[ESLint](https://github.com/eslint/eslint) and [stylelint](https://github.com/stylelint/stylelint) are really amazing tools that allow you to enforce coding patterns among your teams. This has many benefits, like outputting better and more consistent code, getting rid of useless diffs in commits (newline, indentation, et al.) among many others.

But over time, this can prove to be a bit of a hassle among the developers of a team, who find it an extra mental burden to manually add semicolons, newlines, indentations, etc just to conform to the lint rules. This is where a code formatting tool like [Prettier](https://github.com/prettier/prettier) comes in.

Prettier can be set up to automatically format your code according to some specified rules. If you are using VSCode, you can even have your code be formatted whenever you hit save (I’m sure there must ways to set this up in other editors but I haven’t looked into it.)

However, you don’t want to create a new Prettier config file, since you already have all the formatting related rules specified in your ESLint and stylelint  config files. So, we will need some magic for that. ✨

Let’s now dive into a step by step of how to set this all this up, and also how to format all of your existing code according to the lint rules. This guide assumes that your project already has ESLint and stylelint set up with their `.eslintrc` and `.stylelintrc` files.

### PART 1: Formatting the existing codebase

#### **Step 1**

Install [prettier-eslint](https://github.com/prettier/prettier-eslint), which is a tool that formats your JavaScript using Prettier followed by `eslint --fix`. The `--fix` is an ESLint feature that attempts to automatically fix some problems for you.

    npm install --save-dev prettier-eslint

This tool infers the equivalent Prettier config options from your existing _.eslintrc_ file. So you shouldn’t need to create a new _.prettierrc_ file in most cases.

#### **Step 2**

Install [prettier-eslint-cli](https://github.com/prettier/prettier-eslint-cli). This is the CLI tool that’ll help you run all of your files through prettier-eslint at once.

    npm install --save-dev prettier-eslint-cli

#### **Step 3**

Install [prettier-stylelint](https://github.com/hugomrdias/prettier-stylelint), which is a tool that formats your CSS/SCSS with Prettier followed by `stylelint —-fix`. Like ESLint, `--fix` is a stylelint feature that attempts to automatically fix some problems for you.

npm install prettier-stylelint --save-dev

This tool _also_ attempts to create a Prettier config based on the stylelint config.

Note that unlike prettier-eslint, you don’t have to install another package for its CLI since that is already included in it.

#### **Step 4**

Write scripts inside your `package.json` targeting the existing files in your codebase that you wish to run through prettier-eslint and prettier-stylelint.

"scripts": {

  "fix-code": "prettier-eslint --write 'src/**/*.{js,jsx}' ",

  "fix-styles": "prettier-stylelint --write 'src/**/*.{css,scss}' "

}

As you can see, I am targeting all of my existing JS and JSX and all of my existing CSS and SCSS, respectively.

The `--write` flag writes the changes in-place for the file currently being formatted. So, be careful and **make sure that all of your existing files are under source control and that there are no uncommited changed**.

#### **Step 5**

Run the scripts!

npm run fix-code  
npm run fix-styles

Now, you can check-in all of these new changes as a single big commit (maybe even from a temporary git user, if you don’t wanna pollute your own git history.)

### **Part 2: Setting up VSCode**

Now that your existing codebase is formatted, its time to make sure that all the code being written henceforth is formatted automatically.

#### **Step 1**

Install the Prettier, ESLint, and stylelint extensions for VSCode:

#### **Step 2**

Configure a few VSCode settings:

`"prettier.eslintIntegration": true` — tells Prettier to use prettier-eslint instead of Prettier

`"prettier.stylelintIntegration": true` — tells Prettier to use prettier-stylelint instead of Prettier

`"eslint.autoFixOnSave": false` — we don’t need ESLint to fix our code for us directly, since prettier-eslint will be running `eslint --fix` for us anyways.

`"editor.formatOnSave": true` — runs Prettier with the above options on every file save, so you never have to manually invoke VSCode’s format command.

Additionally, you can check-in the above workplace settings to source control so its easier for other team members to set up their editors. You can do so by creating a `.vscode` folder at the root of your project and putting all of the above rules in a `settings.json` file.

Optionally, you can tell Prettier to ignore formatting certain patterns of files. To do this, just add a `.prettierignore` file to the root of your project specifying the paths to ignore. For instance:

strings.json  
scripts/*

**And that’s it! Never worry about code styling again 😄**

This article is by no means intended to be an exhaustive guide, but rather an introduction to what is possible with the amazing tools mentioned herein. I recommend opening up the official GitHub pages for each to learn more about how to utilise these tools more effectively for your specific workflow.

Please write a comment below for any help, suggestion, etc.

#### _References_

https://prettier.io/docs/en/  
https://stylelint.io/user-guide/  
https://eslint.org/  
https://github.com/prettier/prettier-vscode  
https://github.com/prettier/prettier-eslint  
https://github.com/prettier/prettier-eslint-cli  
https://github.com/hugomrdias/prettier-stylelint  
https://www.youtube.com/watch?v=YIvjKId9m2c