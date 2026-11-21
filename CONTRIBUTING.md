# Contributing to OpenThermal

OpenThermal is a shared place for open outdoor-thermal research. You are welcome to add a project, tool, dataset or resource, or to improve
what is here. Use **issues** for questions, bug reports and ideas, and **pull requests** for changes.

## Add a project

1. Fork the repository and create a branch.
2. Put your code in `codebases/<your-project>/` with:
   * a `README.md` that says what the project does, what is included and what is left out, where any sample data come from, and its known limitations;
   * a `LICENSE` file (the repository is Apache-2.0; another open license is fine if it suits the work, for example CERN-OHL-P-2.0 for hardware);
   * a way to run it and tests that pass, or a note saying why there are none;
   * a `CITATION.cff` with the paper to cite.
3. Add its links to the `PORTAL` block at the top of `assets/site.js`: the paper, the code folder and, if there is one, a live tool.
4. Add a card and a section to `index.html`, copying one of the existing projects, and its paper to the *Publications* list.
5. Open a pull request. Say who the authors are and that you have the right to share everything in it.

## What not to include

Credentials or tokens, personal data, private or unpublished data, and anything you do not have the right to redistribute (third-party
images, model weights with a restrictive license, and so on). Link to such things instead.

## Licenses and credit

Each project stays the work of its authors and keeps the license in its folder. The website code is Apache-2.0.
