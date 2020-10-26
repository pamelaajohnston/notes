# Some good things to know when using LaTeX (in no particular order)

1. Something like TexMaker or Overleaf will help.
2. [Tables Generator](https://www.tablesgenerator.com) will help with tables.
3. [MathPix](https://mathpix.com) will help with formula
4. You can try formulas quickly at [QuickLatex](https://quicklatex.com)


5. The following code will get rid of images and tables in a document:
> % For Turnitin
> \usepackage{bibentry}
> \usepackage{comment}
> \excludecomment{figure}
> \let\endfigure\relax
> \expandafter\let\csname figure*\endcsname\figure
> \expandafter\let\csname endfigure*\endcsname\endfigure
> \excludecomment{tabular}
> \let\endtabular\relax


6. If you've compiled something using your "everything" .bib file and want to get a .bib file that has ONLY the correct references in it, use this:
> bibexport -o theNewBibFile.bib theAlreadyGeneratedAuxFile.aux

7.
