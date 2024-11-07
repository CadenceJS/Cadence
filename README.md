# Cadence (Alpha)

Do not use Cadence. It is likely unsupported and will not work. It was developed for [Tachi](https://github.com/zkrising/Tachi), but is no longer being actively maintained.

Sadly, the JS ecosystem moves far too fast - I had the audacity to write this project and then move on with my life, which means it doesn't work any more. Haha.

## Try it out on your own code!

The [Cadence Playground](https://cadence.zkldi.xyz/playground) will automatically show how Cadence treats your code.

## What's the point?

- It's brutally strict. (The config is nearly one thousand lines long.)

Now, let's be honest. TypeScript has a *lot* of warts and footguns. TS would arguably be a better language if we could just cut off the bad parts. So let's do it!
Cadence is strict to the point where these footguns are caught for you *at lint time*. No longer do you have to fall for subtle JS-isms like these:
```js
isNaN("foo"); // true
Number.isNaN("foo"); // false
```
```js
function doStuff() {
	switch (foo) {
		case "bar":
			let thing = 5;
			return thing;
		case "baz":
			let thing = 6; // SyntaxError, thing is already defined !!!
			return thing;
	}
}
```
```js
const users = ["joey", "tim", "bob"];

// This actually happens in parallel !!
users.forEach(async (user) => {
	console.log(`Fetching ${user}'s email...`);
	const usersEmail = await fetch(`https://example.com/api/user-email?user=${user}`).then(r => r.text());

	console.log(`Got ${usersEmail}!`)
})

// this is called first.
console.log(`Done!`);
```

When developing Cadence, I was honestly unconvinced of its merit. However, after it found **14** bugs (some of which were serious!) in one of my codebases (~50kloc), I was completely sold.

- All code linted and formatted with the Cadence config **must have** the same format and rhythm to it. (get it?)

You even get cleaner git diffs for free.
Your imports? Those are automatically put in the right place by the linter.
Typescript union order? Deterministic.

- Cadence code is *forced* to be easily legible.

Cadence **enforces** whitespace, padding and other pretty-isms around the codebase to split up logic visually.

You might think this is nitpicky, but since it can be enforce automatically, all code with Cadence is guaranteed not to be a clumped mess of logic.

It is **extremely difficult** to write messy code with Cadence enabled!

Logic should be separated, and sparse!
We shouldn't have to think about anything other than business logic while reading code.

```js
function scrimbly() {
	const foo = "asdf";  
    const bar = 123
	let {more, variables} = scrobbly(bar);
	// subFunction foos the bar and scrimbles the bimble
	function subFunction() {
		return 1;
	}
	if (statement) 
		doThing();
	return foo;
}
```
Will be **forcibly** changed to:
```js
function scrimbly() {
	const foo = "asdf";
	const bar = 123;
	const { more, variables } = scrobbly(bar);

	// subFunction foos the bar and scrimbles the bimble
	function subFunction() {
		return 1;
	}

	if (statement) {
		doThing();
	}

	return foo;
}
```

## Using Cadence properly

A lot of Cadence's linting/formatting opinions will be **automatically** fixed.
It would be a very frustrating experience to have to go through the codebase and apply Cadence's changes by hand!

Similarly, Cadence is intended to be a real-time feedback loop on your codebase.
As such, it's intended to be used by running *on save* in your editor of choice.
For VSCode, this just means setting ESLint as your default formatter.

## Tools are here to help you!

Cadence might be frustrating at times, the same way something like TypeScript's type checking is.
At its core though, Cadence and TS are trying to do the exact same thing - Stop you from making mistakes! It might be frustrating to work around hoops when you *think you know* your code is safe, but when it actually saves your bacon, it's all worth it.

Sometimes, however, Cadence might just be straight up incorrect about parts of your code.
To get around this, use the standard ESLint ignore syntax.
```js
// eslint-disable-next-line rule-thats-giving-you-trouble
```

If you're doing something across your codebase that Cadence is giving you trouble about, you might want to reduce it to a warning in your `.eslintrc`.
```json
{
	"plugins": [
		"cadence"
	],
	"extends": [
		"plugin:cadence/recommended"
	],
		"rules": {
		"@typescript-eslint/no-unsafe-assignment": "warn"
	},
}
```


**Don't set rules to "off" unless you know EXACTLY what you're doing.**
Leaving would-be errors as warnings is useful for other developers -- they'll know something is out-of-the-ordinary here!

## What's the point of rule (xyz)?

At the moment, you'll have to check `src/configs/recommended.ts` in this repository to see my stance on each rule.
In the future, I hope to have documentation for why every single rule is enabled.
This would double up as a very useful resource on JS/TS footguns!

If the explanation in there doesn't satisfy, feel free to open a [Discussion](https://github.com/CadenceJS/Cadence).


## Setup (Fresh Repository)

You'll first need to install [ESLint](https://eslint.org/) and this plugin:

```sh
# Use your preferred package manager. If you don't have strong opinions on this, here's mine.
# I would **highly** recommend pnpm, as its the most sensible.
pnpm add eslint eslint-plugin-cadence -D

# Yarn is also pretty good, but there's a couple of pain points and generally
# worse performance.
yarn add eslint eslint-plugin-cadence -D

# use NPM if you have to. The other package managers are faster and have saner commands.
# for example, `npm install` ignores your lockfile.
npm i eslint eslint-plugin-cadence --save-dev
```

## Usage

Create an `.eslintrc` file next to your `package.json`. Fill it with the following content.

```json
{
	"plugins": [
		"cadence"
	],
	"extends": [
		"plugin:cadence/recommended"
	],
}
```

You might want to also make an `.eslintignore` file:
```sh
# We don't want to lint node_modules! 
node_modules
```

## Migrating to Cadence

Migrating a codebase to Cadence is honestly a pain in the ass.
Although it can automatically fix a hell of a lot of the issues in your codebase,
you will almost certainly have to do some manual work.

This is especially true if you depend on unsafe/unclean things throughout your codebase, such as
a liberal use of `any`, `for (const k in obj)`, `[].forEach()`, etc.

In my opinion, the best way to migrate to Cadence is as follows:

1. Make a new git branch off of your main branch.
2. Install Cadence there, and run `eslint . --fix --ext .ts`
3. It'll likely give you some errors and warnings. Step through them all and start analysing them.
4. It's quite likely it'll find some bugs in your codebase, aswell! That should hopefully convince you it's worth the setup effort.
5. If you don't like it though, you can always just straight up discard the branch and go back to normal.
6. Once you've analysed and fixed all of the errors and warnings, merge it back into your main branch!
