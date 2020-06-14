## What is this?

I'm a big fan of [Dhall][] in general, because:

 - It's an aggressively simple language, so there's not that much to learn
 - It's pure and functional (all it does is compute config, it's not for writing programs)
 - It's strongly typed, which hugely increases confidence compared to editing pages of YAML

In addition, Dhall turns out to be an _excellent_ way to manage CI boilerplate.

The problem with boilerplate is that you can't abstract it away. We typically end up copy/pasting or using generators, both of which miss out on later improvements made to the source. Ideally we could do what we normally do in languages, which is use libraries.

But when you're talking about github workflow files, travis configuration files, Dockerfiles, cloudbuild files and all the other CI configuration, it's impractical to do that. You'd have to pick a language, and make sure that's installed. Then probably a package manager. _Then_ you'd have to put together a bit of a framework for generating various files, and sort out package publication for anything you want to share between projects.

We need something that's simple, appropriate for the task and supports super lightweight dependency management. Dhall is exactly that:

 - Frictionless dependency management: you can import dhall code from the internet (safely, and with efficient caching)
 - Lightweight and simple to install (a couple static binaries totalling a dozen mb)
 - Purpose-built: dhall's stated goal is to replace YAML. Which is (mostly) what we're doing!

## How does it work?

The idea is that we can have one big dhall expression to define "all the generated files". This is super powerful - thanks to dhall's remote imports, you can use whatever types and abstractions you need by importing straight from the internet (it even supports private Github repositories).

The expression contains a map of `path -> file expression`, each of which is a dhall value (containing the file contents as well as some metadata).

We render this tree of files (into a `generated/` directory for cleanliness), then install symlinks in the workspace, pointing to the various generated files.

There's options to control the generation - you can mark files as executable, or use a plain file instead of a symlink (e.g. to workaround Github actions choking on symlinks).

And it's self-hosting. You can use the `SelfInstall` module to add this tool _as a generated file in your repo_, eliminating manual setup. You can even pre-bake the file path to use, resulting in a custom script which doesn't need any arguments.

## Got examples?

[Indeed I do](./examples/)

## Requirements

`dhall-to-json` and `ruby`

## How does it make you feel?

Honestly, it's _liberating_. Until I started down this path, I didn't realise how much mental friction I had with CI boilerplate. I was always unmotivated to fix things or improve my workflow, because it was tedious and I knew I'd have to completely redo it for every repo where I wanted the benefit.

Now, I feel like it's actually worth making proper reusable abstractions for this stuff because I can actually leverage it across as many repos as I need, and only have to maintain it once.

## Is it for me?

If you only have a couple of repos, probably not. I have a great deal of personal and work repos that I deal with, and for me the thought that each one doesn't have to reinvent the wheel is very satisfying.

## Downsides

It's not _zero_-dependency, obviously -- you can't get around your contributors needing dhall in order to change things. Many work environments have ways to make this simple (e.g. shared Docker tooling). But for public repos it depends what environment / tooling you can assume your users have.

## How stable is this?

I wrote it quickly, and it's got no automated tests. But it's very simple, the examples work, and I use it daily.

## Why is it written in Ruby?

It was expedient, because I want to use it in a workplace where ruby is ubiquitous. Maybe one day I'll rewrite it in Haskell, but then I'd need to set up more infrastructure in order to actually automate and distribute binaries.

[dhall]: https://dhall-lang.org/
