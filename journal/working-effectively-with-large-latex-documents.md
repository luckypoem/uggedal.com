% Working Effectively with Large LaTeX Documents
% 2008-06-19

I'm currently in the latter stages of writing my master thesis. I've been
using LaTeX from the start and have learnt a few tricks for how to work most
effectively with large documents like theses and books.

First off, you are digging your own grave if you're not seperating your
document into separate files. I tend to use a separate file for each chapter.
Long chapters could also be separated into a file for each section. For
chapters you should use the `\include{}` command for including you separate
files into you main document. The `\input{}` command is best suited for
including several sections from multiple files as it does not automatically
start a new page for every included file. A typical base file that includes
all the separate chapters could look like this:

    \documentclass[11pt,a4paper]{book}

    \title{A Long Master Thesis}

    \author{Eivind Uggedal}

    \begin{document}

      \frontmatter
        \maketitle
        \tableofcontents
        \listoffigures
        \listoftables
        \include{acknowledgements}

      \mainmatter
        \include{introduction}
        \include{background}
        \include{methodology}
        \include{implementation}
        \include{analysis}
        \include{discussion}
        \include{conclusion}

      \appendix
        \include{questionnaire}
        \include{source.code}

      \backmatter
        \bibliography{bib.items}

    \end{document}

Note that we have a separate file for our acknowledgements, every chapter,
the appendices, and use BibTeX to separate out our bibliographic information
to a separate file. The files are simply named as the argument given to
`\include{}` plus a `.tex` suffix.

Secondly you need to use a small script or a tool that automates the
compilation process of your document. A LaTeX document have to be compiled
several times to get citations, lists of figures/tables, table of contents,
references, and other items properly formatted and numbered. I wrote my own
tool called [Rubbr][rub] to handle this.

Thirdly the compiling process tends to take several seconds
when your document gets large and you've used several packages to implement
new features in your document. By using the `\includeonly{}` command you can
decide which of your files that are supposed to be included with `\include{}`
will actually be included. This functions as a white list
and should be placed before your `document` environment. I tend to have
every potential included file listed in a commented `\includeonly{}` structure
like this:

    %includeonly{%
    %acknowledgements,%
    %introduction,%
    %background,%
    %methodology,%
    %implementation,%
    %analysis,%
    %discussion,%
    %conclusion,%
    %questionnaire,%
    %source.code,%
    %}

When I'm working on a single chapter I comment out the `\includeonly{}`
command an the chapter in question:

    includeonly{%
    %acknowledgements,%
    %introduction,%
    background,%
    %methodology,%
    %implementation,%
    %analysis,%
    %discussion,%
    %conclusion,%
    %questionnaire,%
    %source.code,%
    }

This way compilation times decreases substantially and you can get more rapid
feedback of how your document looks. I have in place a similar construct in
my document preamble for easily switch on and off `draft` mode:

    \documentclass[12pt,%
                   %draft,%
                   a4paper]{book}

A small deletion of the comment character (`%`) gives me a document in draft
mode:

    \documentclass[12pt,%
                   draft,%
                   a4paper]{book}

[rub]: http://rubbr.rubyforge.org/
