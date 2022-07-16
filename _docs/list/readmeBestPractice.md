# Readme Best Practices
> A place to copy-paste your README.md from

One of the most crucial things in your open source project is the `README.md`
file. This repository has a ready-to-copy-paste template you can use for all
your projects.

## Getting started

Copy the `README-default.md` file for yourself and start editing! At the root of
your project, run:

```shell
curl https://raw.githubusercontent.com/jehna/readme-best-practices/master/README-default.md > README.md
```

The code above fetches the `README-default.md` file from this repository and
renames it to `README.md`.

## Fill with your own text

The default template has some guiding text to get you started. However you'll
need to edit the file with your own text to use it with your project.

```shell
atom README.md
```

If you're using [Atom](https://atom.io/) code editor, the code above opens the
file for editing. If necessary, substitute with your preferred markdown editor.

### Add to git and push

After you've filled your `README.md` file with your own project's text, you
should push it to your GitHub project:

```shell
git add README.md
git commit -m "Added: README"
git push
```

This adds the `README.md` file to your git repository, creates a commit for it
and pushes it to GitHub (or other preferred remote repository).

## Features

This project makes it easy to:
* Bootstrap your open source project properly
* Make sure everyone gets what you're trying to achieve with your project
* Follow simple instructions for a perfect `README.md`

## Contributing

As I use this for my own projects, I know this might not be the perfect approach
for all the projects out there. If you have any ideas, just
[open an issue][issues] and tell me what you think.

If you'd like to contribute, please fork the repository and make changes as
you'd like. Pull requests are warmly welcome.

If your vision of a perfect `README.md` differs greatly from mine, it might be
because your projects are for vastly different. In this case, you can create a
new file `README-yourplatform.md` and create the perfect boilerplate for that.

E.g. if you have a perfect `README.md` for a Grunt project, just name it as
`README-grunt.md`.

## Related projects

Here's a list of other related projects where you can find inspiration for
creating the best possible README for your own project:

- [Billie Thompson's README template](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2)
- [A list of awesome READMEs](https://github.com/matiassingers/awesome-readme)
- [Akash Nimare's kickass README guide](https://gist.github.com/akashnimare/7b065c12d9750578de8e705fb4771d2f)
- [Dan Bader's README template](https://github.com/dbader/readme-template)

## Licensing

This project is licensed under Unlicense license. This license does not require
you to take the license with you to your project.

[issues]:https://github.com/jehna/readme-best-practices/issues/new


![Logo of the project](https://raw.githubusercontent.com/jehna/readme-best-practices/master/sample-logo.png)

# Name of the project
> Additional information or tagline

A brief description of your project, what it is used for and how does life get
awesome when someone starts to use it.

## Installing / Getting started

A quick introduction of the minimal setup you need to get a hello world up &
running.

```shell
packagemanager install awesome-project
awesome-project start
awesome-project "Do something!"  # prints "Nah."
```

Here you should say what actually happens when you execute the code above.

### Initial Configuration

Some projects require initial configuration (e.g. access tokens or keys, `npm i`).
This is the section where you would document those requirements.

## Developing

Here's a brief intro about what a developer must do in order to start developing
the project further:

```shell
git clone https://github.com/your/awesome-project.git
cd awesome-project/
packagemanager install
```

And state what happens step-by-step.

### Building

If your project needs some additional steps for the developer to build the
project after some code changes, state them here:

```shell
./configure
make
make install
```

Here again you should state what actually happens when the code above gets
executed.

### Deploying / Publishing

In case there's some step you have to take that publishes this project to a
server, this is the right time to state it.

```shell
packagemanager deploy awesome-project -s server.com -u username -p password
```

And again you'd need to tell what the previous code actually does.

## Features

What's all the bells and whistles this project can perform?
* What's the main functionality
* You can also do another thing
* If you get really randy, you can even do this

## Configuration

Here you should write what are all of the configurations a user can enter when
using the project.

#### Argument 1
Type: `String`  
Default: `'default value'`

State what an argument does and how you can use it. If needed, you can provide
an example below.

Example:
```bash
awesome-project "Some other value"  # Prints "You're nailing this readme!"
```

#### Argument 2
Type: `Number|Boolean`  
Default: 100

Copy-paste as many of these as you need.

## Contributing

When you publish something open source, one of the greatest motivations is that
anyone can just jump in and start contributing to your project.

These paragraphs are meant to welcome those kind souls to feel that they are
needed. You should state something like:

"If you'd like to contribute, please fork the repository and use a feature
branch. Pull requests are warmly welcome."

If there's anything else the developer needs to know (e.g. the code style
guide), you should link it here. If there's a lot of things to take into
consideration, it is common to separate this section to its own file called
`CONTRIBUTING.md` (or similar). If so, you should say that it exists here.

## Links

Even though this information can be found inside the project on machine-readable
format like in a .json file, it's good to include a summary of most useful
links to humans using your project. You can include links like:

- Project homepage: https://your.github.com/awesome-project/
- Repository: https://github.com/your/awesome-project/
- Issue tracker: https://github.com/your/awesome-project/issues
  - In case of sensitive bugs like security vulnerabilities, please contact
    my@email.com directly instead of using issue tracker. We value your effort
    to improve the security and privacy of this project!
- Related projects:
  - Your other project: https://github.com/your/other-project/
  - Someone else's project: https://github.com/someones/awesome-project/


## Licensing

One really important part: Give your project a proper license. Here you should
state what the license is and how to find the text version of the license.
Something like:

"The code in this project is licensed under MIT license."
