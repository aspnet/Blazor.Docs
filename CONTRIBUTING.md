# Contributing to the Blazor documentation

This document covers the process for contributing to the articles and code samples that are hosted at http://blazor.net. Contributions may be as simple as typo corrections or as complex as new articles.

## How to make a simple correction or suggestion

Articles are stored in the repository as markdown files. Simple changes to the content of a markdown file can be made in the browser by selecting the **Improve this Doc** link in the upper-right corner of the browser window, editing the doc on GitHub, and then submitting a pull request (PR) for the change. We will review the PR and accept it or suggest changes.

## How to make a more complex submission

You'll need a basic understanding of [Git and GitHub.com](https://guides.github.com/activities/hello-world/).

* Open an [issue](https://github.com/aspnet/Blazor.Docs/issues/new) describing what you want to do, such as change an existing article or create a new one. Wait for approval from the team before you invest much time. We usually request an outline for a new topic proposal.
* Fork the [aspnet/Blazor.Docs](https://github.com/aspnet/Blazor.Docs/) repo and create a branch for your changes.
* Submit a pull request (PR) to master with your changes.
* If your PR has the label 'cla-required' assigned, [complete the Contribution License Agreement (CLA)](https://cla.dotnetfoundation.org/)
* Respond to PR feedback.

## Markdown syntax

Articles are written in [DocFx-flavored Markdown (DFM)](http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html), which is a superset of [GitHub-flavored Markdown (GFM)](https://guides.github.com/features/mastering-markdown/). For examples of DFM syntax for UI features commonly used in the Blazor documentation, see [Metadata and Markdown Template](https://github.com/dotnet/docs/blob/master/styleguide/template.md) in the .NET repo style guide. 

## Folder structure conventions

For each markdown file, there may be a folder for images and a folder for sample code. For example, if the article is *tutorials/build-your-first-blazor-app.md*, the images are in *tutorials/build-your-first-blazor-app/\_static* and the sample app project files are in *tutorials/build-your-first-blazor-app/samples/2.x* (for a 2.x versioned sample). An image in the *tutorials/build-your-first-blazor-app.md* file is rendered by the following markdown.

```
![description of image for alt attribute](tutorials/build-your-first-blazor-app/_static/image-name.png)
```

**All** images should have [alt text](https://wikipedia.org/wiki/Alt_attribute).

Markdown file names and image file names should be all lower case. Use dashes to separate words in file names.

## Internal links

Internal links should use the `uid` of the target article with an xref link:

```
[link_text](xref:uid_of_the_topic)
```

For more informatin, see the [DocFX Cross Reference](http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html#cross-reference).

## Code snippets

Articles frequently contain code snippets to illustrate points. DFM lets you copy code into the Markdown file or refer to a separate code file. We prefer to use separate code files whenever possible to minimize the chance of errors in the code. The code files should be stored in the repo using the folder structure described above for sample projects. 

Here are some examples of [DFM code snippet syntax](http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html#code-snippet) that would be used in a *tutorials/build-your-first-blazor-app.md* file.

To render an entire code file as a snippet:

```
[!code-csharp[Main](tutorials/build-your-first-blazor-app/sample/Program.cs)]
```

To render a portion of a file as a snippet by using line numbers:

```
[!code-csharp[Main](tutorials/build-your-first-blazor-app/sample/Program.cs?range=1-10,20,30,40-50]
[!code-html[Main](tutorials/build-your-first-blazor-app/sample/Pages/Index.cshtml?range=1-10,20,30,40-50]
```

For C# snippets, you can reference a [C# region](https://docs.microsoft.com/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-region). Whenever possible, use regions rather than line numbers because line numbers in a code file tend to change and become out of sync with line number references in markdown. C# regions can be nested; and if you reference the outer region, the inner `#region` and `#endregion` directives aren't rendered in a snippet. 

To render a C# region named "snippet_Example":

```
[!code-csharp[Main](tutorials/build-your-first-blazor-app/sample/Program.cs?name=snippet_Example)]
```

To highlight selected lines in a rendered snippet (usually renders as yellow background color):

```
[!code-csharp[Main](tutorials/build-your-first-blazor-app/sample/Program.cs?name=snippet_Example&highlight=1-3,10,20-25)]
[!code-csharp[Main](tutorials/build-your-first-blazor-app/sample/Program.cs?range=10-20&highlight=1-3]
[!code-html[Main](tutorials/build-your-first-blazor-app/sample/Pages/Index.cshtml?range=10-20&highlight=1-3]
[!code-javascript[Main](tutorials/build-your-first-blazor-app/sample/BlazorApp1.csproj?range=10-20&highlight=1-3]
```

## Test your changes with DocFX

Test your changes with the [DocFX command-line tool](https://dotnet.github.io/docfx/tutorial/docfx_getting_started.html#2-use-docfx-as-a-command-line-tool), which creates a locally-hosted version of the site. DocFX doesn't render style and site extensions created for docs.microsoft.com.

DocFX requires the .NET Framework on Windows or Mono for Linux/macOS. 

### Windows instructions

* Download and unzip *docfx.zip* from [DocFX releases](https://github.com/dotnet/docfx/releases).
* Add DocFX to your PATH.
* In a console window, navigate to root of the repo (where the *docfx.json* file resides) and run the following command:

  ```
  docfx --serve
  ```
	
* In a browser, navigate to `http://localhost:8080`.

### Mono instructions

* Install Mono via Homebrew:

  ```
  brew install mono
  ```

* Download the [latest version of DocFX](https://github.com/dotnet/docfx/releases).
* Extract the archive to */bin/docfx*.
* Create an alias for **docfx**:

  ```
  function docfx {
    mono $HOME/bin/docfx/docfx.exe
  }
    
  function docfx-serve {
    mono $HOME/bin/docfx/docfx.exe serve _site
  }
  ```

* Run `docfx` from the root of the repo to build the site, and `docfx-serve` to view the site at `http://localhost:8080`.

## Voice and tone

Our goal is to write documentation that is easily understandable by the widest possible audience. To that end, we've established guidelines for writing style that we ask our contributors to follow. For more information, see [Voice and tone guidelines](https://github.com/dotnet/docs/blob/master/styleguide/voice-tone.md) in the .NET repo.

## Microsoft Writing Style Guide

The [Microsoft Writing Style Guide](https://docs.microsoft.com/style-guide/welcome/) provides writing style and terminology guidance for all forms of technology communication, including the Blazor documentation.
