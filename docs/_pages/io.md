---
permalink: /io/
title: "Input, Output and Formats"
---

With SIRIUS 4.4.0 we finalized and released the SIRIUS project-space,
the file based persistence layer and output format of SIRIUS. Both, CLI
and GUI store the computed results as SIRIUS project-space (see the
Figure below) which in turn can also be an input for the GUI or the CLI.
This allows the user to review results in the GUI that have been
computed with an automated workflow using the CLI.

## Input

### Mass spectra

##### Peakslists/CSV-Format

The input of SIRIUS are MS and MS/MS spectra as simple peak lists.
SIRIUS can read CSV files which contain on each line a m/z and an
intensity value separated by either a whitespace, a comma or a TAB
character. For example:

```
85.041199 4034.674316 
203.052597 12382.624023 
245.063171 50792.085938
275.073975 124088.046875 
305.084106 441539.125 
335.094238 4754.061035
347.09494 13674.210938 
365.105103 55487.472656
```
The intensity values can be arbitrary floating point values. SIRIUS will
transform the intensities into relative intensities, so only the ratio
between the intensity values is important.

##### MGF-Format

SIRIUS also supports the MGF (Mascot Generic Format). This file format
was developed for peptide spectra for the mascot search engine. Each
spectrum in a MGF file can contain many spectra each starting with and
ending with . Peaks are again written as pairs of m/z and intensity
values separated by whitespaces with one peak per line. Further meta
information can be given as NAME=VALUE pairs. SIRIUS recognizes the
following meta information:

  - PEPMASS: contains the measured mass of the ion (e.g. the parent
    peak)

  - CHARGE: contains the charge of the ion. As SIRIUS supports only
    single charged ions, this value can be either 1+ or 1-.

  - MSLEVEL: should be 1 for MS spectra and 2 for MS/MS spectra. SIRIUS
    will treat higher values automatically as MS/MS spectra, although,
    it might be that it supports MSn spectra in future versions.

This is an example for a MGF file:

```
BEGIN IONS 
PEPMASS=438.32382 
CHARGE=1+ 
MSLEVEL=2 
185.041199 4034.674316
203.052597 12382.624023 245.063171 50792.085938 275.073975 124088.046875
305.084106 441539.125 335.094238 4754.061035 347.09494 13674.210938
365.105103 55487.472656 
END IONS
```
See also the GNPS database for other examples of MGF files.

##### SIRIUS MS-Format

<span>**<span style="color: red">\[ms file needs
update\]</span>**</span> 

