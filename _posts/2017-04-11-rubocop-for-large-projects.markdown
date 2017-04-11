---
layout: post
title: "Rubocop for Large Projects"
date: 2017-04-11 12:00
comments: false
---

Long-lived and valuable software projects typically pass though the hands of multiple maintainers. Each maintainer provide their unique insight and leave a fingerprint on the code. During the lifetime of a software project, the code is read many times compared to the times it is actually changed. There are many factors in how easy a piece of code is to understand, including your level of experience, whether you wrote it, and how well it is factored.

One simple aspect of code readability is at the very surface, the characters on the page. The consistency spacing, use or lack of newlines, compactness of complexity. Is the story of the system told consistently and compellingly, or is the essence of the system buried in irrelevant, potentially distracting, details? Does a simple change entail editing code in more locations than seems reasonable, and having to remember to do everything "just so"?

There are a countless ways of standardizing on code style. Personally I've found that the exact details of the style does not matter quite so much as the consistency of the code. Project contributions, including those for open source projects, are more readily accepted if you closely follow the style already found in the codebase, rather than trying to impose your personal style in your contribution "because cleaner".

## Getting started

### Tool Installation

Adding rubocop to a project is very simple. Usually you want it available in the "development" and "test" environments, so add it to your `Gemfile` like this:

``` ruby
group :development, :test do
  gem 'rubocop', require: false
end
```

Run the `bundle` command, then make a commit with these simple changes.

### Configuring and Running

Invoking the command interactively is like you would expect, just a `bundle exec rubocop`. The output from the initial run is most likely overwhelming, or you would not be reading this. We will fix that right up.

Generate a configuration that accepts your code as it currently is, warts and all:

``` shell-command
$ bundle exec rubocop --auto-gen-config
$ echo "inherit_from: .rubocop_todo.yml" > .rubocop.yml
```

Go ahead and commit the two files `.rubocop.yml` and `.rubocop_todo.yml`. If you now run the `rubocop` command, everything should be green and accepted. You now have a baseline to work from.


## Overview of Operations

### Types of automated rules

There are four types of rules called cops in rubocop:

* *Lint cops* check for things that frequently cause problems. A special always-enabled lint cop is `Lint/Syntax` which checks for syntax errors. Other lint cops include whether assignment in conditions are allowed and whether matching `if` and `end` should always line up. These are things that are a good idea to fix right away.

