# Dockerfiles

There are a [lot](https://hub.docker.com/r/publicisworldwide/data-science-python/) [of](https://hub.docker.com/r/datascienceworkshops/data-science-at-the-command-line/~/dockerfile/) [data science](https://hub.docker.com/r/jupyter/datascience-notebook/) docker files out there. Why make another?

1. Automatically [pull in the latest personal dotfiles](https://github.com/vkostyuk/config-files) from my repo. (Outside of Jupyter and RStudio, I mostly use Tmux + Vim for development.)
2. Automatically pull in SHA key files (from build directory to $HOME/.ssh).
3. Install the Python and R packages, and the CLI tools, I most often use for data science.
4. Allow others to run and reproduce analyses in the same environment.

A general data science Dockerfile can be found in the [datasci-img](https://github.com/vkostyuk/Dockerfiles/tree/master/datasci-img) directory. I'll write another Dockerfile that inherits from it and adds [RStudio Server](https://www.rstudio.com/products/rstudio/download-server/) and [LaTeX](https://github.com/yihui/tinytex/).