A disadvantage of these data formats is that
they do not contain all information necessary for SIRIUS to perform the
computation. Missing meta information have to be provided via the
commandline. Therefore, SIRIUS supports also an own file format very
similar to the MGF format above. The file ending of this format is `.ms`.
Each file contains one measured compound (but arbitrary many spectra).
Each line may contain a peak (given as m/z and intensity separated by a
whitespace), meta information (starting with the **\>** symbol followed
by the information type, a whitespace and the value) or comments
(starting with the **\#** symbol). The following fields are recognized
by SIRIUS:


  - \>compound: The name of the measured compound (or any placeholder).
    This field is **mandatory**.

  - \>parentmass: the mass of the parent peak

  - \>formula: The molecular formula of the compound. This information
    is helpful if you already know the correct molecular formula and
    just want to compute a fragmentation tree or recalibrate the
    spectrum

  - \>ion: the ionization mode. See for the format of ion modes.

  - \>charge: is redundant if you already provided the ion mode.
    Otherwise, it gives the charge of the ion (1 or -1).

  - \>ms1: All peaks after this line are interpreted as MS peaks

  - \>ms2: All peaks after this line are interpreted as MS/MS peaks

  - \>collision: The same as \>ms2 with the difference that you can
    provide a collision energy

An example for a .ms file:
```
>compound Gentiobiose
>formula C12H22O11 
>ionization \[M+Na\]+ 
>parentmass 365.10544

>ms1 
365.10543 85.63 366.10887 11.69 367.11041 2.67

>collision 20 
185.041199 4034.674316 
203.052597 12382.624023 
245.063171 50792.085938 
275.073975 124088.046875 
305.084106 441539.125 
335.094238 4754.061035 
347.09494 13674.210938 
365.105103 55487.472656
```
Note: The `.ms` file format is SIRIUS' internal format and might change with as same speed as the SIRIUS
developend went forward. Nevertheless we try to provide backward compatibilty where possible.
A more detailed and commented but also WIP example of the an `.ms` file can be found [here](ms-file.md)   


### LCMS-Runs

SIRIUS can import full LCMS-Runs in (mzML/mzXML) format via the
prepossessing tool . 
<span>**<span style="color: red">\[todo: describe lcms-align\]</span>**</span>

## Output

### SIRIUS project-space

The SIRIUS project-space is a standardized directory structure that is
organized in a three hierarchy levels, namely, the *project level*, the
*compound level* and the *method level* (see Figure below for details).

{% capture fig_img %}
![Foo]({{ "/assets/images/project-space.svg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Schema of the SIRIUS project-space.</figcaption>
</figure>


On the *project level*, each compound corresponds to one sub-directory
(*compound level*) storing the input data, parameters and results of the
different analysis methods. These data is continuously written to the
project-space, so that it represents the actual progress of a SIRIUS
analysis. Further, the `.progress` file gives an overview about the
progress of the ongoing analysis. On the *compound level*, each method
provided by SIRIUS stores its results in its own sub-directory (*method
level*). This allows the user to redo one analysis step without having
to recompute the intermediate results it depends on. Further, SIRIUS is
able to transfer intermediate results to a new project-space, so
different parameters can easily be evaluated without having to recompute
intermediate results. Since a project-space can be imported into the
GUI, the user is able to judge intermediate results using the GUI before
executing further analysis steps. Project-spaces can be read and written
as an uncompressed directory or a compressed zip archive when using the
`.sirius` file extension.

### Summary files

In addition to the *method level* results, the project-space contains
summaries of these results on the *project level* and the *compound
level*. These summaries are in *tsv* (tab-separated-values) format (`summary_<NAME>.tsv`) to
provide easy access to the results for further downstream analysis, data
sharing and data visualization. The summaries are not imported into
SIRIUS but are (re-)created based on the actual results every time a
project-space is exported.

#### Molecular formula results summary

`formula_identifications.tsv` contains the top-ranked formula result of each compound as 
determined by the SIRIUS score, or the ZODIAC if available.
However, different adduct candidates with the same precursor ion molecular formula
will have identical score (e.g. `[C20H14O6 + NH4]+` and `[C20H19NO7 - H2O + H]+`).
In such cases, the the top-ranked candidate in `formula_identifications.tsv` is resolved
to `[C20H17NO6 + H]+` only considering the ion type but ignoring adduct types.
`formula_identifications_adducts.tsv` contains all top-ranked adducts (in this case 
`[C20H14O6 + NH4]+` and `[C20H19NO7 - H2O + H]+`).

#### Molecular structure results summary

`compound_identifications.tsv` contains the top-ranked structure result of each compound 
as determined using the CSI:FingerID score; the molecular formula of the top structure
does not have to be the top-ranked molecular formula of this compound.
`compound_identifications_adducts.tsv` contains the top structure for each considered 
adduct of the top molecular formula.

#### CANOPUS results summary

`canopus_summary.tsv` contains compound classes predicted to be present by CANOPUS for
the top-ranked molecular formula of each compound. *most specific class* denotes the most specific compound class for this compound. The columns
*level 5*, *subclass*, *class*, and *superclass* refer to the ancestors of this most specific class. The column *all classifications* contains
all classes predicted for this compound with probability above 50%.

If there are multiple molecular formulas with
same score (which should happen only for adducts, see the molecular formula results summary) then the
`canopus_summary.tsv` will decide for one molecular formula for each compound. We always choose the molecular formula
for which the CANOPUS probability of the *most specific class* is maximal.

`compound_identifications_adducts.tsv` contains the classifications for the other top-scoring adduct formulas, too.

### Standardized project-space summary with mzTab-M

The project-space is a SIRIUS-specific format that allows the user to
access all results and analysis details, but may not be optimal for
sharing this data with third party tools or data archives. For this
purpose, SIRIUS provides an analysis report (`analysis_report.mztab`) in
the standardized mzTab-M format. All results summarized in this report
are linked to the results in the corresponding SIRIUS project-space,
allowing the user to share summarized results using mzTab-M without
losing the connection to the detailed results provided in the
project-space. Furthermore, SIRIUS passes meta information (if available in the input) 
such as scan numbers and ids of the input data into this analysis report. This allows
for an easy combination of the SIRIUS results with the results of other
analyses such as MS1-based quantification.

### Parameter formats

#### Ion modes

Whenever SIRIUS requires the ion mode, it should be given in the
following format:

```
[M+ADDUCT]+ for positive ions
[M+ADDUCT]- for negative ions
[M-ADDUCT]+/[M-ADDUCT]- for losses 
[M]+/[M]- for instrinsically charged compounds
```

*ADDUCT* is the molecular formula of the adduct. The most common
ionization modes are `[M+H]+, [M+Na]+,[M-H]-, [M+Cl]-`. 
Currently, SIRIUS supports only single-charged compounds, so `[M+2H]2+` 
is not valid and will be skipped during import. Further dimers are also not yet
supported so that everything containing `[2M]` will be skipped during import.

#### Molecular formulas

Molecular Formulas in SIRIUS must not contain brackets. Hence, `2(C2H2)` is not a
valid molecular formula; write `C4H4` instead. Furthermore, all molecular
formulas in SIRIUS are always neutral, and there is no possibility to
add a charge on a molecular formula (instead, charges are given
separately). Hence, `CH3+` is not a valid molecular formula. Write `CH3` instead, and
provide the charge separately via commandline option.

#### Chemical alphabets

Whenever SIRIUS requires the chemical alphabet, you have to provide
which elements should be considered and what is the maximum amount for
each element. Chemical alphabets are written like molecular formulas.
The maximum amount of an element is written in square brackets behind
the element. If no square brackets are given, the element might occur
arbitrary often. The standard alphabet is
CHNOP\[5\]S, allowing the elements C, H, N O and S as well as up to five times the element P.