* *Style cops* are based on the [ruby style guide](https://github.com/bbatsov/ruby-style-guide). These are rules covering whitespace in large detail, comment styles, formatting of hash and array literals, you name it, there's probably a styler for it. Many of these cops can be configured to your preference. This means you can decide if you prefer hash-rockets `=>` in your hashes, parenthesis around method arguments, and lots of other little details. The defaults match the ruby style guide, so if your codebase more or less follow that style, you will have an easy time. There are also quite a few variations on the code standard you can pick and choose from.

* *Metrics cops* look at a bit higher level at code complexity measurements. Are classes and modules a reasonable length? Are methods sufficiently short for easy understanding? Are there an extraordinary number of parameters for a method, signifying underlying issues? Are there way way too many paths through a single method with a bunch of nested conditionals?

* *Rails cops* are concerned with some best practices that are helpful in Rails projects, such as preferring `Time.zone.now` over `Time.now` to avoid issues with timezones, new vs. old style setup of validations, and a few other niceties. Enable these if your project is of the Rails persuation, they are not automatically enabled. Read on for details about how to do this.

### Configuring rules and exceptions

Configuration is stored in a `.rubocop.yml` file in the root of your project. If your project matches the default ruby style guide settings, you can even skip having a configuration file altogether.

By default style, lint and metrics cops are enabled. Rails cops have to be enabled manually, simply for the reason that not all Ruby-code is Rails code. To enable Rails cops, simply create a new file `.rubocop.yml` and add this as the contents:

``` yaml
inherit_from: .rubocop_todo.yml

AllCops:
  RunRailsCops: true
```

**For the very messy projects**: You can even have a `.rubocop.yml` in a subfolder if something needs a whole lot of exceptions initially. Maybe your `spec/` or `test/` folder is going to require a whole lot of changes before the codebase is in a consistently readable state. That said, starting out you will probably appreciate keeping things simple and maintaining just a single `.rubocop.yml` file in the root of your project. This makes it abundantly clear what rules are enabled where.

### Appropriate commit size for easy review

As we go through the rules and options you have, there will arise a great desire to Fix All The Things - Now. Using the power of modern version control, you can most certainly do that. The primary drawback I have observed is the enormous burden this places on the unfortunate human reviewers which have to read through one or a few commits that basically change the entire face of the code-base. You might even be that human, so for the sake of the reviewing human (especially if it is you), go slow and change things deliberately.

Craft each commit to focus on one or a few related issues, document what issues you have fixed, and update the `rubocop` configuration as you go to now forbid the issue you now fixed. This way you can send fewer changes at a time to be reviewed, meaning faster review, and less chance of a merge conflict from an unrelated concurrent code change. Feature additions have right of way when merging, and cleanup commits are most likely easier to re-create when handling conflicts. By keeping each commit small and focused (and as automated as you can), recreating a particular cleanup can be performed without guesswork.


## Pass 1: Fixing all Lint warnings

First of all we are going to focus on fixing up all Lint warnings. These include problems flagged by ruby when you run with warnings enabled such as shadowed and unused variables.

### Quick and automated wins

First of all we are going to fix the Lint issues that rubocop has an automated response to. Make sure you have a clean working copy - commit or stash your current changes. Create a new branch for this work, a good name would be `chore/rubocop-round-1-fix-lint-warnings`.

Then let us automatically fix some of the serious warnings in your codebase:

``` shell-command
$ echo > .rubocop_todo.yml
$ bundle exec rubocop --lint --auto-correct
```

The first `echo` line will clear the current `TODO` file you might have, thereby allowing all automated linters to run in line two.

Typical issues this will fix are:

* Unused variables for methods or blocks get a `_` in front of their old name. That way it is clear they are intentionally not used. Read through these proposed changes carefully, you might find a bug or two this way.

* Replacing `File.exists?` with `File.exist?` because the former is deprecated.

* Smaller things like simplifying `"Oh #{thing.to_s}"` to `"Oh #{thing}"`, where the `.to_s` call made no actual difference to the behavior.

Check through each of the changes, then commit them. Make sure you record the command you used here in your commit message, should you need to recreate this commit later on. Somebody else might have changed the integration branch while you were busy inspecting the changes, and it's nice to be able to easily re-create the update without much effort. Since this particular commit is the result of an automated process, re-running it is a lot easier than starting to manually handle merge-conflicts. Just throw your branch away and re-create it from the latest integration branch if you need to - it should literally take seconds.

### Clean up the rest

There will most likely still be some warnings left when running in `--lint` mode. To see the full name of warnings use the `--display-cop-names` flag like this:

``` shell-command
$ bundle exec rubocop --lint --display-cop-names
```

There is an additional flag that can be helpful at this point called `--display-style-guide`. This adds a link to the ruby style guide for each infraction where it is relevant. Having more information about exactly what the underlying issue is can be quite helpful in determining what to do about it.

You can focus on a single of these issues, by running rubocop with a single linter at a time:

``` shell-command
$ bundle exec rubocop --lint --only AssignmentInCondition
$ bundle exec rubocop --lint --only UselessAssignment
```

Work through each of the cases reported like this:

* Update the source code manually as needed to fix the issue
* Create a commit after fixing each issue. Make sure you add a note to the commit message with details of exactly what issue you have now fixed.
* Re-run the full `bundle exec rubocop --lint`.
* Pick one specific issue to fix
* Keep this up until everything is in order.

Final step in preparing this changeset is to re-run this command:

``` shell-command
$ bundle exec rubocop --auto-gen-config
```

Commit the updated `.rubocop_todo.yml` file. Expect this to a lot shorter than it was before you started fixing the Lint issues.

Have another team member review your changes, then enjoy your updated warning free codebase.

### Automation

Finally, we are going to add a step to your Continuous Integration (CI) flow to ensure no new Lint issues arise moving forward. Simply add this single line as a step that has to pass for the build to be successful:

``` shell-interaction
$ bundle exec rubocop --lint --display-cop-names \
  --display-style-guide --rails
```

Add it now, and make sure the build still passes.

With that addition to CI, we are done with the first pass fixing all the potential issues the Lint cops capture, and we are ready to take on making the code style consistent.


## Pass 2: Easy style fixes

Agreeing on all details of a full code style takes an enormous effort. Much easier is to make gradual changes and gather feedback on what is prefered for each particular aspect.

- Consistency for readability
- Gradually make all whitespace consistent. Start small with `Style/TrailingWhitespace` and `Style/TrailingBlankLines` to get rid of any trailing whitespace and make newlines consistent across the project.
- All the indentation you can eat
- Strings and hashes (let's all agree on something)
- Placing the dots [Style/DotPosition - suggestion to support copy/paste: `Style/DotPosition:  EnforcedStyle: trailing`]
- Big number formatting [Style/NumericLiterals: ]
- Method definitions - to paranthesise or not?
- Strip out redundancies [Style/RedundantBegin, Style/RedundantReturn, Style/RedundantSelf]
- Spaces and operators

Steps:

- Pick a rule in `.rubocop_todo.yml` to remove that sounds promising.
- Lookup the documention for the rule to figure out what configuration options are available. The most complete documentation is the [rubocop defaults.yml](https://github.com/bbatsov/rubocop/blob/master/config/default.yml). There you can see all the various options for each rule. Typically a rule is listed with examples of the effect of the various options.
- Add the new configuration to your `.rubocop.yml` file
- Remove the exception in the `.rubocop_todo.yml` file
- Rerun `rubocop --auto-correct` to take care of the automated fixes.
- Commit these easy changes.
- Run `rubocop` to check for any instances that are not auto-fixable with the new rule in place.
- Fix the issues manually and commit the resulting manual work.
- A run of `rubocop` should now be green again - push your branch, create a Pull Request, and have somebody review the changes.

By keeping separate the automated and manual fixes it becomes easier to review, as the automated fixes are typically extremely similar and unremarkable. For the manual fixes a bit more care has to be spent in the review.

## Pass 3: Start taming complexity

- Large methods
- Large classes or modules
- The less obvious issues such as coupling and inconsistent or non-standard naming

## Your new daily habits

- Lower the limit on line length (usually raises class/module line count)
- Break down a single complex class into more classes
- Document why a single class exists

## Final thoughts

This guide has hopefully helped you get started with Rubocop and applying it to your large project.

For some concrete examples of making a codebase a bit more consistent, you can read through [my merge-requests to the errbit project](https://github.com/errbit/errbit/pulls?utf8=✓&q=is%3Apr%20is%3Aclosed%20author%3Arud%20sort%3Acreated-asc%20rubocop)

## Related reading

* Sandi Metz *Practical Object-Oriented Design in Ruby*, [poodr.com](http://www.poodr.com), 2012
* Nat Pryce and Steve Freeman *Growing Object-Oriented Software, Guided by Tests*, 2009
* Martin, Robert C. *Clean Code*, 2009
